---
layout: post
title: JmsTemplate如何重用Session
category: Java
tags: [Java]
keywords: concurrent
description:
---

# CachingConnectionFactory缓存Session的原理

## 关键类

1. `JmsTemplate`
2. `SingleConnectionFactory`
3. `CachingConnectionFactory`
4. `SharedConnectionInvocationHandler`
5. `CachedSessionInvocationHandler`

## 如何拿到缓存的Session

无论你使用的是`SingleConnectionFactory`还是`CachingConnectionFactory`作为`ConnectionFactory`.
都是调用`SingleConnectionFactory`的`createConnection()`来创建`connection`.

```java
SingleConnectionFactory.java
@Override
public Connection createConnection() throws JMSException {
    return getSharedConnectionProxy(getConnection());
}

protected Connection getSharedConnectionProxy(Connection target) {
    List<Class<?>> classes = new ArrayList<>(3);
    classes.add(Connection.class);
    if (target instanceof QueueConnection) {
        classes.add(QueueConnection.class);
    }
    if (target instanceof TopicConnection) {
        classes.add(TopicConnection.class);
    }
    return (Connection) Proxy.newProxyInstance(Connection.class.getClassLoader(),
            ClassUtils.toClassArray(classes), new SharedConnectionInvocationHandler());
}
```

所以`Spring-Jms`对该`connection`做了一个代理。当后面调用`createSession`等方法的时候，会进入`SharedConnectionInvocationHandler`。

当对`Connection`调用`createSession`的时候会进入`SharedConnectionInvocationHandler`。

```java
CachingConnectionFactory.java("SharedConnectionInvocationHandler")
else if (method.getName().equals("createSession") || method.getName().equals("createQueueSession") ||
        method.getName().equals("createTopicSession")) {
    // Default: JMS 2.0 createSession() method
    Integer mode = Session.AUTO_ACKNOWLEDGE;
    if (args != null) {
        if (args.length == 1) {
            // JMS 2.0 createSession(int) method
            mode = (Integer) args[0];
        }
        else if (args.length == 2) {
            // JMS 1.1 createSession(boolean, int) method
            boolean transacted = (Boolean) args[0];
            Integer ackMode = (Integer) args[1];
            mode = (transacted ? Session.SESSION_TRANSACTED : ackMode);
        }
    }
    Session session = getSession(getConnection(), mode);
    if (session != null) {
        if (!method.getReturnType().isInstance(session)) {
            String msg = "JMS Session does not implement specific domain: " + session;
            try {
                session.close();
            }
            catch (Throwable ex) {
                logger.trace("Failed to close newly obtained JMS Session", ex);
            }
            throw new javax.jms.IllegalStateException(msg);
        }
        return session;
    }
}
try {
    return method.invoke(getConnection(), args);
}
catch (InvocationTargetException ex) {
    throw ex.getTargetException();
}
```

这里的关键便是`getSession(...)`方法，如果使用的是`SingleConnectionFactory`则会直接返回`null`。如果使用的是`CachingConnectionFactory`则会有下面的逻辑。

```java
CachingConnectionFactory.java
@Override
protected Session getSession(Connection con, Integer mode) throws JMSException {
    if (!this.active) {
        return null;
    }

    LinkedList<Session> sessionList = this.cachedSessions.computeIfAbsent(mode, k -> new LinkedList<>());
    Session session = null;
    synchronized (sessionList) {
        if (!sessionList.isEmpty()) {
            session = sessionList.removeFirst();
        }
    }
    if (session != null) {
        if (logger.isTraceEnabled()) {
            logger.trace("Found cached JMS Session for mode " + mode + ": " +
                    (session instanceof SessionProxy ? ((SessionProxy) session).getTargetSession() : session));
        }
    }
    else {
        Session targetSession = createSession(con, mode);
        if (logger.isDebugEnabled()) {
            logger.debug("Registering cached JMS Session for mode " + mode + ": " + targetSession);
        }
        session = getCachedSessionProxy(targetSession, sessionList);
    }
    return session;
}
```

简单来说就是`CachingConnectionFactory`会维护一个以`Mode`为key，`LinkedList<Session>`为value的map。每当请求一个`Session`的时候，会先从这个map取出对应的`LinkedList`,然后`removeFirst()`。若remove的结果不为null，则会直接返回。否则会`createSession(con, mode)`去新生成一个Session。然后使用`CachedSessionInvocationHandler`对该Session进行代理。

## 如何将Session重新放回缓存

上面我们发现Spring每取出一个`Session`就会从缓存将其删除，这是因为Jms Api并没有要求`Session`的实现是线程安全的。所以取出`Session`的时候需要将其删除，以防止其它线程在同一时间使用它。那么在当前线程使用完，什么时候将session重新返还缓存的map的呢？关键代码在`CachedSessionInvocationHandler`中。


`JmsTemplate`在Send完消息之后最终会调用`Session`的`close`方法。无论调用何种方法，最终都由代理类`CachedSessionInvocationHandler`来代理完成。在`close`的时候会将该session重新缓存,具体代码如下。

```java
CachingConnectionFactory.java(CachedSessionInvocationHandler)
else if (methodName.equals("close")) {
    // Handle close method: don't pass the call on.
    if (active) {
        synchronized (this.sessionList) {
            if (this.sessionList.size() < getSessionCacheSize()) {
                try {
                    logicalClose((Session) proxy);
                    // Remain open in the session list.
                    return null;
                }
                catch (JMSException ex) {
                    logger.trace("Logical close of cached JMS Session failed - discarding it", ex);
                    // Proceed to physical close from here...
                }
            }
        }
    }
    // If we get here, we're supposed to shut down.
    physicalClose();
    return null;
}

private void logicalClose(Session proxy) throws JMSException {
    // Preserve rollback-on-close semantics.
    if (this.transactionOpen && this.target.getTransacted()) {
        this.transactionOpen = false;
        this.target.rollback();
    }
    // Physically close durable subscribers at time of Session close call.
    for (Iterator<Map.Entry<ConsumerCacheKey, MessageConsumer>> it = this.cachedConsumers.entrySet().iterator(); it.hasNext();) {
        Map.Entry<ConsumerCacheKey, MessageConsumer> entry = it.next();
        if (entry.getKey().subscription != null) {
            entry.getValue().close();
            it.remove();
        }
    }
    // Allow for multiple close calls...
    boolean returned = false;
    synchronized (this.sessionList) {
        if (!this.sessionList.contains(proxy)) {
            this.sessionList.addLast(proxy);
            returned = true;
        }
    }
    if (returned && logger.isTraceEnabled()) {
        logger.trace("Returned cached Session: " + this.target);
    }
}
```

## 如何删除失败的Connection

`CachingConnectionFactory`会缓存`Connection`(即`SingleConnectionFactory`的成员变量`connection`)，那么在过程中如果因为各种原因connection出了问题，需要将这个`connection`重新重置为null。

那么Spring-Jms是怎样将`connection`重置为的null呢？这个过程有点难以发现，因为在打印出来的异常栈中相关的代码并不能发现任何能够将`connection`置为null的代码。

首先Spring-Jms中将这个connection置为null的代码只有一处，如下：
```java
SingleConnectionFactory.java
public void resetConnection() {
    synchronized (this.connectionMonitor) {
        if (this.target != null) {
            closeConnection(this.target);
        }
        this.target = null;
        this.connection = null;
    }
}

@Override
public void onException(JMSException ex) {
    logger.info("Encountered a JMSException - resetting the underlying JMS Connection", ex);
    resetConnection();
}
调用这个方法的
```

那么是在哪里调用这个方法？

下面以`Artemis-Client`作为实际的Jms Api实现来讲述。

发送消息失败最终会调用`ExceptionListener`的`OnException()`方法，这中间的过程由Aretemis client来实现的，不在本文讨论范围之中。


```java
ActiveMQConnection.java
@Override
public synchronized void connectionFailed(final ActiveMQException me, boolean failedOver) {
    if (me == null) {
    return;
    }

    ActiveMQConnection conn = connectionRef.get();

    if (conn != null) {
    try {
        final ExceptionListener exceptionListener = conn.getExceptionListener();

        if (exceptionListener != null) {
            final JMSException je = new JMSException(me.toString(), failedOver ? EXCEPTION_FAILOVER : EXCEPTION_DISCONNECT);

            je.initCause(me);

            new Thread(new Runnable() {
                @Override
                public void run() {
                exceptionListener.onException(je);
                }
            }).start();
        }
    } catch (JMSException e) {
        if (!conn.closed) {
            ActiveMQJMSClientLogger.LOGGER.errorCallingExcListener(e);
        }
    }
    }
}
```

它会利用`conn.getExceptionListener()`返回一个`ExceptionListener`，而这个`ExceptionListener`实际上就是`SingleConnectionFactory`(该`factory`实现了该接口)。之后调用`ExceptionListener`的`onException`方法来将`connection`置为`null`。

上面`conn.getExceptionListener()`之所以能够返回`SingleConnectionFactory`，是因为该connection实际上是一个代理，真实是调用`SharedConnectionInvocationHandler`(如下)。

```java
SingleConnectionFactory.java
else if (method.getName().equals("getExceptionListener")) {
    synchronized (connectionMonitor) {
        if (this.localExceptionListener != null) {
            return this.localExceptionListener;
        }
        else {
            return getExceptionListener();
        }
    }
}
```


## 相关代码
JmsTmplate发送消息的核心代码

```java
JmsTemplate.java
protected void doSend(Session session, Destination destination, MessageCreator messageCreator)
        throws JMSException {

    Assert.notNull(messageCreator, "MessageCreator must not be null");
    MessageProducer producer = createProducer(session, destination);
    try {
        Message message = messageCreator.createMessage(session);
        if (logger.isDebugEnabled()) {
            logger.debug("Sending created message: " + message);
        }
        doSend(producer, message);
        // Check commit - avoid commit call within a JTA transaction.
        if (session.getTransacted() && isSessionLocallyTransacted(session)) {
            // Transacted session created by this template -> commit.
            JmsUtils.commitIfNecessary(session);
        }
    }
    finally {
        JmsUtils.closeMessageProducer(producer);
    }
}


@Nullable
public <T> T execute(SessionCallback<T> action, boolean startConnection) throws JmsException {
    Assert.notNull(action, "Callback object must not be null");
    Connection conToClose = null;
    Session sessionToClose = null;
    try {
        Session sessionToUse = ConnectionFactoryUtils.doGetTransactionalSession(
                obtainConnectionFactory(), this.transactionalResourceFactory, startConnection);
        if (sessionToUse == null) {
            conToClose = createConnection();
            sessionToClose = createSession(conToClose);
            if (startConnection) {
                conToClose.start();
            }
            sessionToUse = sessionToClose;
        }
        if (logger.isDebugEnabled()) {
            logger.debug("Executing callback on JMS Session: " + sessionToUse);
        }
        return action.doInJms(sessionToUse);
    }
    catch (JMSException ex) {
        throw convertJmsAccessException(ex);
    }
    finally {
        JmsUtils.closeSession(sessionToClose);
        ConnectionFactoryUtils.releaseConnection(conToClose, getConnectionFactory(), startConnection);
    }
}
```