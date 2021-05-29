---
layout: post
title:  "spring-boot-starter-logging日志配置"
date:   2021-05-28 14:29:36 +0800
categories: springboot
---

# spring-boot-starter-logging日志配置
以下所有内容均可在[springboot-demo](https://github.com/zph-programmer/springboot)中找到。
## 所需依赖
在pom.xml引入maven依赖，注意是否存在依赖冲突。
```xml
<dependency>
    <groupId>org.springframework.boot<groupId>
    <artifactId>spring-boot-starter-logging<artifactI>
</dependency>
```

## 默认配置文件logback.xml
在resources目录下添加logback.xml文件，指定日志输出格式和文件。这里将日志文件按天归档，并按100M切割文件。日志文件太大的话查看会很麻烦。另外我个人喜话将终端日志配置彩色效果，并使用ThresholdFilter将error级别的日志单独输出到一个文件，方便排查错误。
```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration scan="true">
    <property name="APP_NAME" value="SpringBootDemo"/>
    <property name="LOG_HOME" value="./log/${APP_NAME}"/>

    <!-- 彩色日志输出格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%yellow(%date{yyyy-MM-dd HH:mm:ss}) |%cyan(%-5level) |%blue(%thread) |%yellow(%file:%line) |%green(%logger) |%highlight(%msg%n)"/>

    <property name="CONSOLE_LOG_PATTERN2"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

    <property name="PATTERN" value="%d{yy-MM-dd.HH:mm:ss.SSS}|%X{invokeNo}|[%-16t] %-5p %-22c{0} - %m%n"/>
    <property name="CHARSET" value="UTF-8"/>

    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>${CHARSET}</charset>
        </encoder>
    </appender>
    <appender name="detailAppender" class="ch.qos.logback.core.rolling.RollingFileAppender" additivity="false">
        <File>${LOG_HOME}/${APP_NAME}_detail.log</File>
        <encoder>
            <pattern>${PATTERN}</pattern>
            <charset>${CHARSET}</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/${APP_NAME}_detail.%d{yyyyMMdd}.%i.log</fileNamePattern>
            <!-- 每天归档，最大100M,保存7天 -->
            <maxHistory>7</maxHistory>
            <maxFileSize>100MB</maxFileSize>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <appender name="errorAppender" class="ch.qos.logback.core.rolling.RollingFileAppender" additivity="false">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <File>${LOG_HOME}/${APP_NAME}_error.log</File>
        <encoder>
            <pattern>${PATTERN}</pattern>
            <charset>${CHARSET}</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/${APP_NAME}_error.%d{yyyyMMdd}.%i.log</fileNamePattern>
            <!-- 每天归档，最大100M,保存7天 -->
            <maxHistory>7</maxHistory>
            <maxFileSize>100MB</maxFileSize>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!-- TRACE < DEBUG < INFO < WARN < ERROR  -->
    <logger name="org.springframework" level="INFO"/>
    <!--> 指定输出的日志级别-->
    <root level="debug">
        <appender-ref ref="consoleAppender"/>
        <appender-ref ref="detailAppender"/>
        <appender-ref ref="errorAppender"/>
    </root>
</configuration>
```

## 使用@Slf4j和log
更多使用方法，请参看[logback官网文档](https://logback.qos.ch/manual/index.html)  

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
@Slf4j
public class SpringBootLogTest  extends SpringBootApplicationTests{
    @Test
    public void logTest() {
        //日志的级别；
        //由低到高   trace<debug<info<warn<error
        //可以调整输出的日志级别；日志就只会在这个级别及以后的高级别生效
        log.trace("这是trace日志...");
        log.debug("这是debug日志...");
        //SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别；root level
        log.info("这是info日志...");
        log.warn("这是warn日志...");
        log.error("这是error日志...");
    }
}
```
输出结果:可以看到由于logback中指定了root level="debug",输出了debug及以上级别的日志。不得不说，彩色日志真好看！可惜这个只能在idea的终端中显示颜色，普通的日志文件就不行了。
![logging](images/logging.png)



* [返回目录](https://zph-programmer.github.io)
    * [上一篇 —— idea社区版配置和插件](01-idea社区版配置和插件.md)
    * [下一篇 —— 使用AOP切面打印日志](03-使用AOP切面打印日志.md)