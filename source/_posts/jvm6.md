---
title: jvm学习(jvm堆分析)
date: 2017-11-24 15:07:18
tags:
  - tech
  - jvm

---

# jvm学习(jvm堆分析)

## oom原因：堆 永久区 线程栈 直接内存
- 线程栈是操作系统分配的
- 堆空间和线程栈和直接内存合起来是 系统可分配内存 解决方法减少堆空间和线程栈空间
- native内存溢出的话 堆空间用得很省，可以减少堆空间，让它触发gc 可以回收直接内存

## 辅助工具
- MAT memory analyzer 基于eclipse软件
  1. 概念：对象引用图-》支配树 
     支配者被回收，被支配者也会被回收 不可达
  2. 用途
	最大对象信息
	查看对象的引用对象 和 被引用的对象
	浅堆 String 24个字节 对象头8字节+3*4(int)+4(char[])引用4个字节 = 24  只和对象结构有关 
	深堆 对象实际内存空间大小 gc后实际释放的内存大小  浅堆之和
## visual vmz分析堆
- oql
## tomcat 分析案例

1. 现象
> jmeter 压测后  找出oom原因和状态  再给出oom问题方法 找到最大对象
最大对象staticmanager 里有个currentHashMap内存都消耗这里session保存过多
接下去使用oql select objects from session 9000多个对象 每个深堆内存1.5k 
2. 解决方案
> select createtime from sesssion 可以得到qps 320/s 导致oom
解决方案：增加堆大小 缩短session时间
