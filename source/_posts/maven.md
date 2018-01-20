
---
title: Hello Docker
date: 2017-10-18 10:07:18
tags:
  - tech
  - maven

---
Maven基础
> 背景：MAVEN作为时下最流行的项目管理工具，设计思想比它的实际使用给人更大的启发和思考。一直没有时间好好整理一个maven的文档，现在做一个整理，也分享给自己和同事，文章包括Maven的基本概念和操作


## Maven应用

- maven clean 清除target maven-clecn-plugin插件jar执行 以下雷同
- maven complie 编译成class文件
- maven test 对@Test单元测试 类名xxxxTest.java
- maven package 打包命令
- maven install 打包到本地仓库 供多个项目使用 

## Maven项目的生命周期

### 三套生命周期 每一套生命周期相互独立 互不影响;在一套生命周期内执行一个命令，前面的都会自动执行
- CleanLifeCycle 清理生命周期

  clean

- defaultLifeCycle默认生命周期

  
  compile test package install deploy


- siteLifeCycle 站点生命周期

## maven整合web项目案例

### eclipse 安装maven插件
1. m2e mars2版本自带
2. preferences - maven - installations 配置maven路径
3. userSetting，让eclipse知道仓库位置
4. 构建索引 windows - show view - local responsity - rebuild index

## maven 整合servlet

1. maven project 构建普通maven 或者 父工程
2. maven module maven模块 隶属于maven父工程
3. Version : shotsnap(快照 测试)/releases(正式)
4. New Maven Project Packaging 选择jar(java工程)/war(web工程)/pom(父工程)
5. 缺失web.xml 要加一个
6. 原来jdk是1.5，通过pom.xml中添加编译插件来设置jdk编译版本为1.8

``` xml
<build>
    <!-- 工程文件名 -->
    <finalName>stone-api</finalName>
    <plugins>
    <!-- springboot 插件 -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <!-- jdk版本 插件 -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
```
7. 创建servlet编译报错 缺失servlet-api-xx.jar
8. 查找依赖 add dependency
9. 依赖范围 <默认就是complie> 

依赖范围 | 编译 | 测试 | 运行 | 例子
--------|-----|----- | --- | ----
compile | Y  | Y | Y | spring-code
test | -  | Y | - | junit
provided | Y  | Y | - | servlet-api
runtime | -  | Y | Y | jdbc驱动
system | Y  | Y | - | maven依赖以外jar


10. servlet-api会打进去war中  和tomcat中自带的servlet-api冲突.所以需要将 servlet-api 改成 provided (类似还有jsp-api)如下

``` xml
<dependency>
      <groupId>javax-servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <!-- 设置作用范围 -->
      <scope>provided</scope>
</dependency>
```

11. 运行项目(于 tomcat server无关 stopped) 
- run as - maven build... - tomcat:run
12. debug运行 
- debug as - maven build... -tomcat:run
- source - add project 添加源码 这样才能进得了断点

## Maven 依赖传递和冲突解决
- maven自我调解原则 
1. 第一申明者优先原则（先申明先优先）
2. 路径近者优先原则(直接依赖高于传递依赖)
- 排除依赖原则
```xml
<dependency>
      <groupId>org.apache.struts</groupId>
      <artifactId>struts2-spring-plugin</artifactId>
      <version>2.3.24</version>
      <!-- 排除依赖 -->
      <exclusions>
        <exclusion>
            <artifactId>spring-beans</artifactId>
            <groupId>org.springframework</groupId>
        </exclusion>
      </exclusions>
</dependency>
```

- 版本锁定(推荐使用)
```xml
<!-- 版本锁定 -->
<dependencyManagement>
    <dependencies>
        <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>4.2.4.RELEASE</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

```xml
  <!-- 版本管理 -->
  <properties>
    <java.version>1.8</java.version>
    <start-class>com.stone.Application</start-class>
    <mybatis.version>3.3.1</mybatis.version>
    <mapper.version>3.3.6</mapper.version>
    <pagehelper.version>4.1.1</pagehelper.version>
    <springfox.version>2.6.1</springfox.version>
    <jackson.version>2.8.9</jackson.version>
  </properties>
```

## maven 项目拆分

父工程 ssh-parent
子模块 ssh-service
子模块 ssh-dao
子模块 ssh-web
1. 创建父工程(packaging 选择pom) 只有src . pom.xml文件

    - 项目需要恩依赖信息 在父工程中定义 子模块继承
    - 将各个子模块聚合到一起

2. 将创建的父工程发布到本地仓库    
3. 创建dao子模块(packaging 选择jar)：包含dao代码和相应配置文件
4. 配置文件拆分 applicationContext-basic.xml 项目基础信息 applicationContext-dao.xml dao配置
5. 创建service子模块(packaging 选择jar): 包含service代码 和对应配置
6. 创建web子模块(packaging 选择war)
7. web模块中 web.xml filter 改成classpath*:applicationContext-*.xml

## maven 单元测试
- maven junit
1. @RunWith(SpringJunitClassRunner.class)
2. @ContextConfiguration("classpath:spring/applicationContext-*.xml")
- 传统 junit
1. new ClassPathXmlApplicationContext（"classpath*:spring/applicationContext-*.xml"）

- 配置文件加载区别

1. 加载本项目配置：classpath:spring/applicationContext-*.xml

2. 批量加载jar中配置：classpath:spring/applicationContext-*.xml

> 传递依赖范围: 传递不是无休止的。当发现依赖缺失的时候。自己手动加个

## maven 运行方式

1. 运行父工程。父工程将各个模块打包成war包
2. 直接运行web工程
3. 部署tomcat中（不用关联源码也能打断点）

## maven 私服
- 安装
1. 下载安装包
2. nexus install安装
3. nexus start启动
4. conf中找到properties中的私服url
5. 设置用户名 密码等

- 私服仓库类型
1. Hosted:宿主仓库 
    - releases正式 
    - snapshot快照 
    - 3rd party第三方(存在版权问题的)
2. Proxy:代理仓库   
    - 代理中央仓库
    - apache 测试版本jar包
3. group 组仓库
    - hosted 宿主仓库
    - proxy 代理仓库

- 上传本地jar包到私服
1. setting.xml中设置用户名密码
```xml
    <servers>
        <server>
        <id>releases</id>
        <username>admin</username>
        <password>admin</password>
        </server>  
    --
        <server>
        <id>snapshot</id>
        <username>admin</username>
        <password>admin2016</password>
        </server>
    </servers>
```    
2. 在将要上传的项目 pom.xml中配置jar包上传路径
```xml
<!-- stone-interface -->
<distributionManagement>
        <snapshotRepository>
            <id>snapshots</id>
            <url>http://repository.manage.imbird.cn/content/repositories/snapshots/</url>
        </snapshotRepository>
        <repository>
            <id>thirdparty</id>
            <url>http://repository.manage.imbird.cn/content/repositories/thirdparty/</url>
        </repository>
    </distributionManagement>
```
3. 执行命令发布项目到私服（上传deploy）

- 下载私服上jar到本地
1. 在setting.xml中配置模板
```xml
<profiles>
  <profile>
    <id>development</id>
    <!-- aliyun仓库地址 非私服内容 -->
    <repositories>
       <repository>
        <id>aliyun</id>
        <name>aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      </repository>
      
      <repository>
        <id>stone</id>
        <name>stone</name>
        <url>http://repository.manage.imbird.cn/content/groups/public/</url>
      </repository>
    </repositories>
    <pluginRepositories>
      <pluginRepository>
        <id>releases</id>
        <url>http://releases</url>
        <releases><enabled>true</enabled></releases>
        <snapshots><enabled>true</enabled></snapshots>
      </pluginRepository>
    </pluginRepositories>
  </profile>
  </profiles>
```

2. 激活模板
```xml
<activeProfiles>
    <activeProfile>development</activeProfile>
  </activeProfiles>
```

