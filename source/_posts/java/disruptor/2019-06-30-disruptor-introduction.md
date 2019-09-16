---
layout: post
title: Disruptor源码分析（一） 简介
category: Java
tags: [Java]
keywords: concurrent
description:
---

Disruptor 是由LMAX Exchange推出的高性能定长无锁队列。它不仅提供了一个队列的实现，且为整个生产者，消费者模式提供了周边的支持。

<!-- more -->
- [简介](#%E7%AE%80%E4%BB%8B)
- [几个概念](#%E5%87%A0%E4%B8%AA%E6%A6%82%E5%BF%B5)
  - [RingBuffer](#RingBuffer)
  - [Sequence](#Sequence)
  - [Sequencer](#Sequencer)
  - [SequenceBarrier](#SequenceBarrier)
- [图片解释](#%E5%9B%BE%E7%89%87%E8%A7%A3%E9%87%8A)
- [相关的文章](#%E7%9B%B8%E5%85%B3%E7%9A%84%E6%96%87%E7%AB%A0)
  - [伪共享](#%E4%BC%AA%E5%85%B1%E4%BA%AB)
  - [锁的缺点](#%E9%94%81%E7%9A%84%E7%BC%BA%E7%82%B9)
  - [内存屏障](#%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C)

## 简介
Disruptor使用环形数组来作为数据容器。利用SequenceBarrier来作为消费者的读屏障，利用消费的sequence作为生产者的写屏障。

大量的文章都是直接开始分析代码，这样的方式有一个问题：由于生产者和消费者以及数据容器之间有一定交互，单独从一部分分析代码都会使初学者有所困扰。所以本文先介绍Disruptor的几个概念以及Disruptor的总体设计，之后的几篇文章再分析具体的代码。

## 几个概念
由于Disruptor为了性能对数组及其它一些类都有填充，这些填充只是基于性能考虑，对代码逻辑没有什么影响。下面所说的数组长度等都不包括填充部分。

### RingBuffer
`RingBuffer`是Disruptor的核心，是其存储数据的地方，本质是一个环形数组，其数组长度是`bufferSize`。环形数组只是一种形象的说明，它仍然是一个普通的Java数组。之所以称其为环形数组是因为Disruptor利用下面介绍的`Sequence`来作为下标，`Sequence`的值是不断地增加的，只要与数组的size取模即可计算出真实的数组下标（Disruptor并不会取模），看起来就像是一个下标可以无限增长的定长数组。

上面说到Disruptor并不会取模，这是因为Disruptor对数组的长度有一个规定，必须是2的幂，这样可以用`index&(bufferSize-1)`来进行下标计算而不是用取模来计算下边。这是因为位操作的速度要远快于取模速度。

RingBuffer在初始化的时候会将数组内的每个位置用开发人员自定义的`EventFactory`将其填满。同时会生成另外一个与存储数据大小相同的标志位数组。用来标记该位是否可以消费，但是并不是我们常见的`boolean`型标记位。原因很简单，因为如果使用`boolean`来标记该位是否可以消费，当生产者生产了个数为`bufferSize`的数据之后，标志位就全都是`true`，之后就起不到标志位的作用。Disruptor是将当前的`sequence`的值写入其中，之后通过巧妙的计算来得出该位是否可以写入。

### Sequence

`Sequence`和`java.util.concurrent`中的`AtomicLong`功能是类似的。只不过做了一些填充。左右各填充8个Long（仅为性能考量，与代码逻辑无关）。

### Sequencer

由于Disruptor使用`Sequence`来作为数组的下标的来源，所以对数组的操作（例如将数据插入数组，覆盖数据）实质上都是对`Sequence`的操作。

Disruptor对于`Sequence`的使用也提供了一个专门的接口即`Sequencer`。`Sequencer`又继承了两个接口：`Sequenced`和`Cursored`。`Sequenced`接口主要是提供给生产者使用的。`Sequencer`接口本身填充了一些方法主要是为了生成一个`SequenceBarrier`.
。它有两个实现：`SingleProducerSequencer`和`MultiProducerSequencer`。

生产者生产实质上是调用其中的两个方法 1. `next(int n)` 2. `publish(long index)`

第1步是申请当前位置的之后的n个位置，成功的条件是不能覆盖最慢的消费者，即currentSequence+n所在的位置必须已经被最慢的消费者消费才可以成功。否则就使用`LockSupport.parkNanos(1);`等待再重试。

第二步是在将利用`EventTranslator`将生产的数据填充到数组之后，将标志位数组对应的位置`setAvailable`


### SequenceBarrier

`SequencerBarrier`可以视作消费者的读屏障，它确保了消费者不会消费还未生产的数据，即消费者的下标不会越过生产者的下标。

## 图片解释

直接用文字解释起来相对比较抽象，我画了一张图来解释。

![Disruptor](https://1drv.ms/u/s!Ag0m8cr29utVgZ0FuGIe9ej6N2292Q)

## 相关的文章

Disruptor是要追求比较高的性能的，所以它在很多细节处都有不一样的设计。下面是几篇关于Disruptor的文章。

### 伪共享

很多时候在代码层面并不存在共享的问题，但是因为硬件系统的一些设计会出现一些意想不到的共享问题，即伪共享。伪共享对性能也有一定的影响。

可以阅读下面的文章了解

[伪共享（false sharing），并发编程无声的性能杀手](https://www.cnblogs.com/cyfonly/p/5800758.html)

### 锁的缺点

Disruptor是一个无锁队列。不使用锁当然是因为使用锁有一定的缺点。

可以阅读下面的文章了解

[Dissecting the Disruptor: Why it's so fast (part one) - Locks Are Bad](http://mechanitis.blogspot.com/2011/07/dissecting-disruptor-why-its-so-fast.html)

对应的中文译文
[剖析Disruptor:为什么会这么快？(一)锁的缺点](http://ifeve.com/locks-are-bad/)


### 内存屏障

Java中的`volatile`关键词提供了关于内存屏障的支持。它会在写操作后插入一个写屏障指令，在读操作前插入一个读屏障指令。使用`volatile`的消耗是小于锁的，但是仍然有一定的性能损耗。

可以阅读下面的文章了解

[Dissecting the Disruptor: Demystifying Memory Barriers](http://mechanitis.blogspot.com/2011/08/dissecting-disruptor-why-its-so-fast.html)
对应的中文译文
[剖析Disruptor:为什么会这么快？(三)揭秘内存屏障](http://ifeve.com/disruptor-memory-barrier/)