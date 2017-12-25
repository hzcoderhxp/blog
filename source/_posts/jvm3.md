---
title: jvm学习(gc参数)
date: 2017-11-24 10:07:18
tags:
  - tech
  - jvm

---

# jvm学习(gc参数)

## 串行收集器
> 效率高 最古老最稳定 线程只有一个 可能会产生较长停顿
- -XX:+UseSerialGC
- 新生代、老年代使用串行回收 对应复制算法  标记压缩
- 引用计数法 (引用)
## 并行收集器
- XX:+UseParNewGC
- 新生代并行 老年代串行
- 多线程需要多核支持
- -XX:ParallelGCThreads限制线程数量
## 并行收集器2
- 类似ParNew
- 更加关注吞吐量
- -XX:+UseParallelGC 新生代并行 老年代串行
- -XX:+UseParallelOldGC 新生代并行 老年代并行
  ### 并行收集器两个参数
  - -XX:MaxGCPauseMills最大停顿时间
  -	-XX:GCTimeRatio占CPU的时间占比
  -	两个参数矛盾不可能同时调优
## CMS收集器
- Concurrent Mak Sweep 并发标记清除
- 标记清除算法
- 并发阶段会降低吞吐量
- *会和应用程序一起执行
- 停顿会比较少
- -XX:+UseConcMarkSweepGC
  ### 过程
  - 初始标记
  - 并发标记(和用户线程一起)
  - 重新标记
  - 并发清除(和用户线程一起)
  ### 特点
  -	可能降低停顿
  -	会影响系统整体吞吐量和性能
  - 清理不彻底
  -	因为和用户线程一起运行，不能在哦你估计快满时清理
  - -XX:CMSInitiatingOccupancyFraction设置GC阀值 
  - 怕内存不够concurrent mode failure错误
## 内存碎片
- -XX+UseCMSCompactAtFullCollection 会停顿时间边长
- -XX:+CMSFullGCBeforeCompaction 设置几次fullgc以后进行碎片整理
- -XX:ParallCMSThreads
 