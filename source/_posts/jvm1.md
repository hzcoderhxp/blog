---
title: jvm学习(基本参数)
date: 2017-11-23 10:07:18
tags:
  - tech
  - jvm

---

# jvm学习(基本参数)

## 基本参数
* ``-XX:+PrintGCDetails`` : 打印gc详情程序结束以后会打印
* ``-Xloggc:log/gc.log`` : 指定gc log位置，以文件输出
* ``-XX:+PrintHeapAtGC`` : 每次一次GC后；打印堆信息
* ``-XX:+TraceClassLoading`` : 监控类的加载
* ``-XX:PrintClassHistogram`` : 按下ctrl+break后打印类的信息
* ``-Xmx -Xms`` : 指定最大堆和最小堆,可以通过``Runtime.getRuntime().maxMemory();``
* ``-Xmn`` 新生代大小 
* ``-XX:NewRatio`` 新生代和老生代比率 
* ``-XX:SurvivorRatio`` 设置两个surivivor(form+to)去和eden的比率
* ``-XX:+HeapDumpOnOutOfMemoryError`` OOM时导出堆到文件
* ``-XX:+HeapDumpPath``导出OOM的路径  两个命令配合起来使用
* ``-XX:OnOutOfMemoryError`` 在OOM时执行一个脚本发送邮件、重启程序等
* ``-XX:PermSize -XX：MaxPermSize`` 永久区初始空间和最大空间 一般表示能放多少类型

## 其他
- 官方推荐新生代占比3/8 
- 在oom的时候记得dump出堆 确保排查现场问题
- 使用cglib等库的时候  会产生大量的类  会撑爆永久代导致oom
- 类加载后一般都放在永久代

## action
- 新建对象时候要是eden区放不下，会到fromto区，要是对象很大就会直接分配在老年代的空间中 就会多触发fullgc
