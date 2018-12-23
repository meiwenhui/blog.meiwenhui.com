---
layout: post
title: "数据结构之队列"
description: "队列（queue）是一种特殊的线性表，具有先进先出的特性。"
keywords: "数据结构, 队列"
categories: [data-structure]
permalink: /:categories/queue/
---

### 什么是队列
队列是一种特殊的线性表，其插入和删除的操作分别在表的两端进行，队列的特点就是先进先出(First In First Out)。我们把向队列中插入元素的过程称为入队(Enqueue)，删除元素的过程称为出队(Dequeue)并把允许入队的一端称为队尾，允许出的的一端称为队头，没有任何元素的队列则称为空队。其一般结构如下：


### 代码实现
关于队列的操作，我们这里主要实现入队，出队，判断空队列和清空队列等操作，声明队列接口Queue（队列抽象数据类型）
```java
public interface Queue<T> { /**
    * 返回队列长度
    * @return
    */ int size(); /**
    * 判断队列是否为空
    * @return
    */ boolean isEmpty(); /**
    * data 入队,添加成功返回true,否则返回false,可扩容
    * @param data
    * @return
    */ boolean add(T data); /**
    * offer 方法可插入一个元素,这与add 方法不同，
    * 该方法只能通过抛出未经检查的异常使添加元素失败。
    * 而不是出现异常的情况，例如在容量固定（有界）的队列中
    * NullPointerException:data==null时抛出
    * @param data
    * @return
    */ boolean offer(T data); /**
    * 返回队头元素,不执行删除操作,若队列为空,返回null
    * @return
    */ T peek(); /**
    * 返回队头元素,不执行删除操作,若队列为空,抛出异常:NoSuchElementException
    * @return
    */ T element(); /**
    * 出队,执行删除操作,返回队头元素,若队列为空,返回null
    * @return
    */ T poll(); /**
    * 出队,执行删除操作,若队列为空,抛出异常:NoSuchElementException
    * @return
    */ T remove(); /**
    * 清空队列
    */ void clearQueue();
}

```
https://blog.csdn.net/javazejian/article/details/53375004
