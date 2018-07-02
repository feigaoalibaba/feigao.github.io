---
layout: post
title: 利用Profile来构建不同环境的部署包
tags:
- maven
- scope
- profile
---

# 前言
   
   profile使得不同环境间构建的可移植性成为可能。Maven中的profile是一组可选的配置，可以用来设置或者覆盖配置默认值。有了profile，你就可以为不同的环境定制构建。
   profile可以在pom.xml中配置，并给定一个id。然后你就可以在运行Maven的时候使用的命令行标记告诉Maven运行特定profile中的目标。

# 问题
   在项目开发好后，通常要在多个环境部署。一般至少有几个环境：开发环境(dev),测试环境(test),预发布环境(pre),正式生成环境(prod)，每种环境都有各自的配置参数，
   比如：数据库连接、远程调用等等。如果每个环境build钱手动修改这些参数，显然不太方便。
   
   在项目开发中还会遇到以下场景：我们一个服务依赖一个cache中间件（以jar包形式），而这个cache中间件又有两种实现（阿里云cache实现、京东云cache实现）。我们的服务（同一套代码，面向cache接口开发）既要部署在阿里云上，又要部署到京东云上。
   这时会有个问题，部署到阿里云的代码，要依赖阿里云cache实现的jar包，不能引入京东云实现的jar包。部署到京东云同理。
   
   以上两个问题小结一下为：
   
   - 同一工程支持多环境部署引用不同的配置文件
      
   - 同一工程支持多环境部署引用不同的中间件jar包
   
# 多环境部署支持不同配置文件

## 项目pom.xml 配置定义
```java   
      <profiles>
           <profile>
               <id>dev</id>
               <activation>
                   <activeByDefault>true</activeByDefault>
               </activation>
               <properties>
                   <environment>dev</environment>
               </properties>
           </profile>
           <profile>
               <id>test</id>
               <properties>
                   <environment>test</environment>
               </properties>
           </profile>
           <profile>
               <id>prod</id>
               <properties>
                   <environment>prod</environment>
               </properties>
           </profile>
       </profiles>
       
       ...........
       
      <build>
              <resources>
                  <resource>
                      <directory>${project.basedir}/src/main/resources</directory>
                  </resource>
                  <resource>
                      <directory>${project.basedir}/src/main/resources-${environment}</directory>
                  </resource>
              </resources>
              ...........
      </build>
```
   其中id代表这个环境的唯一标识，下面会用到
   properties下我们我们自己自定义了标签env，内容分别是dev和prd。
   ctiveByDefault=true代表如果不指定某个固定id的profile，那么就使用这个环境
   build 配置输出指定文件目录
   
   
## resource下目录
```java 
java
resources-dev
    saas-jsf-consumer.xml
    log4j.properties
resources-test
    saas-jsf-consumer.xml
    log4j.properties
resources-prod
    saas-jsf-consumer.xml
    log4j.properties
```
   三个resources-xx 目录，分别放置三个环境的配置文件。
   
## package打包

```
   clean -U package -Ptest -DfailfNoTests=false
```
   -P（大写字母） 参数来确定打哪个环境的包。

# 多环境部署支持不同jar包

## 项目pom.xml 配置定义
```java   
      <profiles>
           <profile>
               <id>dev</id>
               <activation>
                   <activeByDefault>true</activeByDefault>
               </activation>
               <properties>
                   <environment>dev</environment>
                   <jedis.jar.scope>provided</jedis.jar.scope>
                   <jimdb.jar.scope>compile</jimdb.jar.scope>
               </properties>
           </profile>
           <profile>
               <id>test</id>
               <properties>
                   <environment>test</environment>
                   <jedis.jar.scope>provided</jedis.jar.scope>
                   <jimdb.jar.scope>compile</jimdb.jar.scope>
               </properties>
           </profile>
           <profile>
               <id>prod</id>
               <properties>
                   <environment>prod</environment>
                   <jedis.jar.scope>compile</jedis.jar.scope>
                   <jimdb.jar.scope>provided</jimdb.jar.scope>
               </properties>
           </profile>
       </profiles>
       
      <dependencys> 
               <dependency>
                   <groupId>com.jd.y</groupId>
                   <artifactId>saas-cache-jimdb</artifactId>
                   <version>1.0-SNAPSHOT</version>
                   <scope>${jimdb.jar.scope}</scope>
               </dependency>
               <dependency>
                   <groupId>com.jd.y</groupId>
                   <artifactId>saas-cache-jedis</artifactId>
                   <version>1.0-SNAPSHOT</version>
                   <scope>${jedis.jar.scope}</scope>
               </dependency>
      </dependencys>
      
      <build>
              <resources>
                  <resource>
                      <directory>${project.basedir}/src/main/resources</directory>
                  </resource>
                  <resource>
                      <directory>${project.basedir}/src/main/resources-${environment}</directory>
                  </resource>
              </resources>
              ...........
      </build>
```
    
### maven scope含义的说明
   
   依赖范围控制哪些依赖在哪些classpath 中可用，哪些依赖包含在一个应用中。让我们详细看一下每一种范围：
    
   - compile （编译范围）
    
   compile是默认的范围；如果没有提供一个范围，那该依赖的范围就是编译范围。编译范围依赖在所有的classpath 中可用，同时它们也会被打包。
    
   - provided （已提供范围）
    
   provided 依赖只有在当JDK 或者一个容器已提供该依赖之后才使用。例如， 如果你开发了一个web 应用，你可能在编译 classpath 中需要可用的Servlet API 来编译一个servlet，但是你不会想要在打包好的WAR 中包含这个Servlet API；这个Servlet API JAR 由你的应用服务器或者servlet 容器提供。已提供范围的依赖在编译classpath （不是运行时）可用。它们不是传递性的，也不会被打包。
    
   - runtime （运行时范围）
    
   runtime 依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC
    驱动实现。
    
   - test （测试范围）
    
   test范围依赖 在一般的编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用。
    
   system （系统范围）
    
   system范围依赖与provided 类似，但是你必须显式的提供一个对于本地系统中JAR 文件的路径。这么做是为了允许基于本地对象编译，而这些对象是系统类库的一部分。这样的构件应该是一直可用的，Maven 也不会在仓库中去寻找它。如果你将一个依赖范围设置成系统范围，你必须同时提供一个 systemPath 元素。注意该范围是不推荐使用的（你应该一直尽量去从公共或定制的 Maven 仓库中引用依赖）。

### pom说明

   通过scode 来控制是否引入jar包
   dev 和 test 打包时 引入 saas-cache-jimdb包 ，而不引入 saas-cache-jedis包
   prod 环境打包时 引入 saas-cache-jedis ，而不引入 saas-cache-jimdb包 

## package打包

```
   clean -U package -Ptest -DfailfNoTests=false
```
   -P（大写字母） 参数来确定打哪个环境的包。
   

# 总结
   
   通过maven 的Profile 来完美实现 同一工程支持多环境部署引用不同的配置文件，同一工程支持多环境部署引用不同的中间件jar包