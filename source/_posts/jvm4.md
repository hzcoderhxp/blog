---
title: jvm学习(class装载)
date: 2017-11-24 12:07:18
tags:
  - tech
  - jvm

---

# jvm学习(class装载)

## 类加载过程
### 加载	
1. 取得类的二进制流
2. 转为方法区数据结构
3. 在java堆中生成对应的java.lang.Class对象
### 连接
连接->验证 文件格式 元数据 字节码验证 符号引用验证
### 初始化
1. 准备 分配内存 设置初始值	
2. 解析 符号引用转为直接引用
3. 执行类构造器 static变量赋值 static{}语句 clinit是线程安全nosuchfieldexception?
##  类加载器ClassLoader
- 是个抽象类
- 读入java字节码将类装载jvm中
- 可以定制
### 系统中的ClassLoader

#### 分类
- BootStrap ClassLoader 启动	 rt.jar /-Xbootclasspath
- Extension ClassLoader 扩展	 java_home/lib/ext/*.jar
- APP ClassLoader	      应用(自己写的类都加载这里)	classpath下
- Custom ClassLoader    自定义
- 每一个loader都有一个parent父亲 除了bootstrap这个是最早的
#### 特点
- 自底向上检查是否加载指定class，假如上面有了  请求就上传到上面的classloader
- 自上向下加载
- 自下向上查找	
- 这种叫双亲加载 parent	
- orderclassloader也可以突破双亲模式	
> Thread.setContextClassLoader()
		上下文加载器
		是一个角色
		解决顶层classloader无法访问底层的classloader的类的问题
		基本思想 再顶层classloader中传入底层classloader的实例
		classloader可以突破双亲模式的局限性

## action
欢迎大家访问<a href="https://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html" target="_blank">[转]JVM系列三:JVM参数设置、分析</a>        
	

