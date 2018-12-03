---
layout: post
title: "外观模式"
description: "外观模式,他隐藏了系统的复杂性，并向客户端提供了一个可以访问系统的接口。"
keywords: "外观模式, 外观模式"
categories: [design-pattern]
permalink: /:categories/facade/
---

外观模式（Facade）,他隐藏了系统的复杂性，并向客户端提供了一个可以访问系统的接口。这种类型的设计模式属于结构性模式。为子系统中的一组接口提供了一个统一的访问接口，这个接口使得子系统更容易被访问或者使用。 


### UML
![a](/assets/images/design-pattern-facade.jpg)

#### 应用场景

每个Computer都有CPU、Memory、Disk。在Computer开启和关闭的时候，相应的部件也会开启和关闭，所以，使用了该外观模式后，会使用户和部件之间解耦

The standard chunk of Lorem Ipsum used since the 1500s is reproduced below for those interested. Sections 1.10.32 and 1.10.33 from "de Finibus Bonorum et Malorum" by Cicero are also reproduced in their exact original form, accompanied by English versions from the 1914 translation by H. Rackham.

cpu子系统类
```java
public class CPU {
    public void start() {
        System.out.println("cpu is start...");
    }
    
    public void shutDown() {
        System.out.println("CPU is shutDown...");
    }
}
```
Disk子系统类
```java
public class Disk {
    public void start() {
        System.out.println("Disk is start...");
    }
    
    public void shutDown() {
        System.out.println("Disk is shutDown...");
    }
}
```
Memory子系统类
```java
public class Memory {
    public void start() {
        System.out.println("Memory is start...");
    }
    
    public void shutDown() {
        System.out.println("Memory is shutDown...");
    }
}
```
**门面类（核心）, 把所有的子系统的开和关都封装了**

```java
public class Computer {
    
    private CPU cpu;
    private Memory memory;
    private Disk disk;

    public Computer() {
        cpu = new CPU();
        memory = new Memory();
        disk = new Disk();
    }

    public void start() {
        System.out.println("Computer start begin");
        cpu.start();
        disk.start();
        memory.start();
        System.out.println("Computer start end");
    }
    
    public void shutDown() {
        LOSystem.out.println("Computer shutDown begin");
        cpu.shutDown();
        disk.shutDown();
        memory.shutDown();
        System.out.println("Computer shutDown end...");
    }
}
```
#### 优点

* **松散耦合**  
使得客户端和子系统之间解耦，让子系统内部的模块功能更容易扩展和维护；
* **简单易用**  
客户端根本不需要知道子系统内部的实现，或者根本不需要知道子系统内部的构成，它只需要跟Facade类交互即可。
*  **更好的划分访问层次**  
有些方法是对系统外的，有些方法是系统内部相互交互的使用的。子系统把那些暴露给外部的功能集中到门面中，这样就可以实现客户端的使用，很好的隐藏了子系统内部的细节

> This content was copied from http://www.lipsum.com/ as an example of post article.
