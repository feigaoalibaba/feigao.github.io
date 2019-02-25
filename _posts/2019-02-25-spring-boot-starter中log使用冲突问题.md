---
layout: post
title: spring-boot-starter中log使用冲突问题
tags:
- spring-boot-starter-logging
- spring-boot-starter-log4j2
---
# 背景

    最近使用boot2.0.6 开发的服务，通过jar包启动服务正常。在上线时改成war类型进行发布，遇到一些问题。
    其中log互相依赖导致 StackOverflowError的问题比较头疼，特此进行说明，并提供了解决方案。
    

# 问题 

    ```java
            Exception in thread "main" java.lang.StackOverflowError
                at java.util.concurrent.ConcurrentHashMap.get(ConcurrentHashMap.java:936)
                at java.util.concurrent.ConcurrentHashMap.containsKey(ConcurrentHashMap.java:964)
                at org.apache.logging.slf4j.SLF4JLoggerContext.getLogger(SLF4JLoggerContext.java:40)
                at org.apache.logging.slf4j.Log4jLoggerFactory.newLogger(Log4jLoggerFactory.java:37)
                at org.apache.logging.slf4j.Log4jLoggerFactory.newLogger(Log4jLoggerFactory.java:29)
                at org.apache.logging.log4j.spi.AbstractLoggerAdapter.getLogger(AbstractLoggerAdapter.java:47)
                at org.apache.logging.slf4j.Log4jLoggerFactory.getLogger(Log4jLoggerFactory.java:29)
                at org.slf4j.LoggerFactory.getLogger(LoggerFactory.java:284)
    ```

# 原因

    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
    
    和 
    
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
    
    这两个分别使用了 log4j-to-slf4j-2.3.jar  和 log4j-slf4j-impl-2.3.jar

    you're creating a call-loop with log4j-slf4j-impl-2.3.jar and log4j-to-slf4j-2.3.jar.
    
    log4j-slf4j-impl-2.3.jar is the implementation of the adapter that sends slf4j calls to log4j.
    
    log4j-to-slf4j-2.3.jar is sending log4j calls right back to slf4j. 
    
    解决方案：Remove this one.
    
    

# 项目中使用方式

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
