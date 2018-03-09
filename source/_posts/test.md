---
title: 测试问题
date: 2018-03-08 08:07:18
tags:
  - tech
  - log

---

### 背景
公司项目发现一个问题 应用的事务居然没有开启@Transactional居然是没用的。经过排查发现在配置事务的时候，下面这段代码作怪

```java
public Log4jdbcProxyDataSource getLog4jdbcProxyDataSource(DataSource dataSource){
        return new Log4jdbcProxyDataSource(dataSource);
    }
```
