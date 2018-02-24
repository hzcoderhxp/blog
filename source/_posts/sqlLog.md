---
title: SQL LOG问题
date: 2018-02-18 18:07:18
tags:
  - tech
  - log

---

### 背景
公司项目发现一个问题：应用的事务居然没有开启@Transactional居然是没用的。经过排查发现在配置事务的时候，下面这段代码作怪

```java
public Log4jdbcProxyDataSource getLog4jdbcProxyDataSource(DataSource dataSource){
        return new Log4jdbcProxyDataSource(dataSource);
    }
```


应用中的SessionFactory和DataSourceTransactionManager在设置数据源对象的时候都会调用上面这段代码，导致它们引用的其实是两个新的Log4jdbcProxyDataSource；所以使事务失效了。

### 解决
事务问题解决很简单 就是不调用这个getLog4jdbcProxyDataSource方法，直接用dataSource来spring初始化。

### 延伸

顺便连带出来的是一个日志问题，如何在应用中打印出执行的SQL,这个问题去百度的话，五花八门，要么断章取义，要么完全复制。我们是使用logback mybatis（别的未亲测），解决方法就两种

- mybatis自带一个sql 输出的功能；该方法打印出的SQL是带问号的比较普通的形式
```xml
<logger name="com.stone.service.core.repository" level="DEBUG"/>
```

- 使用log4jbdc，需要改驱动和URL前加上log4jdbc:  这种方式打印出来SQL会有表格形状；如下

```xml
jdbc:log4jdbc:mysql:
```


```xml
driver-class-name: net.sf.log4jdbc.DriverSpy
```

### 小结
至于选择根据各自项目特点来吧。。。
