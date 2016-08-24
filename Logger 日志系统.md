---
title: Logger 日志系统
tags: 
grammar_cjkRuby: true
---

[TOC]


##  slf4j + log4j

[Log4j 日志配置示例详解][1]

slf4j-log4j12 用作 slf4j 和 log4j 的桥接器

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.21</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.21</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

log4j 中需要配置记录的级别,只有代码中级别高的 log 才会打印
级别依次从高到底:
- FATAL 0
- ERROR 3
- WARN 6
- INFO 6
- DEBUG 7

```
log4j.rootLogger=ERROR,console

log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%c %m%n
```

##  slf4j + logback

[logback logback.xml常用配置详解（一）<configuration> and][2]

所需包 slf4j,logback-core,logback-classic

**根节点<configuration>包含的属性：**
 
- scan:
当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。

- scanPeriod:
设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。

- debug:
当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <contextName>myAppName</contextName>  
      <!-- 其他配置省略-->  
</configuration>  
```

**loger**
```xml
<configuration>   
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
    <!-- encoder 默认配置为PatternLayoutEncoder -->   
    <encoder>   
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>   
    </encoder>   
  </appender>   
  <root level="INFO">             
    <appender-ref ref="STDOUT" />   
  </root>     
</configuration>  
```
**打印结果:**
```
13:30:38.484 [main] INFO  logback.LogbackDemo - ======info  
13:30:38.500 [main] WARN  logback.LogbackDemo - ======warn  
13:30:38.500 [main] ERROR logback.LogbackDemo - ======error  
```


##  slf4j binding 问题
由于多个模块之间,各自使用的 slf4j 的实现不一样,出现 slf4j 重复绑定,解决方法
在由冲突的包中取出 slf4j-log4j12 引用,引入的包也会是同一个格式输出

```xml
<dependency>
    <groupId>com.jqk</groupId>
    <artifactId>people</artifactId>
    <version>1.0-SNAPSHOT</version>
    <exclusions>
        <exclusion>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
         </exclusion>
    </exclusions>
</dependency>
```
通过查找依赖关系,在引入包的时候,用 `exclusion` 标签去除冲突

![enter description here][3]


  [1]: http://blog.csdn.net/ithomer/article/details/8000466
  [2]: http://aub.iteye.com/blog/1101260
  [3]: ./images/1472026856121.jpg "1472026856121.jpg"
