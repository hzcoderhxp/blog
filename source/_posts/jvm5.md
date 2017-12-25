---
title: jvm学习(性能监控)
date: 2017-11-24 13:07:18
tags:
  - tech
  - jvm

---

# jvm学习(性能监控)
## 命令工具
- uptime
- top
- ``dstat -alt ``彩色看cpu load io等
- vmstat 1 4
- pidstat -p pid -u 1 4 -t可以显示线程id (怎么从这里线程id找到对应java代码？)
- pidstat -p pid -d 1 4 -t可以显示线程id (可以看磁盘io情况)

- jps 查看java进程 类似ps
	1. -q 进程号
	2. -m 
    3. -l(这好用) 
    4. -v
- jinfo 查看java程序运行参数 甚至可以改
	1. jinfo -flag PrintGCDetails 2972
	2. -XX:-PrintGCDetails(主要+ -)
	3. jfinfo -flag +PrintGCDetails 2972打开gc日志
- jmap
	1. 生成java应用程序的堆快照和对象统计信息
	2. jmap -histo 2972 > /stone/data/gc.txt
	3. jmap -dump:format=b,file=/stone/data/heap.hprof 2972将4. 堆信息快照dump下来
- jstack
    1. 线程快照查看  jstack pid
    2. 查看是否先锁占用 jstack -l pid 

	3. 打印线程dump
	> -l 打印锁信息
	> -m 打印java和native的帧信息
	> -F 强制dump 当jstack没有响应时使用
- jstat
	
	1. 垃圾收集查看jstat -gc 12538 5000
## 图形工具    
- jconsole
	> 查看java运行情况 监控堆信息 永久区使用情况 类加载情况
- visual vm
## 实战
### 应用卡死
1. jps 查进程
2. jstatck 线程打印出来
3. 发现等待socket 说明卡在网络io等待
### cpu占很高
1. jps 
2. update load还是高的
3. top cup占用率高
4. pidstat -p pid 1 3 -u -t发现线程3467占用很高cpu资源
5. 将3467改成16进制 因为jstatck也是16进制的 d8b
6. jstack找 d8b Holdcuptask.run这个方法看她在做什么
### 查看是否死锁
1. jstack 线程dump中 查看是否两个线程等待wait一个对象