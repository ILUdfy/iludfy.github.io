---
layout: '解决failed to configure a datasource ‘url‘ attribute is not specified'
title: '解决failed to configure a datasource ‘url‘ attribute is not specified'
date: 2023-03-31 15:18:40
tags: 
 - spring boot
 - spring boot错误记录
 - Maven
categories: spring boot
---
# 解决failed to configure a datasource ‘url‘ attribute is not specified

今天做项目的时候，jar包启动报了数据源url找不到的错误：
```
***************************
APPLICATION FAILED TO START
***************************
 
Description:
 
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
 
Reason: Failed to determine a suitable driver class
 
 
Action:
 
Consider the following:
  If you want an embedded database (H2, HSQL or Derby), please put it on the classpath.
  If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).
 
 
Process finished with exit code 1
```
报错信息提示找不到数据源url，但是迷惑的是，在application.properties文件下确实配置了，也没有问题：
```
spring.datasource.url=jdbc:mysql://localhost:3306/AIO
spring.datasource.username=root
#spring.datasource.password=Shark666@nju
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
所以考虑是不是application.properties在打包时没有加载到classes目录下的问题，一看果然：
![img](./application.png)
那么原因在哪里，为什么加载不出来？
经过排查，找到之前引入外部jar包作为库的时候，在pom.xml文件中加了一段代码：
```
        <resources>
            <resource>
                <directory>${project.basedir}/lib</directory>
                <targetPath>/BOOT-INF/lib/</targetPath>
                <includes>
                    <include>**/*.jar</include>
                </includes>
            </resource>
        </resources>
```
虽然不清楚这个配置的具体原理，不过想到spring有一些其他的配置，和这个include类似，不写在include内的其他东西会被排除掉，所以加了一段代码：
```
        <resources>
            <resource>
                <directory>${project.basedir}/lib</directory>
                <targetPath>/BOOT-INF/lib/</targetPath>
                <includes>
                    <include>**/*.jar</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <targetPath>BOOT-INF/classes/</targetPath>
                <includes>
                    <include>**/*</include>
                </includes>
            </resource>
        </resources>
```
再次打包就正常了。