---
layout: post
title: springboot应用war包部署tomcat
tags:
- springboot
- war
- tomcat
---

# 序言
   springboot的应用打包默认是打成jar包，并且如果是web应用的话，默认使用内置的tomcat充当servlet容器，但毕竟内置的tomcat有时候并不满足我们的需求，如有时候我们想集群或者其他一些特性优化配置，因此我们需要把springboot的jar应用打包成war包，并能够在外部tomcat中运行。
   
   很多人会疑问，你直接打成war包并部署到tomcat的webapp下不就行了么?No,springboot的如果在类路径下有tomcat相关类文件，就会以内置tomcat启动的方式，尽管你把war包扔到外置的tomcat的webapp文件下启动springBoot应用也无事于补。
   
# 正文
如何把springboot应用部署到外部tomcat，需要完成如下五个步骤：
##把pom.xml文件中打包结果由jar改成war,如下：
```
    <modelVersion>4.0.0</modelVersion>
    <groupId>spring-boot-panminlan-mybatis-test</groupId>
    <artifactId>mybatis-test</artifactId>
    <packaging>war</packaging>
    <version>0.0.1-SNAPSHOT</version>
```
##添加maven的war打包插件
并且给war包起一个名字，tomcat部署后的访问路径会需要，如：http:localhost:8080/myweb/****
```
 <plugin>     
    <groupId>org.apache.maven.plugins</groupId>     
    <artifactId>maven-war-plugin</artifactId>     
    <configuration>     
     <warSourceExcludes>src/main/resources/**</warSourceExcludes>
     <warName>myweb</warName>     
    </configuration>     
   </plugin>     
```
##排除org.springframework.boot依赖中的tomcat内置容器，这是很重要的一步
```
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
		<exclusions>
			<exclusion>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-tomcat</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
```
##添加对servlet API的依赖
```
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
    </dependency>
```
##继承`SpringBootServletInitializer` ，并覆盖它的 configure 方法
如下图代码，为什么需要提供这样一个SpringBootServletInitializer子类并覆盖它的config方法呢，我们看下该类原代码的注释:
```
/**Note that a WebApplicationInitializer is only needed if you are building a war file and
 * deploying it. If you prefer to run an embedded container then you won't need this at
 * all.
 **/
```
如果我们构建的是wai包并部署到外部tomcat则需要使用它，如果使用内置servlet容器则不需要，外置tomcat环境的配置需要这个类的configure方法来指定初始化资源。

```
@SpringBootApplication
// mapper 接口类扫描包配置
public class Application extends SpringBootServletInitializer{

    public static void main(String[] args) throws IOException {
        // 程序启动入口
        Properties properties = new Properties();
        InputStream in = Application.class.getClassLoader().getResourceAsStream("app.properties");
        properties.load(in);
        SpringApplication app = new SpringApplication(Application.class);
        app.setDefaultProperties(properties);
        app.run(args);
        /*EmbeddedServletContainerAutoConfiguration*/
      
    }
    
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
	    // TODO Auto-generated method stub
	    builder.sources(this.getClass());
	    return super.configure(builder);
    }
```

经过以上配置，我们把构建好的war包拷到tomcat的webapp下，启动tomcat就可以访问啦!
