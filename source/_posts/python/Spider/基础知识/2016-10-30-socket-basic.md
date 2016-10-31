---
author: ivyxjc
date: 2016-10-30
title: python socket编程功能
category: Python
tags: [python,spider,socket]
keywords:
description: 利用socket获取网络内容
---

## socket编程基本思路

### 服务器端

1. 创建socket, 绑定socket到本地IP, 端口 socket.socket(socket.AF_INET,socket.SOCK_STREAM), s.bind()
2. 开始监听连接  s.listen()
3. 进入循环，不断接受客户端的连接请求  s.accept()
4. 然后接收传来的数据，并发送给对方数据  s.recv() , s.sendall()
5. 传输完毕后，关闭socket  

### 客户端
1. 创建套接字，连接远端地址 #socket.socket(socket.AF_INET,socket.SOCK_STREAM), s.connect()
2. 连接后发送数据和接收数据 #s.sendall(), s.recv()
3. 传输完毕后，关闭套接字 #s.close()

```python
# 导入socket库:
import socket
# 创建一个socket:
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 建立连接:
s.connect(('www.sina.com.cn', 80))
```

创建socket时, `socket.AF_INET`代表使用用IPv4协议, 如果要使用IPv6协议, 则指定`sockt.AF_INET6`, `SOCK_STREAM`指定使用面向流的TCP协议.

```python
s.send(b'GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')

# 接收数据:
buffer = []
while True:
    # 每次最多接收1k字节:
    d = s.recv(1024)
    if d:
        buffer.append(d)
    else:
        break
data = b''.join(buffer)

# 关闭连接:
s.close()


# 处理数据
header, html = response.split(b'\r\n\r\n', 1)
print(header.decode("utf-8"))
print(html.decode('utf-8'))
```

## socket 并发

socket默认是堵塞的, 当socket运行`connect()`,`recv()`方法时, 会堵塞直到运行完成.

```python
sock = socket.socket()
#将socket设为非堵塞的self.
sock.setblocking(False)
```

这样做会直接抛出一个`BlockingIOError`的异常, 详细内容`A non-blocking socket operation could not be completed immediately`,

**解决方法**:
```python
try:
    sock.connect(('xkcd.com', 80))
except BlockingIOError:
    pass
```

```python
request = 'GET {} HTTP/1.0\r\nHost: xkcd.com\r\n\r\n'.format(url)
encoded=request.encode('ascii')
sock.send(encoded)
```

也会抛出异常`OSError: A request to send or receive data was disallowed because the socket is not connected and (when sending on a datagram socket using a sendto call) no address was supplied`

**解决方法**:

```python
while True:
    try:
        sock.send(encoded)
        break  # Done.
    except OSError as e:
        pass
```



















a















n
