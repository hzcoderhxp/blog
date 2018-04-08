---
title: 聊聊AOP
date: 2018-04-08 15:47:19
tags:
  - tech
  - AOP
---

### AOP前言
    AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。其实这个概念是相对于OOP（Object Oriented Programming）来说的。也可以说是这种编程的类型、思想、范式。

### 场景
    当有一个需要需要在某一类请求，或者某一类执行方法中通用的增加一个功能点，比较普通的例子 日志、安全校验、签名验证、权限控制、事务处理等等。假如我把前面这个需求命题改成一个特定方法，可能很多人会直接在方法代码快里加上特定的增强功能（日志为例）。伪代码：
```java
public class SayHelloService {
    public void say(){
        System.out.print("Hello world");
        //TODO log info (增强功能)
    }
} 
```    
当我需要对一批方法都加日志，就有了AOP的诞生了
### 设计
    要达到AOP的效果，很多人应该会想到设计模式中的代理模式，事实也是如此。你理解了代理模式也就理解了AOP。AOP 代理则可分为静态代理和动态代理两大类，其中静态代理是指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强；而动态代理则在运行时借助于 JDK 动态代理、CGLIB 等在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。下面分别介绍下
### 静态代理AspectJ   
    AspectJ 是 Java 语言的一个 AOP 实现，其主要包括两个部分：第一个部分定义了如何表达、定义 AOP 编程中的语法规范，通过这套语言规范，我们可以方便地用 AOP 来解决 Java 语言中存在的交叉关注点问题；另一个部分是工具部分，包括编译器、调试工具等。AspectJ是一套独立的面向切面编程的解决方案。（于spring 无关）,看个简单应用demo

- AspectJ 下载地址(http://www.eclipse.org/aspectj/downloads.php)。
- AspectJ HelloWorld 实现
```java
业务组件  SayHelloService
public class SayHelloService {
    public void say(){
        System.out.print("Hello  AspectJ");
    }
} 
//在需要在调用say()方法之后，需要记录日志。那就是通过AspectJ的后置增强吧。
```
```java
public aspect LogAspect{
    pointcut logPointcut():execution(void SayHelloService.say());
    after():logPointcut(){
         System.out.println("记录日志 ..."); 
    }
}

```
- 编译SayHelloService
```bash
执行命令   ajc -d . SayHelloService.java LogAspect.java
生成 SayHelloService.class
执行命令    java SayHelloService
输出  Hello AspectJ  记录日志
```
注意：ajc.exe 可以理解为 javac.exe 命令，都用于编译 Java 程序，区别是 ajc.exe 命令可识别 AspectJ 的语法；我们可以将 ajc.exe 当成一个增强版的 javac.exe 命令.执行ajc命令后的 SayHelloService.class 文件不是由原来的 SayHelloService.java 文件编译得到的，该 SayHelloService.class 里新增了打印日志的内容——这表明 AspectJ 在编译时“自动”编译得到了一个新类，这个新类增强了原有的 SayHelloService.java 类的功能，因此 AspectJ 通常被称为编译时增强的 AOP 框架。

### 动态代理(JAVA动态代理、CGLIB)
与 AspectJ 相对的还有另外一种 AOP 框架，它不需要在编译时对目标类进行增强，而是运行时生成目标类的代理类，该代理类要么与目标类实现相同的接口，要么是目标类的子类——总之，代理类的实例可作为目标类的实例来使用。一般来说，编译时增强的 AOP 框架在性能上更有优势——因为运行时动态增强的 AOP 框架需要每次运行时都进行动态增强。

- JAVA动态代理
现在要生成某一个对象的代理对象，这个代理对象通常也要编写一个类来生成，所以首先要编写用于生成代理对象的类。在java中如何用程序去生成一个对象的代理对象呢，java在JDK1.5之后提供了一个"java.lang.reflect.Proxy"类，通过"Proxy"类提供的一个newProxyInstance方法用来创建一个对象的代理对象，如下所示：
```java
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) 
```
参数一：指定目标对象的类加载器
参数二：目标对象的实现接口
参数三：invoke目标对象时候的方法增强

- CGLIB代理
CGLIB（Code Generation Library），简单来说，就是一个代码生成类库。它可以在运行时候动态是生成某个类的子类。
下面先为 CGLIB 提供一个拦截器实现类：
```java
public class AroundAdvice implements MethodInterceptor 
{ 
public Object intercept(Object target, Method method 
, Object[] args, MethodProxy proxy) 
throws java.lang.Throwable 
{ 
System.out.println("执行目标方法之前，模拟开始事务 ..."); 
// 执行目标方法，并保存目标方法执行后的返回值
Object rvt = proxy.invokeSuper(target, new String[]{"被改变的参数"}); 
System.out.println("执行目标方法之后，模拟结束事务 ..."); 
return rvt + " 新增的内容"; 
} 
}
```
接下来程序提供一个 ChineseProxyFactory 类，这个 ChineseProxyFactory 类会通过 CGLIB 来为 Chinese 生成代理类：
```java
public class ChineseProxyFactory 
{ 
public static Chinese getAuthInstance() 
{ 
Enhancer en = new Enhancer(); 
// 设置要代理的目标类
en.setSuperclass(Chinese.class);
// 设置要代理的拦截器
en.setCallback(new AroundAdvice());
// 生成代理类的实例 
return (Chinese)en.create();
} 
}
```
- 小结：JAVA 动态代理有一个限制是必须实现接口，反正无法代理。所以实际很多情况我们会使用cglib
### 浅谈SPRING AOP
Spring AOP也是对目标类增强，生成代理类。但是与AspectJ的最大区别在于---Spring AOP的运行时增强，而AspectJ是编译时增强。
曾经以为AspectJ是Spring AOP一部分，是因为Spring AOP使用了AspectJ的Annotation。使用了Aspect来定义切面,使用Pointcut来定义切入点，使用Advice来定义增强处理。虽然使用了Aspect的Annotation，但是并没有使用它的编译器和织入器。其实现原理是JDK 动态代理，在运行时生成代理类。
为了启用 Spring 对 @AspectJ 方面配置的支持，并保证 Spring 容器中的目标 Bean 被一个或多个方面自动增强，必须在 Spring 配置文件中添加如下配置
```xml
<aop:aspectj-autoproxy/>
在springboot中对应 @EnableAspectJAutoProxy(默认打开true)
```

当需要强制使用cglib时候需要加上
```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
 在springboot中对应 spring.aop.proxy-target-class: true
```

当启动了 @AspectJ 支持后，在 Spring 容器中配置一个带 @Aspect 注释的 Bean，Spring 将会自动识别该 Bean，并将该 Bean 作为方面 Bean 处理。方面Bean与普通 Bean 没有任何区别，一样使用 <bean.../> 元素进行配置，一样支持使用依赖注入来配置属性值。
使用Spring AOP的改写 Hello World的例子。
```java
业务组件  SayHelloService
import org.springframework.stereotype.Component;
@Component
public class SayHelloService {
    public void say(){
        System.out.print("Hello  AspectJ");
    }
}
```
做后置增强的日志处理。
```java
@Aspect
@Component
public class LogAspect {
     @After("execution(* com.x.xxx.aspectj.learn.SayHelloService.*(..))")
     public void log(){
         System.out.println("记录日志 ...");
     }
}
```

### 总结 
AOP除了在以上说的业务层面的应用上，在RPC中也得到广泛应用。