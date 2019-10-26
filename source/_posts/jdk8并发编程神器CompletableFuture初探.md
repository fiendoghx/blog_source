---
title: jdk8并发编程神器CompletableFuture初探
date: 2019-10-26 16:22:52
categories:
- Java
tags:
- CompletableFuture
- jdk8
- 并发编程
---


### 目录
* [1.缘由](#1-缘由)
* [2.目的](#2-目的)
* [3.具体实施](#3-具体实施)
    * [3.1 事前准备](#3-1事前准备)
    * [3.2 实施并验证](#3-2实施并验证)
		* [3.2.1 并发打印helloworld](#3-2-1并发打印helloworld)
		* [3.2.2 并发协作](#3-2-2并发协作)
* [4.为什么采用以上方式](#4-为什么采用以上方式)

## 1.缘由

最近在开始整理jdk8的新特性新功能。发现多线程处理这块出了一个超级好用的工具类--CompletableFuture。
之前写并发程序总是采用Thread，Runnable，Callable等源生写法，实现起来比较复杂，稍有不慎就导致bug不断且排查困难。于是准备以几个案例来记录下CompletableFuture的大致用法。


## 2.目的

能够根据案例正确的使用CompletableFuture来解决相关并发问题。

## 3.具体实施

### 3.1事前准备

* 了解线程基础知识
* 了解lambda表达式
* 了解函数式编程
* 了解CountDownLatch

### 3.2实施并验证

**基础方法**

|方法  | 说明 | 线程有无返回值|
| :----------:| :-----------:| :-----------:|
| runAsync() | 异步运行一个线程 | 无 |
| supplyAsync() | 异步运行一个线程 | 有 |
| thenAcceptAsync()() |以上个线程的执行结果作为参数，异步运行一个线程 | 无 |
| thenApplyAsync()() |以上个线程的执行结果作为参数，异步运行一个线程 | 有 |


#### 3.2.1并发打印helloworld

采用CountDownLatch挂起主线程，等所有HelloWorld都打印完毕才结束

```
final int times = 10;
final CountDownLatch countDownLatch = new CountDownLatch(times);
for (int i = 0; i < 10; i++) {
	CompletableFuture.runAsync(()->{
		System.out.println("Hello World. " + Thread.currentThread().getName() + ": " 
				+ Thread.currentThread().getId() + " " + LocalTime.now());
		countDownLatch.countDown();
	});
}
countDownLatch.await();
```

运行可得到:

```
Hello World. ForkJoinPool.commonPool-worker-1: 11 17:11:48.922
Hello World. ForkJoinPool.commonPool-worker-2: 12 17:11:48.922
Hello World. ForkJoinPool.commonPool-worker-3: 13 17:11:48.922
Hello World. ForkJoinPool.commonPool-worker-2: 12 17:11:48.923
Hello World. ForkJoinPool.commonPool-worker-1: 11 17:11:48.923
Hello World. ForkJoinPool.commonPool-worker-2: 12 17:11:48.923
Hello World. ForkJoinPool.commonPool-worker-3: 13 17:11:48.923
Hello World. ForkJoinPool.commonPool-worker-3: 13 17:11:48.923
Hello World. ForkJoinPool.commonPool-worker-1: 11 17:11:48.923
Hello World. ForkJoinPool.commonPool-worker-2: 12 17:11:48.923
```

#### 3.2.2并发协作


## 4.为什么采用以上方式




