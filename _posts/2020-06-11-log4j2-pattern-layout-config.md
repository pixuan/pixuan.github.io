---
layout: post
title:  "springboot project log4j2.xml pattern layout beautify"
date:   2020-06-1 18:00:00 +0800
categories: log4j2 pattern springboot 
---

### log4j2.xml
1、为了springboot日志查看方便，参考默认自带的日志输出格式，即DefaultLogbackConfiguration类的CONSOLE_LOG_PATTERN，设置日志格式化的pattern
2、最终格式化配置为
{% raw %}
```xml
<PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5p %pid - [%15.15t] %-40.40logger{39} : %m%n{%wEx}" />
```
{% endraw %}
包含：时间/level/pid/线程名/logger名/日志内容/换行符  
3、最终效果如下，相当整齐，便于查看
```
...
2020-06-11 18:00:00.000 INFO  9999 - [           main] .test.Application               : this is test application log 
2020-06-11 18:00:00.001 DEBUG 9999 - [p-nio-80-exec-1] .test.controller.TestController : this is test controller log 
...
```
