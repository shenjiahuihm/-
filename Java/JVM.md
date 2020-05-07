# JVM

## 1. 线程

## 2. JVM内存区域

## 2.1 线程私有（Thread Local）

1. 程序计数器 PC
2. 虚拟机栈 VM Stack
3. 本地方法栈 Native Method Stack

## 2.2 线程共享（Thread Shared）

1. 方法区（永久代）Method Area
2. 类实例区 (Java堆) Objects

## 2.3 直接内存



## Minor GC ，Full GC 触发条件

Minor GC触发条件：当Eden区满时，触发Minor GC。

Full GC触发条件：

（1）调用System.gc时，系统建议执行Full GC，但是不必然执行

（2）老年代空间不足

（3）方法去空间不足

（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存

（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

 