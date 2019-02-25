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

    Exception in thread "main" java.lang.StackOverflowError
                    at java.util.concurrent.ConcurrentHashMap.get(ConcurrentHashMap.java:936)
                    at java.util.concurrent.ConcurrentHashMap.containsKey(ConcurrentHashMap.java:964)
                    at org.apache.logging.slf4j.SLF4JLoggerContext.getLogger(SLF4JLoggerContext.java:40)
                    at org.apache.logging.slf4j.Log4jLoggerFactory.newLogger(Log4jLoggerFactory.java:37)
                    at org.apache.logging.slf4j.Log4jLoggerFactory.newLogger(Log4jLoggerFactory.java:29)
                    at org.apache.logging.log4j.spi.AbstractLoggerAdapter.getLogger(AbstractLoggerAdapter.java:47)
                    at org.apache.logging.slf4j.Log4jLoggerFactory.getLogger(Log4jLoggerFactory.java:29)
                    at org.slf4j.LoggerFactory.getLogger(LoggerFactory.java:284)
        

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

# 扩展

  log4j2配置示例
    
    <?xml version="1.0" encoding="UTF-8"?>
    
    <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
    <!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出 -->
    <!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数 -->
    <configuration status="WARN" monitorInterval="30">
    
    	<!-- 日志文件目录、压缩文件目录、日志格式配置 -->
    	<properties>
    		<Property name="fileName">/export/Logs/order-interface</Property>
    		<Property name="log_pattern">%-d{yyyy-MM-dd HH:mm:ss SSS}-->[%t]--[%-5p]--[%c{1}:%L]--[%X{requestNo} %X{userId} %X{tenantId}] %m%n</Property>
    	</properties>
    
    	<!--先定义所有的appender -->
    	<appenders>
    		<!--这个输出控制台的配置 -->
    		<console name="Console" target="SYSTEM_OUT">
    			<!--输出日志的格式 -->
    			<PatternLayout pattern="${log_pattern}" />
    		</console>
    
    		<!-- 这个会打印出所有的debug及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
    		<RollingFile name="RollingFileDebug" fileName="${fileName}/debug.log" filePattern="${fileName}/$${date:yyyy-MM}/debug-%d{yyyy-MM-dd}-%i.log">
    			<!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
    			<ThresholdFilter level="debug" onMatch="ACCEPT"
    							 onMismatch="DENY" />
    			<PatternLayout pattern="${log_pattern}" />
    			<Policies>
    				<TimeBasedTriggeringPolicy />
    				<SizeBasedTriggeringPolicy size="100 MB" />
    			</Policies>
    		</RollingFile>
    
    		<!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
    		<RollingFile name="RollingFileInfo" fileName="${fileName}/info.log" filePattern="${fileName}/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
    			<!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
    			<ThresholdFilter level="info" onMatch="ACCEPT"
    				onMismatch="DENY" />
    			<PatternLayout pattern="${log_pattern}" />
    			<Policies>
    				<TimeBasedTriggeringPolicy />
    				<SizeBasedTriggeringPolicy size="100 MB" />
    			</Policies>
    		</RollingFile>
    
    		<RollingFile name="RollingFileWarn" fileName="${fileName}/warn.log"
    			filePattern="${fileName}/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
    			<ThresholdFilter level="warn" onMatch="ACCEPT"
    				onMismatch="DENY" />
    			<PatternLayout pattern="${log_pattern}" />
    			<Policies>
    				<TimeBasedTriggeringPolicy />
    				<SizeBasedTriggeringPolicy size="100 MB" />
    			</Policies>
    			<!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
    			<DefaultRolloverStrategy max="20" />
    		</RollingFile>
    
    		<RollingFile name="RollingFileError" fileName="${fileName}/error.log"
    			filePattern="${fileName}/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
    			<ThresholdFilter level="error" onMatch="ACCEPT"
    				onMismatch="DENY" />
    			<PatternLayout pattern="${log_pattern}" />
    			<Policies>
    				<TimeBasedTriggeringPolicy />
    				<SizeBasedTriggeringPolicy size="100 MB" />
    			</Policies>
    		</RollingFile>
    	</appenders>
    
    	<!--然后定义logger，只有定义了logger并引入的appender，appender才会生效 -->
    	<loggers>
    		<!--过滤掉spring和mybatis的一些无用的DEBUG信息 -->
    		<logger name="org.mybatis" level="INFO"></logger>
    		<logger name="com.jd.scc.sdk.config" level="debug"></logger>
    		<root level="info">
    			<appender-ref ref="Console" />
    			<appender-ref ref="RollingFileDebug" />
    			<appender-ref ref="RollingFileInfo" />
    			<appender-ref ref="RollingFileWarn" />
    			<appender-ref ref="RollingFileError" />
    		</root>
    	</loggers>
    </configuration>
