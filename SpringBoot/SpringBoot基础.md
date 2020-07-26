# SpringBoot简介

## 原有Spring优缺点分析

Spring的优点分析

Spring是Java企业版（Java Enterprise Edition，JEE，也称J2EE）的轻量级代替品。无需开发重量级的Enterprise JavaBean（EJB），Spring为企业级Java开发提供了一种相对简单的方法，通过依赖注入和面向切面编程，用简单的Java对象（Plain Old Java Object，POJO）实现了EJB的功能。

Spring的缺点分析

虽然Spring的组件代码是轻量级的，但它的配置却是重量级的。一开始，Spring用XML配置，而且是很多XML配置。Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。

所有这些配置都代表了开发时的损耗。因为在思考Spring特性配置和解决业务问题之间需要进行思维切换，所以编写配置挤占了编写应用程序逻辑的时间。和所有框架一样，Spring实用，但与此同时它要求的回报也不少。

除此之外，项目的依赖管理也是一件耗时耗力的事情。在环境搭建时，需要分析要导入哪些库的坐标，而且还需要分析导入与之有依赖关系的其他库的坐标，一旦选错了依赖的版本，随之而来的不兼容问题就会严重阻碍项目的开发进度。



## SpringBoot的概述

SpringBoot解决上述Spring的缺点

SpringBoot对上述Spring的缺点进行的改善和优化，基于约定优于配置的思想，可以让开发人员不必在配置与逻辑业务之间进行思维的切换，全身心的投入到逻辑业务的代码编写中，从而大大提高了开发的效率，一定程度上缩短了项目周期。

SpringBoot的特点

- 为基于Spring的开发提供更快的入门体验
- 开箱即用，没有代码生成，也无需XML配置。同时也可以修改默认值来满足特定的需求
- 提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等
- SpringBoot不是对Spring功能上的增强，而是提供了一种快速使用Spring的方式

SpringBoot的核心功能

- 起步依赖

  起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。

  简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能。

- 自动配置

  Spring Boot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定Spring配置应该用哪个，不该用哪个。该过程是Spring自动完成的。



# SpringBoot快速入门

创建Maven工程

使用idea工具创建一个maven工程，该工程为普通的java工程即可

添加SpringBoot的起步依赖

SpringBoot要求，项目要继承SpringBoot的起步依赖spring-boot-starter-parent

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
</parent>
```

SpringBoot要集成SpringMVC进行Controller的开发，所以项目要导入web的启动依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

编写SpringBoot引导类

要通过SpringBoot提供的引导类起步SpringBoot才可以进行访问

```java
package com.itheima;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MySpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(MySpringBootApplication.class);
    }
}
```

编写Controller

在引导类MySpringBootApplication**同级包或者子级包**中创建QuickStartController

```java
package com.itheima.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class QuickStartController {   
    @RequestMapping("/quick")
    @ResponseBody
    public String quick(){
        return "springboot 访问成功!";
    }
    
}
```

执行SpringBoot起步类的主方法，控制台打印日志如下：

```
.   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.1.RELEASE)

2018-05-08 14:29:59.714  INFO 5672 --- [           main] com.itheima.MySpringBootApplication      : Starting MySpringBootApplication on DESKTOP-RRUNFUH with PID 5672 (C:\Users\muzimoo\IdeaProjects\IdeaTest\springboot_quick\target\classes started by muzimoo in C:\Users\muzimoo\IdeaProjects\IdeaTest)
... ... ...
o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-05-08 14:30:03.126  INFO 5672 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2018-05-08 14:30:03.196  INFO 5672 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-05-08 14:30:03.206  INFO 5672 --- [           main] com.itheima.MySpringBootApplication      : Started MySpringBootApplication in 4.252 seconds (JVM running for 5.583)
```

通过日志发现，Tomcat started on port(s): 8080 (http) with context path ''

tomcat已经起步，端口监听8080，web应用的**虚拟工程名称为空**

打开浏览器访问url地址为：http://localhost:8080/quick



# SpringBoot的热部署与单元测试

## SpringBoot简化部署

```xml
<!--这个插件，可以将应用打包成一个可执行的jar包-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```



## SpringBoot工程热部署

我们在开发中反复修改类、页面等资源，每次修改后都是需要重新启动才生效，这样每次启动都很麻烦，浪费了大量的时间，我们可以在修改代码后不重启就能生效，在 pom.xml 中添加如下配置就可以实现这样的功能，我们称之为热部署。

```xml
<!--热部署配置-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

注意：IDEA进行SpringBoot热部署失败原因

出现这种情况，并不是热部署配置问题，其根本原因是因为Intellij IEDA默认情况下不会自动编译，需要对IDEA进行自动编译的设置，如下：

![image-20200601172102376](..\笔记图片\image-20200601172102376.png)

然后 Shift+Ctrl+Alt+/，选择Registry

![image-20200601172120734](..\笔记图片\image-20200601172120734.png)



## SpringBoot自动提示配置

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```



## SpringBoot单元测试

```xml
<!--测试的起步依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

测试代码

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = MySpringBootApplication.class)
public class MapperTest {
    @Autowired
    private UserMapper userMapper;
    //注入Spring容器
    @Autowired
    private WebApplicationContext wac;
    //MockMvc实现了对HTTP请求的模拟
    private MockMvc mvc;
    
    @Before
    public void setupMockMvc(){
        mvc = MockMvcBuilders.webAppContextSetup(wac).build();// 初始化MockMvc对象
    }

    @Test
    public void test() {
        List<User> users = userMapper.queryUserList();
        System.out.println(users);
    }
    
    @Test
    public void qryStudent() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/student/get/1")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .accept(MediaType.APPLICATION_JSON_UTF8)
            )
           .andExpect(MockMvcResultMatchers.status().isOk())
           .andExpect(MockMvcResultMatchers.jsonPath("$.name").value("孙悟空"))
           .andExpect(MockMvcResultMatchers.jsonPath("$.address").value("花果山"))
           .andDo(MockMvcResultHandlers.print());
    }
}
```

其中，SpringRunner继承自SpringJUnit4ClassRunner，使用哪一个Spring提供的测试测试引擎都可以

```java
public final class SpringRunner extends SpringJUnit4ClassRunner 
```

@SpringBootTest的属性指定的是引导类的字节码对象



# SpringBoot原理分析

## 起步依赖原理分析

### 分析spring-boot-starter-parent

按住Ctrl点击pom.xml中的spring-boot-starter-parent，跳转到了spring-boot-starter-parent的pom.xml，xml配置如下（只摘抄了部分重点配置）：

```xml
//版本仲裁中心
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>2.0.1.RELEASE</version>
  <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

点击pom.xml中的spring-boot-starter-dependencies，跳转到了spring-boot-starter-dependencies的pom.xml，xml配置如下（只摘抄了部分重点配置）：

```xml
<properties>
  	<activemq.version>5.15.3</activemq.version>
  	<antlr2.version>2.7.7</antlr2.version>
  	<appengine-sdk.version>1.9.63</appengine-sdk.version>
  	<artemis.version>2.4.0</artemis.version>
  	<aspectj.version>1.8.13</aspectj.version>
  	<assertj.version>3.9.1</assertj.version>
  	<atomikos.version>4.0.6</atomikos.version>
  	<bitronix.version>2.1.4</bitronix.version>
  	<build-helper-maven-plugin.version>3.0.0</build-helper-maven-plugin.version>
  	<byte-buddy.version>1.7.11</byte-buddy.version>
  	... ... ...
</properties>
<dependencyManagement>
  	<dependencies>
      	<dependency>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot</artifactId>
        	<version>2.0.1.RELEASE</version>
      	</dependency>
      	<dependency>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-test</artifactId>
        	<version>2.0.1.RELEASE</version>
      	</dependency>
      	... ... ...
	</dependencies>
</dependencyManagement>
<build>
  	<pluginManagement>
    	<plugins>
      		<plugin>
        		<groupId>org.jetbrains.kotlin</groupId>
        		<artifactId>kotlin-maven-plugin</artifactId>
        		<version>${kotlin.version}</version>
      		</plugin>
      		<plugin>
        		<groupId>org.jooq</groupId>
        		<artifactId>jooq-codegen-maven</artifactId>
        		<version>${jooq.version}</version>
      		</plugin>
      		<plugin>
        		<groupId>org.springframework.boot</groupId>
        		<artifactId>spring-boot-maven-plugin</artifactId>
        		<version>2.0.1.RELEASE</version>
      		</plugin>
          	... ... ...
    	</plugins>
  	</pluginManagement>
</build>
```

从上面的spring-boot-starter-dependencies的pom.xml中我们可以发现，一部分坐标的版本、依赖管理、插件管理已经定义好，所以我们的SpringBoot工程继承spring-boot-starter-parent后已经具备版本锁定等配置了。所以起步依赖的作用就是进行依赖的传递。

### 分析spring-boot-starter-web

点击pom.xml中的spring-boot-starter-web，跳转到了spring-boot-starter-web的pom.xml，xml配置如下（只摘抄了部分重点配置）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  	<modelVersion>4.0.0</modelVersion>
  	<parent>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starters</artifactId>
    	<version>2.0.1.RELEASE</version>
  	</parent>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-web</artifactId>
  	<version>2.0.1.RELEASE</version>
  	<name>Spring Boot Web Starter</name>
  
  	<dependencies>
    	<dependency>
      		<groupId>org.springframework.boot</groupId>
      		<artifactId>spring-boot-starter</artifactId>
      		<version>2.0.1.RELEASE</version>
      		<scope>compile</scope>
    	</dependency>
    	<dependency>
      		<groupId>org.springframework.boot</groupId>
      		<artifactId>spring-boot-starter-json</artifactId>
      		<version>2.0.1.RELEASE</version>
      		<scope>compile</scope>
    	</dependency>
    	<dependency>
      		<groupId>org.springframework.boot</groupId>
      		<artifactId>spring-boot-starter-tomcat</artifactId>
      		<version>2.0.1.RELEASE</version>
      		<scope>compile</scope>
    	</dependency>
    	<dependency>
      		<groupId>org.hibernate.validator</groupId>
      		<artifactId>hibernate-validator</artifactId>
      		<version>6.0.9.Final</version>
      		<scope>compile</scope>
    	</dependency>
    	<dependency>
      		<groupId>org.springframework</groupId>
      		<artifactId>spring-web</artifactId>
      		<version>5.0.5.RELEASE</version>
      		<scope>compile</scope>
    	</dependency>
    	<dependency>
      		<groupId>org.springframework</groupId>
      		<artifactId>spring-webmvc</artifactId>
      		<version>5.0.5.RELEASE</version>
      		<scope>compile</scope>
    	</dependency>
  	</dependencies>
</project>
```

从上面的spring-boot-starter-web的pom.xml中我们可以发现，spring-boot-starter-web就是将web开发要使用的spring-web、spring-webmvc等坐标进行了“打包”，这样我们的工程只要引入spring-boot-starter-web起步依赖的坐标就可以进行web开发了，同样体现了依赖传递的作用。

## 自动配置原理解析

查看启动类MySpringBootApplication上的注解@SpringBootApplication

```java
@SpringBootApplication
public class MySpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(MySpringBootApplication.class);
    }
}
```

注解@SpringBootApplication的源码

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};
	... ... ...
}
```

@SpringBootConfiguration：等同于@Configuration，即标注该类是Spring的一个配置类

@EnableAutoConfiguration：SpringBoot自动配置功能开启

@EnableAutoConfiguration的源码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
	... ... ...
}
```

  @AutoConﬁgurationPackage：自动配置包
  @Import(AutoConﬁgurationPackages.Registrar.class)：
  Spring的底层注解@Import，给容器中导入一个组件；导入的组件由 AutoConﬁgurationPackages.Registrar.class；
  将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器

@Import(AutoConfigurationImportSelector.class) 导入了AutoConfigurationImportSelector类

查看AutoConfigurationImportSelector源码

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
        ... ... ...
        List<String> configurations = getCandidateConfigurations(annotationMetadata,attributes);
        configurations = removeDuplicates(configurations);
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = filter(configurations, autoConfigurationMetadata);
        fireAutoConfigurationImportEvents(configurations, exclusions);
        return StringUtils.toStringArray(configurations);
}
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());		
		return configurations;
}
```

List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);获取候选的配置

SpringFactoriesLoader.loadFactoryNames 方法的作用就是扫描所有jar包类路径下  META‐INF/spring.factories 

把扫描到的这些文件的内容包装成properties对象 

从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中 

spring.factories 文件中有关自动配置的配置信息如下：

```properties
# Auto Configure 
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\ 
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
... ... ...
```

上面配置文件存在大量的以AutoConfiguration为结尾的类名称，这些类就是存有自动配置信息的类，而SpringApplication在获取这些类名后，都加入到容器中

我们以HttpEncodingAutoConﬁguration（Http编码自动配置）为例解释自动配置原理：

```java
@Configuration   //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件 
@EnableConfigurationProperties(HttpEncodingProperties.class)  //启动指定类的 ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把 HttpEncodingProperties加入到ioc容器中   
@ConditionalOnWebApplication //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；    判断当前应用是否是web应用，如果是，当前配置类生效 
@ConditionalOnClass(CharacterEncodingFilter.class)  //判断当前项目有没有这个类 CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing =  true) 
//判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {      
    //他已经和SpringBoot的配置文件映射了   
   private final HttpEncodingProperties properties; 
   //只有一个有参构造器的情况下，参数的值就会从容器中拿  
   public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {   
       this.properties = properties;          
   }            
   @Bean   //给容器中添加一个组件，这个组件的某些值需要从properties中获取 
   @ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？      
   public CharacterEncodingFilter characterEncodingFilter() {    
       CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();  
       filter.setEncoding(this.properties.getCharset().name());      
       filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST)); 
       filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));         
       return filter;          
   }
```

根据当前不同的条件判断，决定这个配置类是否生效？
一旦这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取 的，这些properties类里面的每一个属性又是和配置文件绑定的；

所有在配置文件中能配置的属性都是在xxxxProperties类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类

```java
@ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属 性进行绑定 
public class HttpEncodingProperties {      public static final Charset DEFAULT_CHARSET = Charset.forName("UTF‐8");
}
```

精髓：

 1、SpringBoot启动会加载大量的自动配置类
 2、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；
 3、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）

 4、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；

xxxxAutoConﬁgurartion：自动配置类；
给容器中添加组件

xxxxProperties:封装配置文件中相关属性； 



# SpringBoot的配置文件

## SpringBoot配置文件类型

SpringBoot配置文件类型和作用

SpringBoot是基于约定的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的话，就可以使用application.properties或者application.yml（application.yaml）进行配置。

SpringBoot默认会从Resources目录下加载application.properties或application.yml（application.yaml）文件

其中，application.properties文件是键值对类型的文件，之前一直在使用，所以此处不在对properties文件的格式进行阐述。除了properties文件外，SpringBoot还可以使用yml文件进行配置，下面对yml文件进行讲解。

### application.yml配置文件

yml配置文件简介

YML文件格式是YAML (YAML Aint Markup Language)编写的文件格式，YAML是一种直观的能够被电脑识别的的数据数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，比如： C/C++, Ruby, Python, Java, Perl, C#, PHP等。YML文件是以数据为核心的，比传统的xml方式更加简洁。

YML文件的扩展名可以使用.yml或者.yaml。

 yml配置文件的语法

**配置普通数据**

- 语法： key: value

- 示例代码：

- ```yaml
  name: haohao
  ```

- 注意：value之前有一个空格

**配置对象数据/配置Map数据**

- 语法： 

  ​	key: 

  ​		key1: value1

  ​		key2: value2

  ​	或者：

  ​	key: {key1: value1,key2: value2}

- 示例代码：

- ```yaml
  person:
    name: haohao
    age: 31
    addr: beijing

  #或者
person: {name: haohao,age: 31,addr: beijing}
  ```
  
- 注意：key1前面的空格个数不限定，在yml语法中，相同缩进代表同一个级别

**配置数组（List、Set）数据**

- 语法： 

  ​	key: 

  ​		- value1

  ​		- value2

  或者：

  ​	key: [value1,value2]

- 示例代码：

- ```yaml
  city:
    - beijing
    - tianjin
    - shanghai
    - chongqing
    
  #或者
city: [beijing,tianjin,shanghai,chongqing]
  
#集合中的元素是对象形式
  student:
    - name: zhangsan
      age: 18
      score: 100
    - name: lisi
      age: 28
      score: 88
    - name: wangwu
      age: 38
      score: 90
  ```
  
- 注意：value1与之间的 - 之间存在一个空格

### SpringBoot配置信息的查询

上面提及过，SpringBoot的配置文件，主要的目的就是对配置信息进行修改的，但在配置时的key从哪里去查询呢？我们可以查阅SpringBoot的官方文档

文档URL：https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/htmlsingle/#common-application-properties

常用的配置摘抄如下：

```properties
# QUARTZ SCHEDULER (QuartzProperties)
spring.quartz.jdbc.initialize-schema=embedded # Database schema initialization mode.
spring.quartz.jdbc.schema=classpath:org/quartz/impl/jdbcjobstore/tables_@@platform@@.sql # Path to the SQL file to use to initialize the database schema.
spring.quartz.job-store-type=memory # Quartz job store type.
spring.quartz.properties.*= # Additional Quartz Scheduler properties.

# ----------------------------------------
# WEB PROPERTIES
# ----------------------------------------

# EMBEDDED SERVER CONFIGURATION (ServerProperties)
server.port=8080 # Server HTTP port.
server.servlet.context-path= # Context path of the application.
server.servlet.path=/ # Path of the main dispatcher servlet.

# HTTP encoding (HttpEncodingProperties)
spring.http.encoding.charset=UTF-8 # Charset of HTTP requests and responses. Added to the "Content-Type" header if not set explicitly.

# JACKSON (JacksonProperties)
spring.jackson.date-format= # Date format string or a fully-qualified date format class name. For instance, `yyyy-MM-dd HH:mm:ss`.

# SPRING MVC (WebMvcProperties)
spring.mvc.servlet.load-on-startup=-1 # Load on startup priority of the dispatcher servlet.
spring.mvc.static-path-pattern=/** # Path pattern used for static resources.
spring.mvc.view.prefix= # Spring MVC view prefix.
spring.mvc.view.suffix= # Spring MVC view suffix.

# DATASOURCE (DataSourceAutoConfiguration & DataSourceProperties)
spring.datasource.driver-class-name= # Fully qualified name of the JDBC driver. Auto-detected based on the URL by default.
spring.datasource.password= # Login password of the database.
spring.datasource.url= # JDBC URL of the database.
spring.datasource.username= # Login username of the database.

# JEST (Elasticsearch HTTP client) (JestProperties)
spring.elasticsearch.jest.password= # Login password.
spring.elasticsearch.jest.proxy.host= # Proxy host the HTTP client should use.
spring.elasticsearch.jest.proxy.port= # Proxy port the HTTP client should use.
spring.elasticsearch.jest.read-timeout=3s # Read timeout.
spring.elasticsearch.jest.username= # Login username.

```

我们可以通过配置application.poperties 或者 application.yml 来修改SpringBoot的默认配置

例如：

application.properties文件

```properties
server.port=8888
server.servlet.context-path=demo
```

application.yml文件

```yaml
server:
  port: 8888
  servlet:
    context-path: /demo
```



## 项目属性配置

少量配置信息的情形：**使用注解@Value映射**

我们可以通过@Value注解将配置文件中的值映射到一个Spring管理的Bean的字段上

application.yml配置如下：

```yaml
person:
  name: zhangsan
  age: 18
```

实体Bean代码如下：

```java
@Controller
public class QuickStartController {
    @Value("${person.name}")
    private String name;
    @Value("${person.age}")
    private Integer age;

    @RequestMapping("/quick")
    @ResponseBody
    public String quick(){
        return "springboot 访问成功! name="+name+",age="+age;
    }
}
```



多个配置信息的情形：**使用注解@ConfigurationProperties映射**

通过注解@ConfigurationProperties(prefix="配置文件中的key的前缀")可以将配置文件中的配置自动与实体进行映射

导入依赖

```xml
<dependency>    
    <groupId>org.springframework.boot</groupId>    
    <artifactId>spring-boot-configuration-processor</artifactId>  
    <optional>true</optional> 
</dependency>
```

application.yml配置如下：

```yaml
url:
  orderUrl: http://localhost:8002
  userUrl: 222
  shoppingUrl: 333
  sportsUrl: sports
```

实体Bean代码如下：

**只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能**

```java
@Component
@ConfigurationProperties(prefix = "url")
public class MicroServiceUrl {
    private String orderUrl;  
    private String userUrl;  
    private String shoppingUrl;
    @Value("${url.sportsUrl}")
    private String sportsUrl;

    public void setSportsUrl(String sportsUrl) {
        this.sportsUrl = sportsUrl;
    }

    public void setOrderUrl(String orderUrl) {
        this.orderUrl = orderUrl;
    }

    public void setUserUrl(String userUrl) {
        this.userUrl = userUrl;
    }

    public void setShoppingUrl(String shoppingUrl) {
        this.shoppingUrl = shoppingUrl;
    }
}
```

测试代码

```java
@Resource
private MicroServiceUrl microServiceUrl;//要获取容器中的这个bean

@RequestMapping("url")
public String testConfig() {
    logger.info("=====获取的订单服务地址为：{}", microServiceUrl.getOrderUrl());
    logger.info("=====获取的用户服务地址为：{}", microServiceUrl.getUserUrl());
    logger.info("=====获取的购物车服务地址为：{}", microServiceUrl.getShoppingUrl());
    logger.info("=====获取的体育服务地址为：{}", microServiceUrl.getSportsUrl());
    return "success";
}
```

注意：使用@ConfigurationProperties方式可以进行配置文件与实体字段的自动映射，但需要字段必须提供set方法才可以，而使用@Value注解修饰的字段不需要提供set方法

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value； 如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConﬁgurationProperties；



**@PropertySource&@ImportResource&@Bean**

```
@PropertySource：加载指定的配置文件
@ImportResource：导入Spring的配置文件，让配置文件里面的内容生效； 
Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别； 
想让Spring的配置文件生效，加载进来；@ImportResource标注在一个配置类上
SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式
1、配置类@Conﬁguration------>Spring配置文件 
2、使用@Bean给容器中添加组件
```



**Profile**

多Profile文件

我们在主配置文件编写的时候，文件名可以是 application-{proﬁle}.properties/yml
默认使用application.properties的配置； 

**yml支持多文档块方式**

```yaml
server:   
	port: 8081 
spring:   
	profiles:     
		active: prod   
		
---
server:   
	port: 8083 
spring:   
	profiles: dev   
    
---
server:   
	port: 8084 
spring:   
    profiles: prod  #指定属于哪个环境
```







# Web开发

## SpringBoot对静态资源的映射规则

1）、所有 /webjars/** ，都去 classpath:/META-INF/resources/webjars/ 找资源；
  webjars：以jar包的方式引入静态资源；

 http://www.webjars.org/

2）、"/**" 访问当前项目的任何资源，都去（静态资源的文件夹）找映射(注意访问路径不要加上以下四个静态资源文件夹名，直接写静态资源即可)

```
"classpath:/META‐INF/resources/",  
"classpath:/resources/", 
"classpath:/static/",  
"classpath:/public/"  
"/"：当前项目的根路径
```

3）、欢迎页； 静态资源文件夹下的所有index.html页面；被"/**"映射；

4）、所有的 **/favicon.ico 都是在静态资源文件下找；



## 模板引擎Thymeleaf的使用

```java
@ConfigurationProperties(prefix = "spring.thymeleaf") 
public class ThymeleafProperties {   
    private static final Charset DEFAULT_ENCODING = Charset.forName("UTF‐8"); 
    private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");        
    public static final String DEFAULT_PREFIX = "classpath:/templates/";        
    public static final String DEFAULT_SUFFIX = ".html";         
```

只要我们把HTML页面放在classpath:/templates/，thymeleaf就能自动渲染；

在 html 页面上如果要使用 thymeleaf 模板，需要在页面标签中引入：

```html
<html xmlns:th="http://www.thymeleaf.org">
```

因为 Thymeleaf 中已经有默认的配置了，我们不需要再对其做过多的配置，有一个需要注意一下， Thymeleaf 默认是开启页面缓存的，所以在开发的时候，需要关闭这个页面缓存，配置如下。

```yaml
spring:  
	thymeleaf:   
		cache: false #关闭缓存
```

否则会有缓存，导致页面没法及时看到更新后的效果。 比如你修改了一个文件，已经 update 到 tomcat 了，但刷新页面还是之前的页面，就是因为缓存引起的。



## 对JSP的支持

```xml
<!-- 添加 servlet 依赖. -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>

<!-- 添加 JSTL（JSP Standard Tag Library，JSP标准标签库) -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>

<!--添加 tomcat 的支持.-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>

<!-- Jasper是tomcat中使用的JSP引擎，运用tomcat-embed-jasper可以将项目与tomcat分开 -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
```

```
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```



## SpringMVC配置

#### 1、自动配置

#### 2、扩展SpringMVC

编写一个配置类 (@`Configuration` )  实现`WebMvcConfigurer`  但是必须没有`@EnableWebMvc`

```java
@Configuration
public class MyConfig  implements WebMvcConfigurer {
    //WebMvcConfigurer 用来进行springmvc的扩展功能
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
             //将请求进行转发
         registry.addViewController("/fsl").setViewName("home");
    }
}
```

原理：

 1）、WebMvcAutoConﬁguration是SpringMVC的自动配置类

 2）、在做其他自动配置时会导入 ;@Import(EnableWebMvcConfiguration.class)

WebMvcAutoConfiguration  - >  WebMvcAutoConfigurationAdapter:

```java
@Configuration
@Import({WebMvcAutoConfiguration.EnableWebMvcConfiguration.class})
@EnableConfigurationProperties({WebMvcProperties.class, ResourceProperties.class})
@Order(0)
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer, ResourceLoaderAware {
//查看:EnableWebMvcConfiguration方法
@Configuration
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
```

观察DelegatingWebMvcConfiguration ,发现其中有一个方法是

```java
////从容器中获取所有的WebMvcConfigurer
	@Autowired(required = false)
    public void setConfigurers(List<WebMvcConfigurer> configurers) {
        if (!CollectionUtils.isEmpty(configurers)) {
            this.configurers.addWebMvcConfigurers(configurers);
        }
    }
	public void addWebMvcConfigurers(List<WebMvcConfigurer> configurers) {
        if (!CollectionUtils.isEmpty(configurers)) {
            this.delegates.addAll(configurers);
        }
    }
```

  3）、容器中所有的WebMvcConﬁgurer都会一起起作用；
  4）、我们的配置类也会被调用；

 效果：SpringMVC的自动配置和我们的扩展配置都会起作用； 

#### 3、全面接管SpringMVC

我们需要在上面扩展配置类中添加上@EnableWebMvc注解即可；

原理：
为什么@EnableWebMvc自动配置就失效了；

@EnableWebMvc的核心：、@EnableWebMvc将WebMvcConﬁgurationSupport组件导入进来

```java
@Import({DelegatingWebMvcConfiguration.class})
public @interface EnableWebMvc {
}

@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```

```java
@Configuration @ConditionalOnWebApplication 
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurerAdapter.class })         
//容器中没有这个组件的时候，这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class) @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10) @AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, ValidationAutoConfiguration.class }) 
public class WebMvcAutoConfiguration {
```



## RestfulCRUD

RestfulCRUD：CRUD满足Rest风格

|      | RestfulCRUD      |
| ---- | ---------------- |
| 查询 | emp——GET         |
| 添加 | emp——POST        |
| 更新 | emp/{id}——PUT    |
| 删除 | emp/{id}——DELETE |

```html
<!‐‐需要区分是员工修改还是添加；‐‐> 
<form th:action="@{/emp}" method="post">   
    <!‐‐发送put请求修改员工数据‐‐>    
    <!‐1、SpringMVC中配置HiddenHttpMethodFilter;（SpringBoot自动配置好的）
    2、页面创建一个post表单
    3、创建一个input项，name="_method";值就是我们指定的请求方式
	‐‐>
 <input type="hidden" name="_method" value="put" th:if="${emp!=null}"/> 
```



## 错误处理机制

#### 1、SpringBoot默认的错误处理机制

默认效果：

 1）、浏览器，返回一个默认的错误页面

 2）、如果是其他客户端，默认响应一个json数据

原理：参照ErrorMvcAutoConﬁguration；错误处理的自动配置

给容器中添加以下组件

 DefaultErrorAttributes：

```java
//在页面共享错误信息
errorAttributes.put("timestamp", new Date());
errorAttributes.put("status", status);
errorAttributes.put("error", HttpStatus.valueOf(status).getReasonPhrase());
errorAttributes.put("errors", result.getAllErrors());
errorAttributes.put("exception", error.getClass().getName());
errorAttributes.put("message", error.getMessage());
errorAttributes.put("trace", stackTrace.toString());
errorAttributes.put("path", path);
```

BasicErrorController：处理默认/error请求

```java
	//产生html类型的数据；浏览器发送的请求来到这个方法处理 
	@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections
				.unmodifiableMap(getErrorAttributes(request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        ////去哪个页面作为错误页面；包含页面地址和页面内容 
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

	//产生json数据，其他客户端来到这个方法处理
	@RequestMapping
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		HttpStatus status = getStatus(request);
		if (status == HttpStatus.NO_CONTENT) {
			return new ResponseEntity<Map<String, Object>>(status);
		}
		Map<String, Object> body = getErrorAttributes(request, isIncludeStackTrace(request, MediaType.ALL));
		return new ResponseEntity<>(body, status);
	}
```

ErrorPageCustomizer

```java
// 注册错误页面
// this.dispatcherServletPath.getRelativePath(this.properties.getError().getPath())
public void registerErrorPages(ErrorPageRegistry errorPageRegistry) {
    //getPath()得到如下地址，如果没有自定义error.path属性，则去/error位置
    //@Value("${error.path:/error}")
    //ErrorProperties: private String path = "/error";
    ErrorPage errorPage = new ErrorPage(
        this.dispatcherServletPath.getRelativePath(this.properties.getError().getPath()));
    errorPageRegistry.addErrorPages(errorPage);
}
```

DefaultErrorViewResolver：

```java
static {
    Map<Series, String> views = new EnumMap<>(Series.class);
    views.put(Series.CLIENT_ERROR, "4xx");
    views.put(Series.SERVER_ERROR, "5xx");
    SERIES_VIEWS = Collections.unmodifiableMap(views);
}
public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
    // 使用HTTP完整状态码检查是否有页面可以匹配
    ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);
     //所有的ErrorViewResolver得到ModelAndView 
    if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
        // 使用 HTTP 状态码第一位匹配初始化中的参数创建视图对象
        modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
    }
    return modelAndView;
}
private ModelAndView resolve(String viewName, Map<String, Object> model) {
    // 拼接错误视图路径 /error/[viewname]
    String errorViewName = "error/" + viewName;
    // 使用模版引擎尝试创建视图对象
    TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName,
                                                                                           this.applicationContext);
    if (provider != null) {
        return new ModelAndView(errorViewName, model);
    }
    // 没有模版引擎，使用静态资源文件夹解析视图
    return resolveResource(errorViewName, model);
}
private ModelAndView resolveResource(String viewName, Map<String, Object> model) {
    // 遍历静态资源文件夹,检查是否有存在视图
    for (String location : this.resourceProperties.getStaticLocations()) {
        try {
            Resource resource = this.applicationContext.getResource(location);
            resource = resource.createRelative(viewName + ".html");
            if (resource.exists()) {
                return new ModelAndView(new HtmlResourceView(resource), model);
            }
        }
        catch (Exception ex) {
        }
    }
    return null;
}
```

#### 2、自定义错误响应

2.1 定制错误的页面

1）、有模板引擎的情况下；error/状态码; 【将错误页面命名为 错误状态码.html 放在模板引擎文件夹里面的 error文件夹下】，发生此状态码的错误就会来到 对应的页面；

 我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态 码.html）；
  页面能获取的信息；
  timestamp：时间戳   status：状态码  error：错误提示  exception：异常对象  message：异常消息
  errors：JSR303数据校验的错误都在这里

 2）、没有模板引擎（模板引擎找不到这个错误页面），静态资源文件夹下找；
 3）、以上都没有错误页面，就是默认来到SpringBoot默认的错误提示页面；

2.2 定制错误的json数据

```java
//给容器中加入我们自己定义的ErrorAttributes
@Component 
public class MyErrorAttributes extends DefaultErrorAttributes {       
    @Override     
    public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, 
 boolean includeStackTrace) { 
        Map<String, Object> map = super.getErrorAttributes(requestAttributes,includeStackTrace);         map.put("company","atguigu");      
        return map;   
    } 
}
```



## 返回Json数据及数据封装

@RestController

 @RestController 注解包含了原来的 @Controller 和 @ResponseBody 注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME) 
@Documented 
@Controller
@ResponseBody 
public @interface RestController {    
    String value() default ""; 
}
```



**Spring Boot 默认对Json的处理**

导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
</dependency>
```

默认使用 jackson 框架 

```java
@RequestMapping("user")
public User user(){
    return new User(1l,"bin","123");
}
//结果：{"id":1,"username":"bin","password":"123"}

@RequestMapping("list")
public List<User> list(){
    List<User> list = new LinkedList<>();
    User u1 = new User(1l,"jack","111");
    User u2 = new User(1l,"mike","113");
    User u3 = new User(1l,"amy","114");
    list.add(u1);
    list.add(u2);
    list.add(u3);
    return list;
}
//结果：[{"id":1,"username":"jack","password":"111"},{"id":1,"username":"mike","password":"113"},{"id":1,"username":"amy","password":"114"}]

@RequestMapping("map")
public Map<String,Object> map(){
    Map<String,Object> map = new HashMap<>();
    List<User> list = new LinkedList<>();
    User u1 = new User(1l,"jack","111");
    User u2 = new User(1l,"mike","113");
    list.add(u1);
    list.add(u2);
    map.put("list",list);
    map.put("user",u1);
    map.put("name","jack");
    map.put("age",12);
    map.put("address","深圳");
    return map;
}
//结果：{"address":"深圳","name":"jack","list":[{"id":1,"username":"jack","password":"111"},{"id":1,"username":"mike","password":"113"}],"user":{"id":1,"username":"jack","password":"111"},"age":12}
```



**jackson 中对null的处理**

期望所有的 null 在转 json 时都变成 "" 这种空字符串，需要做一个配置

```java
@Configuration
public class JacksonConfig {
    @Bean
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build(); 
        objectMapper.getSerializerProvider().setNullValueSerializer(new JsonSerializer<Object>() {
            @Override
            public void serialize(Object o, JsonGenerator jsonGenerator, SerializerProvider
                    serializerProvider) throws IOException {
                jsonGenerator.writeString("");
            } 
        }); 
        return objectMapper;
    }
}
```



**使用阿里巴巴的FastJson**

导入依赖

```xml
<dependency>    
    <groupId>com.alibaba</groupId>   
    <artifactId>fastjson</artifactId>  
    <version>1.2.35</version> 
</dependency>
```

FastJson对null的处理

```java
@Configuration
public class FastjsonConfig extends WebMvcConfigurationSupport {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig config = new FastJsonConfig();
        config.setSerializerFeatures(
                // 保留map空的字段              
                SerializerFeature.WriteMapNullValue,
                // 将String类型的null转成""              
                SerializerFeature.WriteNullStringAsEmpty,
                // 将Number类型的null转成0              
                SerializerFeature.WriteNullNumberAsZero,
                // 将List类型的null转成[]                
                SerializerFeature.WriteNullListAsEmpty,
                // 将Boolean类型的null转成false              
                SerializerFeature.WriteNullBooleanAsFalse,
                // 避免循环引用                
                SerializerFeature.DisableCircularReferenceDetect);
        converter.setFastJsonConfig(config);
        converter.setDefaultCharset(Charset.forName("UTF-8"));
        List<MediaType> mediaTypeList = new ArrayList<>();
        // 解决中文乱码问题，相当于在Controller上的@RequestMapping中加了个属性produces = "application/json"  
        mediaTypeList.add(MediaType.APPLICATION_JSON);
        converter.setSupportedMediaTypes(mediaTypeList);
        converters.add(converter);
    }
}
```



**封装统一返回的数据结构**

定义统一的JSON结构

```java
public class JsonResult<T> {
    private T data; 
    private String code;
    private String msg;
  
    public JsonResult() {   
        this.code = "0";   
        this.msg = "操作成功！"; 
    }
  
    public JsonResult(String code, String msg) {  
        this.code = code;  
        this.msg = msg;
    }
  
    public JsonResult(T data) { 
        this.data = data; 
        this.code = "0"; 
        this.msg = "操作成功！"; 
    }

    public JsonResult(T data, String msg) {  
        this.data = data; 
        this.code = "0"; 
        this.msg = msg; 
    }   
    // 省略get和set方法
}
```



## 使用slf4j进行日志记录

**application.yml 文件中对日志的配置：**

```yaml
logging:
  config: src/logback.xml
  level:
    com.hzb.mapper: trace
```

logging.config 是用来指定项目启动的时候，读取哪个配置文件，这里指定的日志配置文件的位置是类路径下的 logback.xml 文件，关于日志的相关配置信息，都放在 logback.xml 文件中了。 logging.level 是用来指定具体的 mapper 中日志的输出级别

常用的日志级别按照从高到低依次为：ERROR、WARN、INFO、DEBUG



 **logback.xml 配置文件解析** 

```xml
<configuration>
    //定义日志输出格式和存储路径
    <property name="LOG_PATTERN" value="%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n" />
    <property name="FILE_PATH" value="D:/logs/demo.%d{yyyy-MMdd}.%i.log" />

    //定义控制台输出
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>          
            <!-- 按照上面配置的LOG_PATTERN来打印日志 -->
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    //定义日志文件的相关参数
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 按照上面配置的FILE_PATH路径来保存日志 -->
            <fileNamePattern>${FILE_PATH}</fileNamePattern>
            <!-- 日志保存15天 -->
            <maxHistory>15</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- 单个日志文件的最大，超过则新建日志文件存储 -->
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <!-- 按照上面配置的LOG_PATTERN来打印日志 -->
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    //定义日志输出级别
    <logger name="com.hzb" level="INFO" />
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```



**使用logger在项目中打印日志**

```java
private final static Logger logger = LoggerFactory.getLogger(TestController.class);

@RequestMapping("log")
public String testLog() {
    logger.debug("=====测试日志debug级别打印====");
    logger.info("======测试日志info级别打印=====");
    logger.error("=====测试日志error级别打印====");
    logger.warn("======测试日志warn级别打印=====");
    // 可以使用占位符打印出一些参数信息
    String str1 = "blog.itcodai.com";
    String str2 = "blog.csdn.net/eson_15";
    logger.info("======个人博客：{}；CSDN博客：{}", str1, str2);
    return "success";
}
```





## 全局异常处理

#### 自定义基础接口

```java
public interface BaseErrorInfoInterface {
    /** 错误码*/
	 String getResultCode();
	
	/** 错误描述*/
	 String getResultMsg();
}
```

#### 自定枚举类实现接口

```java
public enum CommonEnum implements BaseErrorInfoInterface {
	// 数据操作错误定义
	SUCCESS("200", "成功!"), 
	BODY_NOT_MATCH("400","请求的数据格式不符!"),
	SIGNATURE_NOT_MATCH("401","请求的数字签名不匹配!"),
	NOT_FOUND("404", "未找到该资源!"), 
	INTERNAL_SERVER_ERROR("500", "服务器内部错误!"),
	SERVER_BUSY("503","服务器正忙，请稍后再试!");

	/** 错误码 */
	private String resultCode;

	/** 错误描述 */
	private String resultMsg;

	CommonEnum(String resultCode, String resultMsg) {
		this.resultCode = resultCode;
		this.resultMsg = resultMsg;
	}

	@Override
	public String getResultCode() {
		return resultCode;
	}

	@Override
	public String getResultMsg() {
		return resultMsg;
	}
}
```

#### 定义返回的统一JSON结构

```java
public class ResultBody {
	// 响应代码
	private String code;

	//响应消息
	private String message;

	//响应结果
	private Object result;

	public ResultBody() {
	}

	public ResultBody(BaseErrorInfoInterface errorInfo) {
		this.code = errorInfo.getResultCode();
		this.message = errorInfo.getResultMsg();
	}

	public String getCode() {
		return code;
	}

	public void setCode(String code) {
		this.code = code;
	}

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}

	public Object getResult() {
		return result;
	}

	public void setResult(Object result) {
		this.result = result;
	}

	/**
	 * 成功
	 * 
	 * @return
	 */
	public static ResultBody success() {
		return success(null);
	}

	//成功
	public static ResultBody success(Object data) {
		ResultBody rb = new ResultBody();
		rb.setCode(CommonEnum.SUCCESS.getResultCode());
		rb.setMessage(CommonEnum.SUCCESS.getResultMsg());
		rb.setResult(data);
		return rb;
	}

	//失败
	public static ResultBody error(BaseErrorInfoInterface errorInfo) {
		ResultBody rb = new ResultBody();
		rb.setCode(errorInfo.getResultCode());
		rb.setMessage(errorInfo.getResultMsg());
		rb.setResult(null);
		return rb;
	}

	//失败
	public static ResultBody error(String code, String message) {
		ResultBody rb = new ResultBody();
		rb.setCode(code);
		rb.setMessage(message);
		rb.setResult(null);
		return rb;
	}

	//失败
	public static ResultBody error(String message) {
		ResultBody rb = new ResultBody();
		rb.setCode("-1");
		rb.setMessage(message);
		rb.setResult(null);
		return rb;
	}

	@Override
	public String toString() {
		return JSONObject.toJSONString(this);
	}
}
```

#### 自定义异常类

```java
public class BizException extends RuntimeException {
	private static final long serialVersionUID = 1L;
	//错误码
	protected String errorCode;
	//错误信息
	protected String errorMsg;

	public BizException() {
		super();
	}

	public BizException(BaseErrorInfoInterface errorInfoInterface) {
		super(errorInfoInterface.getResultCode());
		this.errorCode = errorInfoInterface.getResultCode();
		this.errorMsg = errorInfoInterface.getResultMsg();
	}
	
	public BizException(BaseErrorInfoInterface errorInfoInterface, Throwable cause) {
		super(errorInfoInterface.getResultCode(), cause);
		this.errorCode = errorInfoInterface.getResultCode();
		this.errorMsg = errorInfoInterface.getResultMsg();
	}
	
	public BizException(String errorMsg) {
		super(errorMsg);
		this.errorMsg = errorMsg;
	}
	
	public BizException(String errorCode, String errorMsg) {
		super(errorCode);
		this.errorCode = errorCode;
		this.errorMsg = errorMsg;
	}

	public BizException(String errorCode, String errorMsg, Throwable cause) {
		super(errorCode, cause);
		this.errorCode = errorCode;
		this.errorMsg = errorMsg;
	}
	
	public String getErrorCode() {
		return errorCode;
	}

	public void setErrorCode(String errorCode) {
		this.errorCode = errorCode;
	}

	public String getErrorMsg() {
		return errorMsg;
	}

	public void setErrorMsg(String errorMsg) {
		this.errorMsg = errorMsg;
	}

	public String getMessage() {
		return errorMsg;
	}

	@Override
	public Throwable fillInStackTrace() {
		return this;
	}
}
```

#### 自定义全局异常处理类

```java
@ControllerAdvice
public class GlobalExceptionHandler {
	private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
	//处理自定义的业务异常
    @ExceptionHandler(value = BizException.class)  
    @ResponseBody  
	public  ResultBody bizExceptionHandler(HttpServletRequest req, BizException e){
    	logger.error("发生业务异常！原因是：{}",e.getErrorMsg());
    	return ResultBody.error(e.getErrorCode(),e.getErrorMsg());
    }
    
    //缺少请求参数异常 
    @ExceptionHandler(MissingServletRequestParameterException.class)
    @ResponseStatus(value = HttpStatus.BAD_REQUEST) 
    public ResultBody handleHttpMessageNotReadableException(   
        MissingServletRequestParameterException ex) {   
        logger.error("缺少请求参数，{}", ex.getMessage());   
        return ResultBody.error("400", "缺少必要的请求参数"); 
    }

	//处理空指针的异常
	@ExceptionHandler(value =NullPointerException.class)
	@ResponseBody
	public ResultBody exceptionHandler(HttpServletRequest req, NullPointerException e){
		logger.error("发生空指针异常！原因是:",e);
		return ResultBody.error(CommonEnum.BODY_NOT_MATCH);
	}

    //处理其他异常
    @ExceptionHandler(value =Exception.class)
	@ResponseBody
	public ResultBody exceptionHandler(HttpServletRequest req, Exception e){
    	logger.error("未知异常！原因是:",e);
       	return ResultBody.error(CommonEnum.INTERNAL_SERVER_ERROR);
    }
}
```



## 拦截器

#### 1、定义拦截器

```java
public class MyInterceptor implements HandlerInterceptor {
    private static final Logger logger = LoggerFactory.getLogger(MyInterceptor.class);
    @Override    
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) 
        throws Exception {
         HandlerMethod handlerMethod = (HandlerMethod) handler;      
        Method method = handlerMethod.getMethod();      
        String methodName = method.getName();       
        logger.info("====拦截到了方法：{}，在该方法执行之前执行====", methodName);      
        // 返回true才会继续执行，返回false则取消当前请求   
        return true;   
    }
    @Override    
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, 
                           ModelAndView modelAndView) throws Exception {      
        logger.info("执行完方法之后进执行(Controller方法调用之后)，但是此时还没进行视图渲 染");  
    }
    @Override   
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, 
                                Exception ex) throws Exception {      
        logger.info("整个请求都处理完咯，DispatcherServlet也渲染了对应的视图咯，此时我可 以做一些清理的工作了");   
    } 
}
```

#### 2、配置拦截器

 Spring Boot 2.0 之前，我们都是直接继承 WebMvcConﬁgurerAdapter 类，然后重写 addInterceptors 方法来实现拦截器的配置。

Spring Boot 2.0 之后，取而代之的是 WebMvcConﬁgurationSupport ，但是这种方式会导致SpringBoot的SpingMVC自动配置失效，并且静态资源会被拦截，解决静态资源被拦截问题的办法如下：

```java
//重写该方法，指定静态资源不被拦截
protected void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/**").addResourceLocations("classpath:/static/");
    super.addResourceHandlers(registry);
}
```

当然，还有更加方便的做法。我们不继承 WebMvcConﬁgurationSupport 类，直接实现 WebMvcConﬁgurer 接口，然后重写 addInterceptors 方法，将自定义的拦截器添加进去即可，这种方式不会拦截 Spring Boot 默认的静态资源，如下：

```java
@Configuration
public class MyInterceptorConfig implements WebMvcConfigurer {
    @Override 
    public void addInterceptors(InterceptorRegistry registry) {  
        // 实现WebMvcConfigurer不会导致静态资源被拦截    
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/**"); 
    }
}
```

#### 3、拦截器使用实例

一般用户登录功能我们可以这么做，要么往 session 中写一个 user，要么针对每个 user 生成一个 token，第二种要更好一点，那么针对第二种方式，如果用户登录成功了，每次请求的时候都会带上该 用户的 token，如果未登录，则没有该 token，服务端可以检测这个 token 参数的有无来判断用户有没 有登录，从而实现拦截功能。

```java
@Override 
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    HandlerMethod handlerMethod = (HandlerMethod) handler;   
    Method method = handlerMethod.getMethod();   
    String methodName = method.getName();   
    logger.info("====拦截到了方法：{}，在该方法执行之前执行====", methodName);
    // 判断用户有没有登陆，一般登陆之后的用户都有一个对应的token   
    String token = request.getParameter("token");   
    if (null == token || "".equals(token)) {       
        logger.info("用户未登录，没有权限执行……请登录");     
        return false;   
    }
    // 返回true才会继续执行，返回false则取消当前请求    
    return true; 
}
```

#### 4、基于注解的拦截器

如果我要拦截所有 /admin 开头的 url 请求的话，需要在拦截器配置中添加这个前缀，但是 在实际项目中，可能会有这种场景出现：某个请求也是 /admin 开头的，但是不能拦截，比如 /admin/login 等等，这样的话又需要去配置。那么，可不可以做成一个类似于开关的东西，哪里不需 要拦截，我就在哪里弄个开关上去，做成这种灵活的可插拔的效果呢？
是可以的，我们可以定义一个注解，该注解专门用来取消拦截操作，如果某个 Controller 中的方法我们 不需要拦截掉，即可在该方法上加上我们自定义的注解即可，下面先定义一个注解：

```java
//该注解用来指定某个方法不用拦截 
@Target(ElementType.METHOD) 
@Retention(RetentionPolicy.RUNTIME) 
public @interface UnInterception { }
```

然后在 Controller 中的某个方法上添加该注解，在拦截器处理方法中添加该注解取消拦截的逻辑，如 下：

```java
@Override 
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    HandlerMethod handlerMethod = (HandlerMethod) handler;   
    Method method = handlerMethod.getMethod();   
    String methodName = method.getName();   
    logger.info("====拦截到了方法：{}，在该方法执行之前执行====", methodName);
    // 通过方法，可以获取该方法上的自定义注解，然后通过注解来判断该方法是否要被拦截   
    // @UnInterception 是我们自定义的注解   
    UnInterception unInterception = method.getAnnotation(UnInterception.class);   
    if (null != unInterception) {       
        return true;    }  
    // 返回true才会继续执行，返回false则取消当前请求   
    return true;
}
```



## 事务配置管理

使用@Transactional注解

常见问题总结：

1、 异常并没有被 ”捕获“ 到

Spring Boot 默认的事务规则是遇到运行异常（RuntimeException）和程序 错误（Error）才会回滚，而抛出 SQLException 就无法回滚了。针对非运行时异常，如果要进行事务回滚的话，可以在 @Transactional 注解中使用 rollbackFor 属性来指定异常，比如 @Transactional(rollbackFor = Exception.class) ，这样就没有问题了，所以在实际项目中，一定要指定异常。 

2、异常被 ”吃“ 掉

因为有try...catch，所以导致异常被 ”吃“ 掉，事务无法回滚。

这种怎么解决呢？直接往上抛，给上一层来处理即可，千万不要在事务中把异常自己 ”吃“ 掉。

3、事务的范围 

```java
@Service
public class UserServiceImpl implements UserService {
    @Resource    
    private UserMapper userMapper;
    @Override   
    @Transactional(rollbackFor = Exception.class)   
    public synchronized void isertUser4(User user) {        
        // 实际中的具体业务……       
        userMapper.insertUser(user);   
    } 
}
```

可以看到，因为要考虑并发问题，我在业务层代码的方法上加了个 synchronized 关键字。我举个实际 的场景，比如一个数据库中，针对某个用户，只有一条记录，下一个插入动作过来，会先判断该数据库 中有没有相同的用户，如果有就不插入，就更新，没有才插入，所以理论上，数据库中永远就一条同一 用户信息，不会出现同一数据库中插入了两条相同用户的信息。  
但是在压测时，就会出现上面的问题，数据库中确实有两条同一用户的信息，分析其原因，在于事务的 范围和锁的范围问题。  
从上面方法中可以看到，方法上是加了事务的，那么也就是说，在执行该方法开始时，事务启动，执行 完了后，事务关闭。但是 synchronized 没有起作用，其实根本原因是因为事务的范围比锁的范围大。 也就是说，在加锁的那部分代码执行完之后，锁释放掉了，但是事务还没结束，此时另一个线程进来 了，事务没结束的话，第二个线程进来时，数据库的状态和第一个线程刚进来是一样的。即由于mysql Innodb引擎的默认隔离级别是可重复读（在同一个事务里，SELECT的结果是事务开始时时间点的状 态），线程二事务开始的时候，线程一还没提交完成，导致读取的数据还没更新。第二个线程也做了插 入动作，导致了脏数据。  
这个问题可以避免，第一，把事务去掉即可（不推荐）；第二，在调用该 service 的地方加锁，保证锁 的范围比事务的范围大即可。 



## 监听器

#### 1、监听器的使用

1.1 监听Servlet上下文对象

针对首页数据，大部分都不常更新的话，我们完全可以把它们缓存起来，每次用户点击的时候，我们都直接从缓存中拿，这样既可以提高首页的访问速度，又可以降低服务器的压力。如果做的更加灵活一点，可以再加个定时器，定期的来更新这个首页缓存。

写一个监听器，实现 ApplicationListener<ContextRefreshedEvent> 接口，重写 onApplicationEvent 方法，将 ContextRefreshedEvent 对象传进去。如果我们想在加载或刷新应用上下文时，也重新刷新下我们预加载的资源，就可以通过监听 ContextRefreshedEvent 来做这样的事情。如下：

```java
@Component
public class MyServletContextListener implements ApplicationListener<ContextRefreshedEvent> {
     @Override   
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {   
        // 先获取到application上下文      
        ApplicationContext applicationContext = contextRefreshedEvent.getApplicationContext();   
        // 获取对应的service        
        UserService userService = applicationContext.getBean(UserService.class);      
        User user = userService.getUser();      
        // 获取application域对象，将查到的信息放到application域中    
        ServletContext application = applicationContext.getBean(ServletContext.class);   
        application.setAttribute("user", user);   
    } 
}
```

1.2  监听HTTP会话 Session对象 

首先该监听器需要实现 HttpSessionListener 接口，然后重写 sessionCreated 和 sessionDestroyed 方法，在 sessionCreated 方法中传递一个 HttpSessionEvent 对象，然后将当前 session 中的用户数量加1， sessionDestroyed 方法刚好相反

```java
@Component 
public class MyHttpSessionListener implements HttpSessionListener {
    private static final Logger logger = LoggerFactory.getLogger(MyHttpSessionListener.class);
    //记录在线的用户数量     
    public Integer count = 0;
    @Override  
    public synchronized void sessionCreated(HttpSessionEvent httpSessionEvent) {   
        logger.info("新用户上线了");   
        count++;      
        httpSessionEvent.getSession().getServletContext().setAttribute("count", count); 
    }
    @Override   
    public synchronized void sessionDestroyed(HttpSessionEvent httpSessionEvent) {      
        logger.info("用户下线了");     
        count--;      
        httpSessionEvent.getSession().getServletContext().setAttribute("count", count);  
    } 
}
```

启动服务器，在浏览器中输入 localhost:8080/listener/total 可以看到返回的结果是1，再打开一个浏览器，请求相同的地址可以看到 count 是 2 ，这没有问题。但是如果关闭一个浏览器再打开，理论上应该还是2，但是实际测试 却是 3。原因是 session 销毁的方法没有执行（可以在后台控制台观察日志打印情况），当重新打开 时，服务器找不到用户原来的 session，于是又重新创建了一个 session，那怎么解决该问题呢？

```java
@GetMapping("/total2")
public String getTotalUser(HttpServletRequest request, HttpServletResponse response) {  
    Cookie cookie;
     try {       
         // 把sessionId记录在浏览器中       
         cookie = new Cookie("JSESSIONID", URLEncoder.encode(request.getSession().getId(), "utf-8"));   
         cookie.setPath("/");      
         //设置cookie有效期为2天，设置长一点      
         cookie.setMaxAge( 48*60 * 60);      
         response.addCookie(cookie);   
     } catch (UnsupportedEncodingException e) {     
         e.printStackTrace();   
     }   
    Integer count = (Integer) request.getSession().getServletContext().getAttribute("count");  
    return "当前在线人数：" + count;
}
```

该处理逻辑是让服务器记得原来那个 session，即把原来的 sessionId 记录在浏览器中，下次再打开时，把这个 sessionId 传过去，这样服务器就不会重新再创建了。重启一下服务器，在浏览器中再次测试一下，即可避免上面的问题。 

1.3  监听客户端请求Servlet Request对象 

```java
@Component
public class MyServletRequestListener implements ServletRequestListener {
    private static final Logger logger = LoggerFactory.getLogger(MyServletRequestListener.class);
    @Override   
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {    
        HttpServletRequest request = (HttpServletRequest) servletRequestEvent.getServletRequest();   
        logger.info("session id为：{}", request.getRequestedSessionId());     
        logger.info("request url为：{}", request.getRequestURL());
        request.setAttribute("name", "倪升武");   
    }
    @Override   
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {
        logger.info("request end");      
        HttpServletRequest request = (HttpServletRequest) servletRequestEvent.getServletRequest(); 
        logger.info("request域中保存的name值为：{}", request.getAttribute("name"));
    }
}
```

#### 2、自定义事件监听

自定义事件需要继承 ApplicationEvent 对象，在事件中定义一个 User 对象来模拟数据，构造方法中将 User 对象传进来初始化。

```java
public class MyEvent extends ApplicationEvent {
    private User user;
    public MyEvent(Object source, User user) {  
        super(source);       
        this.user = user;
    }
    // 省去get、set方法
}
```

自定义监听器需要实现 ApplicationListener 接口

```java
@Component 
public class MyEventListener implements ApplicationListener<MyEvent> {  
    @Override  
    public void onApplicationEvent(MyEvent myEvent) {    
        // 把事件中的信息获取到
        User user = myEvent.getUser();       
        // 处理事件，实际项目中可以通知别的微服务或者处理其他逻辑等等 
        System.out.println("用户名：" + user.getUsername());      
        System.out.println("密码：" + user.getPassword());
    } 
}
```

定义好了事件和监听器之后，需要手动发布事件，这样监听器才能监听到，这需要根据实际业务 场景来触发，针对本文的例子，我写个触发逻辑，如下：

```java
@Service 
public class UserService {
    @Resource   
    private ApplicationContext applicationContext;  
    public User getUser2() {       
        User user = new User(1L, "倪升武", "123456");      
        // 发布事件       
        MyEvent event = new MyEvent(this, user);      
        applicationContext.publishEvent(event);    
        return user;    
    } 
}
```



## 嵌入式Servlet容器

默认使用Tomcat作为嵌入式的Servlet容器

#### 1、定制和修改Servlet容器的相关配置

```properties
server.port=8081 
server.context‐path=/crud  
server.tomcat.uri‐encoding=UTF‐8   
//通用的Servlet容器设置 
server.xxx 
//Tomcat的设置
server.tomcat.xxx
```

**自定义WebServerFactoryCustomizer定制器类**：嵌入式的Servlet容器的定制器；来修改Servlet容器的 配置

```java
@Component
public class WebServerFactoryCustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
        server.setPort(8081);
    }
}
```

#### 2、注册Servlet三大组件Servlet、Filter、Listener

SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文 件。

```java
//注册三大组件 
@Bean
public ServletRegistrationBean myServlet(){ 
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(new  MyServlet(),"/myServlet");     return registrationBean;
}
@Bean 
public FilterRegistrationBean myFilter(){     FilterRegistrationBean registrationBean = new FilterRegistrationBean();     registrationBean.setFilter(new MyFilter());     registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));   
    return registrationBean; 
}
@Bean 
public ServletListenerRegistrationBean myListener(){     ServletListenerRegistrationBean<MyListener> registrationBean = new 
    ServletListenerRegistrationBean<>(new MyListener());    
	return registrationBean; 
}
```

SpringBoot帮我们自动配置SpringMVC的时候，自动的注册SpringMVC的前端控制器；DIspatcherServlet； DispatcherServletAutoConﬁguration

#### 3、替换为其他嵌入式Servlet容器 

```xml
	<!‐‐ 引入web模块 ‐‐> 
    <dependency>   
        <groupId>org.springframework.boot</groupId>   
        <artifactId>spring‐boot‐starter‐web</artifactId>   
    <exclusions>       
        <exclusion>         
            <artifactId>spring‐boot‐starter‐tomcat</artifactId>        
            <groupId>org.springframework.boot</groupId>    
        </exclusion>   
    </exclusions> 
    </dependency> 
    <!‐‐引入其他的Servlet容器‐‐> 
    <dependency>   
        <artifactId>spring‐boot‐starter‐jetty</artifactId>   
        <groupId>org.springframework.boot</groupId>
    </dependency>
```

#### 4、嵌入式Servlet容器自动配置及启动原理

自动配置类

```java
@Configuration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
//导入BeanPostProcessorsRegistrar：Spring注解版；给容器中导入一些组件 
//导入了WebServerFactoryCustomizerBeanPostProcessor： 
//后置处理器：bean初始化前后（创建完对象，还没赋值赋值）执行初始化工作 
public class ServletWebServerFactoryAutoConfiguration {

	@Bean
	public ServletWebServerFactoryCustomizer servletWebServerFactoryCustomizer(ServerProperties serverProperties) {
		return new ServletWebServerFactoryCustomizer(serverProperties);
	}

	@Bean
	@ConditionalOnClass(name = "org.apache.catalina.startup.Tomcat")
	public TomcatServletWebServerFactoryCustomizer tomcatServletWebServerFactoryCustomizer(
			ServerProperties serverProperties) {
		return new TomcatServletWebServerFactoryCustomizer(serverProperties);
	}

	/**
	 * Registers a {@link WebServerFactoryCustomizerBeanPostProcessor}. Registered via
	 * {@link ImportBeanDefinitionRegistrar} for early registration.
	 */
	public static class BeanPostProcessorsRegistrar implements ImportBeanDefinitionRegistrar, BeanFactoryAware {

		private ConfigurableListableBeanFactory beanFactory;

		@Override
		public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
			if (beanFactory instanceof ConfigurableListableBeanFactory) {
				this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
			}
		}

		@Override
		public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
				BeanDefinitionRegistry registry) {
			if (this.beanFactory == null) {
				return;
			}
			registerSyntheticBeanIfMissing(registry, "webServerFactoryCustomizerBeanPostProcessor",
					WebServerFactoryCustomizerBeanPostProcessor.class);
			registerSyntheticBeanIfMissing(registry, "errorPageRegistrarBeanPostProcessor",
					ErrorPageRegistrarBeanPostProcessor.class);
		}

		private void registerSyntheticBeanIfMissing(BeanDefinitionRegistry registry, String name, Class<?> beanClass) {
			if (ObjectUtils.isEmpty(this.beanFactory.getBeanNamesForType(beanClass, true, false))) {
				RootBeanDefinition beanDefinition = new RootBeanDefinition(beanClass);
				beanDefinition.setSynthetic(true);
				registry.registerBeanDefinition(name, beanDefinition);
			}
        }
	}
}
```

创建容器工厂类

```java
@Configuration
class ServletWebServerFactoryConfiguration {
	@Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class, UpgradeProtocol.class })
	@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
	static class EmbeddedTomcat {
		@Bean
		public TomcatServletWebServerFactory tomcatServletWebServerFactory() {
			return new TomcatServletWebServerFactory();
		}
	}

	/**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Server.class, Loader.class, WebAppContext.class })
	@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
	static class EmbeddedJetty {
		@Bean
		public JettyServletWebServerFactory JettyServletWebServerFactory() {
			return new JettyServletWebServerFactory();
		}
	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })
	@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
	static class EmbeddedUndertow {
		@Bean
		public UndertowServletWebServerFactory undertowServletWebServerFactory() {
			return new UndertowServletWebServerFactory();
        }
	}
}
```

扫描ioc容器里是否有对应的ServletWebServerFactory类，TomcatServletWebServerFactory是其中一种，通过调试，是有扫描到的，所以从ioc容器里将这个bean对应的信息封装到ServletWebServerFactory对象

接着是ioc容器创建bean

TomcatServletWebServerFactory工厂类进行Tomcat对象的创建，必要参数的自动配置

```java
public WebServer getWebServer(ServletContextInitializer... initializers) {
        Tomcat tomcat = new Tomcat();
        File baseDir = this.baseDirectory != null ? this.baseDirectory : this.createTempDir("tomcat");
        tomcat.setBaseDir(baseDir.getAbsolutePath());
        Connector connector = new Connector(this.protocol);
        tomcat.getService().addConnector(connector);
        this.customizeConnector(connector);
        tomcat.setConnector(connector);
        tomcat.getHost().setAutoDeploy(false);
        this.configureEngine(tomcat.getEngine());
        Iterator var5 = this.additionalTomcatConnectors.iterator();

        while(var5.hasNext()) {
            Connector additionalConnector = (Connector)var5.next();
            tomcat.getService().addConnector(additionalConnector);
        }

        this.prepareContext(tomcat.getHost(), initializers);
        return this.getTomcatWebServer(tomcat);
    }
```

TomcatWebServer

```java
public TomcatWebServer(Tomcat tomcat, boolean autoStart) {
        this.monitor = new Object();
        this.serviceConnectors = new HashMap();
        Assert.notNull(tomcat, "Tomcat Server must not be null");
        this.tomcat = tomcat;
        this.autoStart = autoStart;
        this.initialize();
    }

private void initialize() throws WebServerException {
    logger.info("Tomcat initialized with port(s): " + this.getPortsDescription(false));
    synchronized(this.monitor) {
        try {
            this.addInstanceIdToEngineName();
            Context context = this.findContext();
            context.addLifecycleListener((event) -> {
                if (context.equals(event.getSource()) && "start".equals(event.getType())) {
                    this.removeServiceConnectors();
                }

            });
            this.tomcat.start();
            this.rethrowDeferredStartupExceptions();

            try {
                ContextBindings.bindClassLoader(context, context.getNamingToken(), this.getClass().getClassLoader());
            } catch (NamingException var5) {
            }
            this.startDaemonAwaitThread();
        } catch (Exception var6) {
            this.stopSilently();
            this.destroySilently();
            throw new WebServerException("Unable to start embedded Tomcat", var6);
        }
    }
}
```

**bean被创建之后，调用了后置处理器**

WebServerFactoryCustomizerBeanPostProcessor

```java
public class WebServerFactoryCustomizerBeanPostProcessor implements BeanPostProcessor, BeanFactoryAware {
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
         //如果当前初始化的是一个WebServerFactory类型的组件 
        if (bean instanceof WebServerFactory) {
            this.postProcessBeforeInitialization((WebServerFactory)bean);
        }

        return bean;
    }
    private void postProcessBeforeInitialization(WebServerFactory webServerFactory) {
        //获取所有的定制器，调用每一个定制器的customize方法来给Servlet容器进行属性赋值； 
        ((Callbacks)LambdaSafe.callbacks(WebServerFactoryCustomizer.class, this.getCustomizers(), webServerFactory, new Object[0]).withLogger(WebServerFactoryCustomizerBeanPostProcessor.class)).invoke((customizer) -> {
            customizer.customize(webServerFactory);
        });
    }
    private Collection<WebServerFactoryCustomizer<?>> getCustomizers() {
        if (this.customizers == null) {
            //从容器中获取所有这个类型的组件：WebServerFactoryCustomizer           
            //定制Servlet容器，给容器中可以添加一个WebServerFactoryCustomizer类型的组件 
            this.customizers = new ArrayList(this.getWebServerFactoryCustomizerBeans());
            this.customizers.sort(AnnotationAwareOrderComparator.INSTANCE);
            this.customizers = Collections.unmodifiableList(this.customizers);
        }
        return this.customizers;
    }
```

具体步骤说明：

1、Springboot的ServletWebServerFactoryAutoConfiguration是嵌入式Servlet容器的自动配置类，这个类的主要作用是创建TomcatServletWebServerFactory工厂类，创建定制器类TomcatServletWebServerFactoryCustomizer，创建FilterRegistrationBean类，同时很关键的一步是注册后置处理器webServerFactoryCustomizerBeanPostProcessor

2、然后Springboot的Application类一启动，就会执行run方法，run经过一系列调用会通过ServletWebServerApplicationContext的onRefresh方法创建ioc容器，然后通过createWebServer方法，createWebServer方法会去ioc容器里扫描是否有对应的ServletWebServerFactory工厂类(TomcatServletWebServerFactory是其中一种)，扫描得到，就会触发webServerFactoryCustomizerBeanPostProcessor后置处理器类，这个处理器类会获取TomcatServletWebServerFactoryCustomizer定制器，并调用customize方法进行定制，这时候工厂类起作用，调用getWebServer方法进行Tomcat属性配置和引擎设置等等，再创建TomcatWebServer启动Tomcat容器



## 使用外置的Servlet容器

**步骤：**

1）、必须创建一个war项目；（利用idea创建好目录结构） 

2）、将嵌入式的Tomcat指定为provided；

```xml
<dependency>    
    <groupId>org.springframework.boot</groupId>    
    <artifactId>spring‐boot‐starter‐tomcat</artifactId>  
    <scope>provided</scope> 
</dependency>
```

3）、必须编写一个SpringBootServletInitializer的子类，并调用conﬁgure方法

```java
public class ServletInitializer extends SpringBootServletInitializer {  
    @Override    
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {      
        //传入SpringBoot应用的主程序    
        return application.sources(SpringBoot04WebJspApplication.class);   
    }   
}
```

4）、启动服务器就可以使用； 

**原理** 
jar包：执行SpringBoot主类的main方法，启动ioc容器，创建嵌入式的Servlet容器；

war包：启动服务器，服务器启动SpringBoot应用【SpringBootServletInitializer】，启动ioc容器；  

servlet3.0（Spring注解版）：
8.2.4 Shared libraries / runtimes pluggability：
**规则：**

 1）、服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面ServletContainerInitializer实例：

 2）、ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，有一个名为 javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitializer的实现类的全类名

 3）、还可以使用@HandlesTypes，在应用启动的时候加载我们感兴趣的类；

**流程：**
1）、启动Tomcat

2）、org\springframework\spring-web\4.3.14.RELEASE\spring-web-4.3.14.RELEASE.jar!\METAINF\services\javax.servlet.ServletContainerInitializer： Spring的web模块里面有这个文件：org.springframework.web.SpringServletContainerInitializer

3）、SpringServletContainerInitializer将@HandlesTypes(WebApplicationInitializer.class)标注的所有这个类型 的类都传入到onStartup方法的Set>；为这些WebApplicationInitializer类型的类创建实例； 

4）、每一个WebApplicationInitializer都调用自己的onStartup；

5）、相当于我们的SpringBootServletInitializer的类会被创建对象，并执行onStartup方法

6）、SpringBootServletInitializer实例执行onStartup的时候会createRootApplicationContext；创建容器

```java
protected WebApplicationContext createRootApplicationContext(     
    ServletContext servletContext) {     
    //1、创建SpringApplicationBuilder   
    SpringApplicationBuilder builder = createSpringApplicationBuilder();
    StandardServletEnvironment environment = new StandardServletEnvironment(); 
    environment.initPropertySources(servletContext, null);  
    builder.environment(environment);  
    builder.main(getClass()); 
    ApplicationContext parent = getExistingRootWebApplicationContext(servletContext); 
    if (parent != null) {      
        this.logger.info("Root context already created (using as parent)."); 
        servletContext.setAttribute( 
            WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null); 
        builder.initializers(new ParentContextApplicationContextInitializer(parent));  
    }   
    builder.initializers(     
        new ServletContextApplicationContextInitializer(servletContext)); 
    builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);        
    //调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来  
    builder = configure(builder);          
    //使用builder创建一个Spring应用    
    SpringApplication application = builder.build(); 
    if (application.getSources().isEmpty() && AnnotationUtils 
        .findAnnotation(getClass(), Configuration.class) != null) { 
        application.getSources().add(getClass());  
    }   
    Assert.state(!application.getSources().isEmpty(), 
                 "No SpringApplication sources have been defined. Either override the " 
                 + "configure method or add an @Configuration annotation"); 
    // Ensure error pages are registered  
    if (this.registerErrorPageFilter) { 
        application.getSources().add(ErrorPageFilterConfiguration.class); 
    }     
    //启动Spring应用  
    return run(application);
}
```

7）、Spring的应用就启动并且创建IOC容器



## 国际化

在Spring程序中，国际化主要是通过`ResourceBundleMessageSource`这个类来实现的

Spring Boot通过`MessageSourceAutoConfiguration`为我们自动配置好了管理国际化资源文件的组件

```java
public class MessageSourceProperties {
	/**
	 * Comma-separated list of basenames (essentially a fully-qualified classpath
	 * location), each following the ResourceBundle convention with relaxed support for
	 * slash based locations. If it doesn't contain a package qualifier (such as
	 * "org.mypackage"), it will be resolved from the classpath root.
	 */
	private String basename = "messages";
```

类中首先声明了一个属性basename,默认值为messages。看其介绍，这是一个以逗号分隔的基本名称列表，如果它不包含包限定符（例如“org.mypackage”），它将从类的根路径解析。它的意思是如果你不在配置文件中指定以逗号分隔开的国际化资源文件名称的话，它默认会去类路径下找messages.properties作为国际化资源文件的基本文件。若是你的国际化资源文件是在类路径某个包(如：i18n)下的话，你就需要在配置文件中指定基本名称了。

```properties
spring.messages.basename=i18n/login/login,i18n/index/index
```

**原理：**

 国际化Locale（区域信息对象）；LocaleResolver（获取区域信息对象）；

在WebMvcAutoConfiguration中找到：

```java
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
public LocaleResolver localeResolver() {
    if (this.mvcProperties.getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
        return new FixedLocaleResolver(this.mvcProperties.getLocale());
    }
    AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
    localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
    return localeResolver;
}

//进入AcceptHeaderLocaleResolver中，找到如下方法
//它是根据HttpServletRequest中的locale属性来判定启用哪个语言文件的
public Locale resolveLocale(HttpServletRequest request) {
    Locale defaultLocale = this.getDefaultLocale();
    if (defaultLocale != null && request.getHeader("Accept-Language") == null) {
        return defaultLocale;
    } else {
        Locale requestLocale = request.getLocale();
        List<Locale> supportedLocales = this.getSupportedLocales();
        if (!supportedLocales.isEmpty() && !supportedLocales.contains(requestLocale)) {
            Locale supportedLocale = this.findSupportedLocale(request, supportedLocales);
            if (supportedLocale != null) {
                return supportedLocale;
            } else {
                return defaultLocale != null ? defaultLocale : requestLocale;
            }
        } else {
            return requestLocale;
        }
    }
}
```

通过点击链接来切换语言，那么我们可以自定义一个区域信息解析器来替代这个默认的解析器

```java
@Bean
public LocaleResolver localeResolver(){
    return new NativeLocaleResolver();
}

protected static class NativeLocaleResolver implements LocaleResolver{
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String language = request.getParameter("language");
        Locale locale = Locale.getDefault();
        if(!StringUtils.isEmpty(language)){
            String[] split = language.split("_");
            locale = new Locale(split[0],split[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
    }
}
```

```html
<a th:href="@{/login.html(language='zh_CN')}">中文</a>
<a th:href="@{/login.html(language='en_US')}">English</a>
```



## 切面AOP处理

```java
//实现AOP切面，@Aspect 注解用来描述一个切面类， @Component 注解让该类交给 Spring 来管理。
@Aspect 
@Component
public class LogAspectHandler {
    //@Pointcut 注解：用来定义一个切面（切入点），定义需要拦截的东西
    /*
        execution() 为表达式主体 
        第一个 * 号的位置：表示返回值类型， * 表示所有类型 
        包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包
        第二个 * 号的位置：表示类名， * 表示所有类 
        *(..) ：这个星号表示方法名， * 表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数
    */
    //annotation() 方式是针对某个注解来定义切面
    //annotation(org.springframework.web.bind.annotation.GetMapping)
     @Pointcut("execution(* com.itcodai.course09.controller..*.*(..))")   
    public void pointCut() {} 
    
    //@Before 注解指定的方法在切面切入目标方法之前执行，可以做一些 log 处理，也可以做一些信息的 统计，比如获取用户的请求 url 以及用户的 ip 地址等等，这个在做个人站点的时候都能用得到，都是常 用的方法。
    //JointPoint 对象很有用，可以用它来获取一个签名，然后利用签名可以获取请求的包名、方法名，包括 参数（通过 joinPoint.getArgs() 获取）等等。 
    @Before("pointCut()")    
    public void doBefore(JoinPoint joinPoint) {
    	logger.info("====doBefore方法进入了====");
        // 获取签名      
        Signature signature = joinPoint.getSignature();    
        // 获取切入的包名       
        String declaringTypeName = signature.getDeclaringTypeName();     
        // 获取即将执行的方法名     
        String funcName = signature.getName();   
        logger.info("即将执行方法为: {}，属于{}包", funcName, declaringTypeName);  
        // 也可以用来记录一些信息，比如获取请求的url和ip    
        ServletRequestAttributes attributes = (ServletRequestAttributes) 
            RequestContextHolder.getRequestAttributes();   
        HttpServletRequest request = attributes.getRequest();      
        // 获取请求url        
        String url = request.getRequestURL().toString();  
        // 获取请求ip       
        String ip = request.getRemoteAddr();       
        logger.info("用户请求的url为：{}，ip地址为：{}", url, ip);   
    } 
    
    @After("pointCut()")   
    public void doAfter(JoinPoint joinPoint) {
        logger.info("====doAfter方法进入了====");    
        Signature signature = joinPoint.getSignature();  
        String method = signature.getName();      
        logger.info("方法{}已经执行完", method);   
    } 
    
    //在 @AfterReturning 注解中，属性 returning 的值必须要和参数保持一致，否则会检测不到。该方法中的第二个入参就是被切方法的返回值，在 doAfterReturning 方法中可以对返回值进行增强，可以根据业务需要做相应的封装。
    @AfterReturning(pointcut = "pointCut()", returning = "result")  
    public void doAfterReturning(JoinPoint joinPoint, Object result) {
        Signature signature = joinPoint.getSignature();       
        String classMethod = signature.getName();       
        logger.info("方法{}执行完毕，返回参数为：{}", classMethod, result);  
        // 实际项目中可以根据业务做具体的返回值增强     
        logger.info("对返回参数进行业务上的增强：{}", result + "增强版");  
    } 
    
    // @AfterThrowing 注解是当被切方法执行时抛出异常时，会进入 @AfterThrowing 注解的 方法中执行，在该方法中可以做一些异常的处理逻辑。要注意的是 throwing 属性的值必须要和参数一致，否则会报错。该方法中的第二个入参即为抛出的异常。
    @AfterThrowing(pointcut = "pointCut()", throwing = "ex")   
    public void afterThrowing(JoinPoint joinPoint, Throwable ex) {  
        Signature signature = joinPoint.getSignature();    
        String method = signature.getName();        
        // 处理异常的逻辑      
        logger.info("执行方法{}出错，异常为：{}", method, ex); 
    } 
}
```



## 文件上传下载

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.2.2</version>
</dependency>

```



```properties
# Single file max size  
spring.http.multipart.maxFileSize=50MB
# All files max size  
spring.http.multipart.maxRequestSize=50MB
```



```java
@Controller
public class FileUploadController {
	
	// 上传文件会自动绑定到MultipartFile中
	 @PostMapping(value="/upload")
	 public String upload(HttpServletRequest request,
			@RequestParam("description") String description,
			@RequestParam("file") MultipartFile file) throws Exception{
		// 接收参数description
	    System.out.println("description = " + description);
	    // 如果文件不为空，写入上传路径
		if(!file.isEmpty()){
			// 上传文件路径
			String path = request.getServletContext().getRealPath(
	                "/upload/");
			System.out.println("path = " + path);
			// 上传文件名
			String filename = file.getOriginalFilename();
		    File filepath = new File(path,filename);
			// 判断路径是否存在，如果不存在就创建一个
	        if (!filepath.getParentFile().exists()) { 
	        	filepath.getParentFile().mkdirs();
	        }
	        // 将上传文件保存到一个目标文件当中
			file.transferTo(new File(path+File.separator+ filename));
			return "success";
		}else{
			return "error";
		}
		 
	 }
	 
    //使用对象方式接受上传文件
	 @RequestMapping(value="/register")
	 public String register(HttpServletRequest request,
			 @ModelAttribute User user,
			 Model model)throws Exception{
		// 接收参数username
		System.out.println("username = " +user.getUsername());
		// 如果文件不为空，写入上传路径
		if(!user.getHeadPortrait().isEmpty()){
			// 上传文件路径
			String path = request.getServletContext().getRealPath(
	                "/upload/");
			System.out.println("path = " + path);
			// 上传文件名
			String filename = user.getHeadPortrait().getOriginalFilename();
		    File filepath = new File(path,filename);
			// 判断路径是否存在，如果不存在就创建一个
	        if (!filepath.getParentFile().exists()) { 
	        	filepath.getParentFile().mkdirs();
	        }
	        // 将上传文件保存到一个目标文件当中
	        user.getHeadPortrait().transferTo(new File(path+File.separator+ filename));
	        // 将用户添加到model
	        model.addAttribute("user", user);
	        return "userInfo";
		}else{
			return "error";
		}
	}
	 
	 @RequestMapping(value="/download")
	 public ResponseEntity<byte[]> download(HttpServletRequest request,
			 @RequestParam("filename") String filename,
			 @RequestHeader("User-Agent") String userAgent,
			 Model model)throws Exception{
		// 下载文件路径
		String path = request.getServletContext().getRealPath(
                "/upload/");
		// 构建File
		File file = new File(path+File.separator+ filename);
		// ok表示Http协议中的状态 200
        BodyBuilder builder = ResponseEntity.ok();
        // 内容长度
        builder.contentLength(file.length());
        // application/octet-stream ： 二进制流数据（最常见的文件下载）。
        builder.contentType(MediaType.APPLICATION_OCTET_STREAM);
        // 使用URLDecoder.decode对文件名进行解码
        filename = URLEncoder.encode(filename, "UTF-8");
        // 设置实际的响应文件名，告诉浏览器文件要用于【下载】、【保存】attachment 以附件形式
        // 不同的浏览器，处理方式不同，要根据浏览器版本进行区别判断
        if (userAgent.indexOf("MSIE") > 0) {
                // 如果是IE，只需要用UTF-8字符集进行URL编码即可
                builder.header("Content-Disposition", "attachment; filename=" + filename);
        } else {
                // 而FireFox、Chrome等浏览器，则需要说明编码的字符集
                // 注意filename后面有个*号，在UTF-8后面有两个单引号！
                builder.header("Content-Disposition", "attachment; filename*=UTF-8''" + filename);
        }
        return builder.body(FileUtils.readFileToByteArray(file));
	 }	
}
```





# SpringBoot与整合其他技术

## SpringBoot整合Mybatis

添加Mybatis的起步依赖

```xml
<!--mybatis起步依赖-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<!-- MySQL连接驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

添加配置信息

```yaml
# 服务端口号
server:
  port: 8080

# 数据库地址
datasource:
  url: localhost:3306/blog_test

# 数据库配置
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://${datasource.url}? useSSL=false&useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&autoReconnect=true&failOverReadOnly=false&maxReconnects =10  
    username: root  
    password: 123456

mybatis:
  # 指定别名设置的包为所有entity
  type-aliases-package: com.hzb.model
  configuration:  
    map-underscore-to-camel-case: true # 驼峰命名规范
  mapper-locations: # mapper映射文件位置  
    - classpath:mapper/*.xml
```

在 Spring Boot 启动类上添加 @MapperScan 注解，来扫描一个包下的所有 mapper



## Spring Boot 集成 Shiro 

1、添加Shiro起步依赖

```xml
<dependency>    
    <groupId>org.apache.shiro</groupId>   
    <artifactId>shiro-spring</artifactId>   
    <version>1.4.0</version> 
</dependency>
```

2、自定义Realm

```java
public class UserRealm extends AuthorizingRealm {
	private UserService userService=new UserServiceImpl();
	private RoleService roleService =new RoleServiceImpl();
	private PermissionService permissionService=new PermissionServiceImpl();
	// 做认证
	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
		String username=token.getPrincipal().toString();
		token.getCredentials();
		System.out.println(username);
		/**
		 * 以前登陆的逻辑是  把用户和密码全部发到数据库去匹配
		 * 在shrio里面是先根据用户名把用户对象查询出来，再来做密码匹配
		 */
		User user=userService.queryUserByUserName(username);
		if(null!=user) {
			List<String> roles=roleService.queryRoleByUserName(user.getUsername());
			List<String> permissions=permissionService.queryPermissionByUserName(user.getUsername());
			ActiverUser activerUser=new ActiverUser(user, roles, permissions);
			/**
			 * 参数说明
			 * 参数1:可以传到任意对象
			 * 参数2:从数据库里面查询出来的密码
			 * 参数3:当前类名
			 */
			SimpleAuthenticationInfo info=new SimpleAuthenticationInfo(activerUser, user.getPwd(), this.getName());
			return info;
		}else {
			//用户不存在  shiro会抛 UnknowAccountException
			return null;
		}
	}

	/**
	 * 作授权
	 * 参数说明
	 */
	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
		ActiverUser activerUser = (ActiverUser) principals.getPrimaryPrincipal();
		SimpleAuthorizationInfo info=new SimpleAuthorizationInfo();
		//添加角色
		Collection<String> roles=activerUser.getRoles();
		if(null!=roles&&roles.size()>0) {
			info.addRoles(roles);
		}
		Collection<String> permissions=activerUser.getPermissions();
		//添加权限
		if(null!=permissions&&permissions.size()>0) {
			info.addStringPermissions(permissions);
		}
//		if(activerUser.getUser().getType()==0) {
//			info.addStringPermission("*:*");
//		}
		return info;
	}
}
```

3、shiro配置

```java
@Configuration 
public class ShiroConfig {
    private static final Logger logger = LoggerFactory.getLogger(ShiroConfig.class);
   
    @Bean   
    public MyRealm myAuthRealm() {     
        MyRealm myRealm = new MyRealm();    
        logger.info("====myRealm注册完成=====");     
        return myRealm;  
    } 
    
     @Bean  
    public SecurityManager securityManager() {  
        // 将自定义realm加进来     
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager(myAuthRealm());     
        logger.info("====securityManager注册完成====");      
        return securityManager;  
    } 
    
     @Bean   
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {        
        // 定义shiroFactoryBean      
        ShiroFilterFactoryBean shiroFilterFactoryBean=new ShiroFilterFactoryBean();
        // 设置自定义的securityManager      
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 设置默认登录的url，身份认证失败会访问该url     
        shiroFilterFactoryBean.setLoginUrl("/login");    
        // 设置成功之后要跳转的链接       
        shiroFilterFactoryBean.setSuccessUrl("/success");  
        // 设置未授权界面，权限认证失败会访问该url    
        shiroFilterFactoryBean.setUnauthorizedUrl("/unauthorized");
        // LinkedHashMap是有序的，进行顺序拦截器配置      
        Map<String,String> filterChainMap = new LinkedHashMap<>();
        // 配置可以匿名访问的地址，可以根据实际情况自己添加，放行一些静态资源等，anon表示放行 
        filterChainMap.put("/css/**", "anon");       
        filterChainMap.put("/imgs/**", "anon");       
        filterChainMap.put("/js/**", "anon");    
        filterChainMap.put("/swagger-*/**", "anon");    
        filterChainMap.put("/swagger-ui.html/**", "anon");  
        // 登录url 放行        
        filterChainMap.put("/login", "anon");
        // “/user/admin” 开头的需要身份认证，authc表示要身份认证   
        filterChainMap.put("/user/admin*", "authc");       
        // “/user/student” 开头的需要角色认证，是“admin”才允许 
        filterChainMap.put("/user/student*/**", "roles[admin]");       
        // “/user/teacher” 开头的需要权限认证，是“user:create”才允许   
        filterChainMap.put("/user/teacher*/**", "perms[\"user:create\"]");
        // 配置logout过滤器       
        filterChainMap.put("/logout", "logout");
        // 设置shiroFilterFactoryBean的FilterChainDefinitionMap 
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainMap);
        logger.info("====shiroFilterFactoryBean注册完成====");       
        return shiroFilterFactoryBean;  
    } 
}
```

4、实现认证和授权

```java
// 1，创建安全管理器的工厂对象 org.apache.shiro.mgt.SecurityManager;  不能使用java.lang.SecurityManager
		Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
		// 2,使用工厂创建安全管理器
		SecurityManager securityManager = factory.getInstance();
		// 3,把当前的安全管理器绑定当到线的线程
		SecurityUtils.setSecurityManager(securityManager);
		// 4,使用SecurityUtils.getSubject得到主体对象
		Subject subject = SecurityUtils.getSubject();
		// 5，封装用户名和密码
		AuthenticationToken token = new UsernamePasswordToken(username, password);
		// 6,得到认证
		try {
			subject.login(token);
			System.out.println("认证通过");
		} catch (AuthenticationException e) {
			System.out.println("用户名或密码不正确");
		} 
		/*} catch (IncorrectCredentialsException e) {
			System.out.println("密码不正确");
		} catch (UnknownAccountException e) {
			System.out.println("用户名不存在");
		}*/

相关方法
subject.hasRole(“”); 判断是否有角色
subject.hashRoles(List);分别判断用户是否具有List中每个内容
subject.hasAllRoles(Collection);返回boolean,要求参数中所有角色用户都需要具有.
subject.isPermitted(“”);判断是否具有权限.
//判断用户是否认证通过
		boolean authenticated = subject.isAuthenticated();
		System.out.println("是否认证通过:"+authenticated);
		//角色判断
		boolean hasRole1 = subject.hasRole("role1");
		System.out.println("是否有role1的角色:"+hasRole1);
		//分别判断集合里面的角色 返回数组
		List<String> roleIdentifiers=Arrays.asList("role1","role2","role3");
		boolean[] hasRoles = subject.hasRoles(roleIdentifiers);
		for (boolean b : hasRoles) {
			System.out.println(b);
		}
		//判断当前用户是否有roleIdentifiers集合里面的所有角色
		boolean hasAllRoles = subject.hasAllRoles(roleIdentifiers);
		System.out.println(hasAllRoles);
		
		//权限判断
		boolean permitted = subject.isPermitted("user:query");
		System.out.println("判断当前用户是否有user:query的权限  "+permitted);
		boolean[] permitted2 = subject.isPermitted("user:query","user:add","user:export");
		for (boolean b : permitted2) {
			System.out.println(b);
		}
		boolean permittedAll = subject.isPermittedAll("user:query","user:add","user:export");
		System.out.println(permittedAll);
```



## Spring Boot集成Lucence 



## SpringBoot整合Spring Data JPA

添加Spring Data JPA的起步依赖

```xml
<!-- springBoot JPA的起步依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!-- MySQL连接驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

在application.properties中配置数据库和jpa的相关属性

```properties
#DB Configuration:
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=root

#JPA Configuration:
spring.jpa.database=MySQL
spring.jpa.show-sql=true
spring.jpa.generate-ddl=true
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming_strategy=org.hibernate.cfg.ImprovedNamingStrategy
```

创建实体配置实体

```java
@Entity
public class User {
    // 主键
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // 用户名
    private String username;
    // 密码
    private String password;
    // 姓名
    private String name;
 
    //此处省略setter和getter方法... ...
}
```

编写UserRepository

```java
public interface UserRepository extends JpaRepository<User,Long>{
    public List<User> findAll();
}
```

编写测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes=MySpringBootApplication.class)
public class JpaTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void test(){
        List<User> users = userRepository.findAll();
        System.out.println(users);
    }

}
```



## SpringBoot整合Redis

添加redis的起步依赖

```xml
<!-- 配置使用redis启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

配置redis的连接信息

```properties
#Redis
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

注入RedisTemplate测试redis操作

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringbootJpaApplication.class)
public class RedisTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Test
    public void test() throws JsonProcessingException {
        //从redis缓存中获得指定的数据
        String userListData = redisTemplate.boundValueOps("user.findAll").get();
        //如果redis中没有数据的话
        if(null==userListData){
            //查询数据库获得数据
            List<User> all = userRepository.findAll();
            //转换成json格式字符串
            ObjectMapper om = new ObjectMapper();
            userListData = om.writeValueAsString(all);
            //将数据存储到redis中，下次在查询直接从redis中获得数据，不用在查询数据库
            redisTemplate.boundValueOps("user.findAll").set(userListData);
            System.out.println("===============从数据库获得数据===============");
        }else{
            System.out.println("===============从redis缓存中获得数据===============");
        }

        System.out.println(userListData);

    }

}
```

