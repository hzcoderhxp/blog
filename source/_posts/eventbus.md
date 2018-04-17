---
title: google eventbus线程问题
date: 2018-03-23 17:47:19
tags:
  - tech
  - eventbus
---


### 背景
 公司最近在合同批量存档的时候发现速度很慢，发现以前对google eventbus 存在误解，特此记录一下

```java
@Configuration
public class EventConfig {
    @Autowired
    private ApplicationContext applicationContext;
    public static ThreadPoolTaskExecutor eventBusExecutorService= new ThreadPoolTaskExecutor();
    @Bean
    public EventBus eventBus(){
        //设置了10个核心线程数
        eventBusExecutorService.setCorePoolSize(16);
        //最大线程数量为100
        eventBusExecutorService.setMaxPoolSize(100);
        eventBusExecutorService.setThreadGroupName("event_bus_thread_group");
        eventBusExecutorService.setWaitForTasksToCompleteOnShutdown(true);
        eventBusExecutorService.setAwaitTerminationSeconds(10);
        eventBusExecutorService.initialize();
        //异步处理方式
        EventBus eventBus= new AsyncEventBus(eventBusExecutorService);
        Map<String,IListener> listenerMap=applicationContext.getBeansOfType(IListener.class);
        listenerMap.keySet().stream()
                .peek(item->eventBus.register(listenerMap.get(item)))
                .collect(Collectors.toList());
        return eventBus;
    }
}
```
### 分析
看上面这个代码的话，比较普通配置，以前线上都是在用这套东西，并且都是比较稳定的情况。因为这次是批量上传，写个伪代码说明下

- 事件发布类
```java
public class PostEventClass {
    //事件工具类
    private eventBusHelper;

    public void method(){

        for(需要处理的数据){
            //发布指定订单号的事件
            eventBusHelper.post(new UplaodEvent(orderId));
        }

    }

}
```

- 事件订阅类
```java
public class SubscribeEventClass {
    

    @Subscribe
    public void deal(UploadEvent e){

    //TODO 业务操作（因为这里涉及很多生成盖章 校验之类的所以比较耗时）

    }

}
```

看上面的代码，以前本来想我扔了10000个事件出来，那这个线程池会一起去消费我的这些上传合同事件的任务（当然是在没有别的事件消费的情况下）；然而事与愿违，从后台日志看到，虽然有这么多线程，不过实际并发执行的永远都是一个线程在处理。正是这个原因，没有达到批量去上传的效果

针对这个情况写了测试EventBus的测试类看它的消费情况，发布事件类略，代码如下

```java
public class SubscribeTestEventClass {
    

    @Subscribe
    public void deal(TestEvent e){

    //TODO 业务操作（因为这里涉及很多生成盖章 校验之类的所以比较耗时）
    logger.info(Thread.currentThread().getName()+"======START1===="+DateUtil.format(new Date(), "yyyy-MM-dd HH:mm:ss sss"));
    	Thread.sleep(5000);
    	logger.info(Thread.currentThread().getName()+"======END1===="+DateUtil.format(new Date(), "yyyy-MM-dd HH:mm:ss sss"));
    }

}
```

看下日志

```log
ThreadPoolTaskExecutor-7=====START1====2018-03-23 05:22:56 056
ThreadPoolTaskExecutor-7=====END1====2018-03-23 05:23:01 001
ThreadPoolTaskExecutor-3=====START1====2018-03-23 05:23:01 001
ThreadPoolTaskExecutor-3======END1====2018-03-23 05:23:06 006
ThreadPoolTaskExecutor-3=====START1====2018-03-23 05:23:06 006
ThreadPoolTaskExecutor-3======END1====2018-03-23 05:23:11 011
```
从日志中可以看出其实，消费事件永远都是一个线程在执行。好了现在发现了问题所在了，那大家肯定会想，那设置这个线程池有什么用呢，其实这个单线程执行是针对一个订阅者来说的。而线程池是针对所有订阅该事件的订阅者来说的

继续看代码
```java
public class Subscribe2testClass{
    @Subscribe
    public void deal1(TestEvent event) throws InterruptedException {

    	logger.info(Thread.currentThread().getName()+"======START1===="+DateUtil.format(new Date(), DateUtil.FORMAT_DATETIME));
    	Thread.sleep(5000);
    	logger.info(Thread.currentThread().getName()+"======END1===="+DateUtil.format(new Date(), DateUtil.FORMAT_DATETIME));

    }
    
    @Subscribe
    public void deal2(TestEvent event) throws InterruptedException {

    	logger.info(Thread.currentThread().getName()+"======START2===="+DateUtil.format(new Date(), DateUtil.FORMAT_DATETIME));
    	Thread.sleep(5000);
    	logger.info(Thread.currentThread().getName()+"======END2===="+DateUtil.format(new Date(), DateUtil.FORMAT_DATETIME));

    }
    
    @Subscribe
    public void deal3(TestEvent event) throws InterruptedException {
    	logger.info(Thread.currentThread().getName()+"======START3===="+DateUtil.format(new Date(), DateUtil.FORMAT_DATETIME));
    	Thread.sleep(5000);
    	logger.info(Thread.currentThread().getName()+"======END3===="+DateUtil.format(new Date(), DateUtil.FORMAT_DATETIME));

    }
} 
```

再看下日志,三个订阅者是并发消费了
```log
ThreadPoolTaskExecutor-10====START2====2018-03-23 05:19:51 051
ThreadPoolTaskExecutor-6====START3====2018-03-23 05:19:51 051
ThreadPoolTaskExecutor-1====START1====2018-03-23 05:19:51 051
```


### 总结
针对这个google eventbus的这个情况，eventbus 提供了一个并发发布的方式，只要在订阅的方法上加上注解@AllowConcurrentEvents 就可以解决这个问题。因为查看eventbus源码可以看到，一个subscriber有一个synchronized(this)的同步机制。希望大家注意下，一般线上项目的话，抛出事件消费，不大会发现这个问题。碰到批量操作还是老老实实的建个线程池吧。。



