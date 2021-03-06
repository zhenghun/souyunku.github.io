---
layout: post
title: 什么是阻塞队列？如何使用阻塞队列来实现生产者-消费者模型？
categories: java
description: 什么是阻塞队列？如何使用阻塞队列来实现生产者-消费者模型？
keywords: java 
---

## 什么是阻塞队列？

阻塞队列是一个在队列基础上又支持了两个附加操作的队列。

2个附加操作：

支持阻塞的**插入**方法：队列满时，队列会阻塞插入元素的线程，直到队列不满。

支持阻塞的**移除**方法：队列空时，获取元素的线程会等待队列变为非空。

## 阻塞队列的应用场景

阻塞队列常用于生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。简而言之，阻塞队列是生产者用来存放元素、消费者获取元素的容器。

## 几个方法

在阻塞队列不可用的时候，上述2个附加操作提供了四种处理方法

方法\处理方式 |	抛出异常 |	返回特殊值 |	一直阻塞 |	超时退出
---|---|---|---|---|
插入方法|	add(e)	   |offer(e)|	put(e)	|   offer(e,time,unit)
移除方法|	remove()   |poll()  |	take()  |	poll(time,unit)
检查方法|	element()  |peek()  |	不可用  |	不可用

## JAVA里的阻塞队列

JDK 7 提供了7个阻塞队列，如下

1、**ArrayBlockingQueue** 数组结构组成的有界阻塞队列。

此队列按照先进先出（FIFO）的原则对元素进行排序，但是默认情况下不保证线程公平的访问队列，即如果队列满了，那么被阻塞在外面的线程对队列访问的顺序是不能保证线程公平（即先阻塞，先插入）的。

2、**LinkedBlockingQueue**一个由链表结构组成的有界阻塞队列

此队列按照先出先进的原则对元素进行排序

3、**PriorityBlockingQueue** 支持优先级的无界阻塞队列

4、**DelayQueue** 支持延时获取元素的无界阻塞队列，即可以指定多久才能从队列中获取当前元素

5、**SynchronousQueue**不存储元素的阻塞队列，每一个put必须等待一个take操作，否则不能继续添加元素。并且他支持公平访问队列。

6、**LinkedTransferQueue**由链表结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列，多了tryTransfer和transfer方法

transfer方法

如果当前有消费者正在等待接收元素（take或者待时间限制的poll方法），transfer可以把生产者传入的元素立刻传给消费者。如果没有消费者等待接收元素，则将元素放在队列的tail节点，并等到该元素被消费者消费了才返回。

tryTransfer方法

用来试探生产者传入的元素能否直接传给消费者。，如果没有消费者在等待，则返回false。和上述方法的区别是该方法无论消费者是否接收，方法立即返回。而transfer方法是必须等到消费者消费了才返回。

7、LinkedBlockingDeque链表结构的双向阻塞队列，优势在于多线程入队时，减少一半的竞争。

## 如何使用阻塞队列来实现生产者-消费者模型？

通知模式实现：所谓通知模式，就是当生产者往满的队列里添加元素时会阻塞住生产者，当消费者消费了一个队列中的元素后，会通知生产者当前队列可用。

## 使用BlockingQueue解决生产者消费者问题

**为什么BlockingQueue适合解决生产者消费者问题**

任何有效的生产者-消费者问题解决方案都是通过控制生产者put()方法（生产资源）和消费者take()方法（消费资源）的调用来实现的，一旦你实现了对方法的阻塞控制，那么你将解决该问题。

Java通过BlockingQueue提供了开箱即用的支持来控制这些方法的调用（一个线程创建资源，另一个消费资源）。java.util.concurrent包下的BlockingQueue接口是一个线程安全的可用于存取对象的队列。

**BlockingQueue是一种数据结构，支持一个线程往里存资源，另一个线程从里取资源。这正是解决生产者消费者问题所需要的，那么让我们开始解决该问题吧。**

**生产者**

以下代码用于生产者线程

```java
package io.ymq.example.thread;

import java.util.concurrent.BlockingQueue;

/**
 * 描述:生产者
 *
 * @author yanpenglei
 * @create 2018-03-14 15:52
 **/
class Producer implements Runnable {

    protected BlockingQueue<Object> queue;

    Producer(BlockingQueue<Object> theQueue) {
        this.queue = theQueue;
    }

    public void run() {
        try {
            while (true) {
                Object justProduced = getResource();
                queue.put(justProduced);
                System.out.println("生产者资源队列大小= " + queue.size());
            }
        } catch (InterruptedException ex) {
            System.out.println("生产者 中断");
        }
    }

    Object getResource() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException ex) {
            System.out.println("生产者 读 中断");
        }
        return new Object();
    }
}
```


**消费者**

以下代码用于消费者线程
```java
package io.ymq.example.thread;

import java.util.concurrent.BlockingQueue;

/**
 * 描述: 消费者
 *
 * @author yanpenglei
 * @create 2018-03-14 15:54
 **/
class Consumer implements Runnable {
    protected BlockingQueue<Object> queue;

    Consumer(BlockingQueue<Object> theQueue) {
        this.queue = theQueue;
    }

    public void run() {
        try {
            while (true) {
                Object obj = queue.take();
                System.out.println("消费者 资源 队列大小 " + queue.size());
                take(obj);
            }
        } catch (InterruptedException ex) {
            System.out.println("消费者 中断");
        }
    }

    void take(Object obj) {
        try {
            Thread.sleep(100); // simulate time passing
        } catch (InterruptedException ex) {
            System.out.println("消费者 读 中断");
        }
        System.out.println("消费对象 " + obj);
    }
}
```

**测试该解决方案是否运行正常**

```java
package io.ymq.example.thread;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

/**
 * 描述: 测试
 *
 * @author yanpenglei
 * @create 2018-03-14 15:58
 **/
public class ProducerConsumerExample {

    public static void main(String[] args) throws InterruptedException {

        int numProducers = 4;
        int numConsumers = 3;

        BlockingQueue<Object> myQueue = new LinkedBlockingQueue<Object>(5);

        for (int i = 0; i < numProducers; i++) {
            new Thread(new Producer(myQueue)).start();
        }

        for (int i = 0; i < numConsumers; i++) {
            new Thread(new Consumer(myQueue)).start();
        }

        Thread.sleep(1000);

        System.exit(0);
    }
}
```

**运行结果**

```sh
生产者资源队列大小= 1
生产者资源队列大小= 1
消费者 资源 队列大小 1
生产者资源队列大小= 1
消费者 资源 队列大小 1
消费者 资源 队列大小 1
生产者资源队列大小= 1
生产者资源队列大小= 3
消费对象 java.lang.Object@1e1aa52b
生产者资源队列大小= 2
生产者资源队列大小= 5
消费对象 java.lang.Object@6e740a76
消费对象 java.lang.Object@697853f6

......

消费对象 java.lang.Object@41a10cbc
消费对象 java.lang.Object@4963c8d1
消费者 资源 队列大小 5
生产者资源队列大小= 5
生产者资源队列大小= 5
消费者 资源 队列大小 4
消费对象 java.lang.Object@3e49c35d
消费者 资源 队列大小 4
生产者资源队列大小= 5
```

**从输出结果中,我们可以发现队列大小永远不会超过5，消费者线程消费了生产者生产的资源**。


# Contact

 - 作者：鹏磊  
 - 出处：[http://www.ymq.io](http://www.ymq.io)  
 - 版权归作者所有，转载请注明出处
 - Wechat：关注公众号，搜云库，专注于开发技术的研究与知识分享
 
![关注公众号-搜云库](http://www.ymq.io/images/souyunku.png "搜云库")