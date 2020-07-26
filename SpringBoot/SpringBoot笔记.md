## 热部署与单元测试

**简化部署**

内置tomcat，项目可以直接打成jar包

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



**热部署**

```xml
<!--热部署配置-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

在IDE中开启自动编译



**自动提示配置**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```



**单元测试**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

测试代码

@SpringBootTest的属性指定的是引导类的字节码对象

其中，SpringRunner继承自SpringJUnit4ClassRunner，使用哪一个Spring提供的测试测试引擎都可以

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



## 起步依赖原理分析

**分析spring-boot-starter-parent**

```xml
//版本仲裁中心
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>2.0.1.RELEASE</version>
  <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

从spring-boot-starter-dependencies的pom.xml中我们可以发现，一部分坐标的版本、依赖管理、插件管理已经定义好，所以我们的SpringBoot工程继承spring-boot-starter-parent后已经具备版本锁定等配置了。所以起步依赖的作用就是进行依赖的传递。



**以spring-boot-starter-web为例分析starter启动器**

从spring-boot-starter-web的pom.xml中我们可以发现，spring-boot-starter-web就是将web开发要使用的spring-web、spring-webmvc等坐标进行了“打包”，这样我们的工程只要引入spring-boot-starter-web起步依赖的坐标就可以进行web开发了，同样体现了依赖传递的作用。

Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter 相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器



## 自动配置原理解析

查看启动类上的注解@SpringBootApplication

@SpringBootConfiguration：等同于@Configuration，即标注该类是Spring的一个配置类

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
    ...
}
```

@EnableAutoConfiguration：SpringBoot自动配置功能开启

  @AutoConﬁgurationPackage：自动配置包
  @Import(AutoConﬁgurationPackages.Registrar.class)：
  Spring的底层注解@Import，给容器中导入一个组件；这里导入的组件AutoConﬁgurationPackages.Registrar.class
  将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器

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

AutoConfigurationImportSelector

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

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



## 配置文件

SpringBoot配置文件类型和作用

SpringBoot是基于约定的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的话，就可以使用application.properties或者application.yml（application.yaml）进行配置。

SpringBoot默认会从Resources目录下加载application.properties或application.yml（application.yaml）文件



**yml配置文件**

YML文件格式是YAML (YAML Aint Markup Language)编写的文件格式，YAML是一种直观的能够被电脑识别的的数据数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，比如： C/C++, Ruby, Python, Java, Perl, C#, PHP等。YML文件是以数据为核心的，比传统的xml方式更加简洁。

YML文件的扩展名可以使用.yml或者.yaml

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

**定义返回的统一json结构**

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

	//省略get、set方法
}
```



**全局异常处理类**

@ControllerAdvice 注解可拦截项目中抛出的异常，@ControllerAdvice 注解包含了 @Component 注解，说明在 Spring Boot 启动时，也会把该类作为组件交给 Spring 来管理。除此之外，该注解还有basePackages 属性，该属性是用来指定拦截哪个包中的异常信息，一般我们不指定这个属性，我们拦截项目工程中的所有异常。 

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {
    private static final Logger LOGGER = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    //处理系统异常
    @ExceptionHandler(MissingServletRequestParameterException.class)
    public JsonResult<String> handleHttpMessageNoteReadableException(MissingServletRequestParameterException e){
        LOGGER.error("缺少请求参数，{}", e.getMessage());
        return new JsonResult("400", "缺少必要的请求参数");
    }
    @ExceptionHandler(NullPointerException.class)
    public JsonResult<String> handleNullPointerException(NullPointerException e){
        LOGGER.error("空指针异常，{}", e.getMessage());
        return new JsonResult("500", "空指针异常");
    }

    //处理自定义异常
    @ExceptionHandler(BusinessErrorException.class)
    public JsonResult<String> handleBusinessErrorException(BusinessErrorException e){
       String code = e.getErrorCode();
       String msg = e.getErrorMsg();
       return new JsonResult<>(code,msg);
    }

    //把拦截Exception异常写在最下面，如果都没有找到，最后再拦截一下Exception异常，保证输出信息友好。
    @ExceptionHandler(Exception.class)
    public JsonResult<String> handleException(Exception e){
        LOGGER.error("系统异常，{}", e.getMessage());
        return new JsonResult("500", "系统发生异常，请联系管理员");
    }
}
```



## AOP处理

引入依赖

```xml
<dependency>    
    <groupId>org.springframework.boot</groupId>   
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```



**实现AOP切面**

```java
//实现AOP切面，@Aspect 注解用来描述一个切面类， @Component 注解让该类交给 Spring 来管理。
@Aspect 
@Component
public class LogAspectHandler {
    private static final Logger logger = LoggerFactory.getLogger(LogAspectHandler.class);

    /*
    	@Pointcut 注解：用来定义一个切面（切入点），定义需要拦截的东西
        execution() 为表达式主体 
            第一个 * 号的位置：表示返回值类型， * 表示所有类型 
            包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包
            第二个 * 号的位置：表示类名， * 表示所有类 
            *(..) ：这个星号表示方法名， * 表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数
        annotation() 方式是针对某个注解来定义切面
        annotation(org.springframework.web.bind.annotation.GetMapping)
    */
    @Pointcut("execution(* com.hzb.controller..*.*(..))")
    public void pointcut(){}   
    
    /*
    	@Before 注解指定的方法在切面切入目标方法之前执行，可以做一些 log 处理，也可以做一些信息的统计，
    	比如获取用户的请求 url 以及用户的 ip 地址等等，这个在做个人站点的时候都能用得到，都是常用的方法。
    	JointPoint 对象很有用，可以用它来获取一个签名，然后利用签名可以获取请求的包名、方法名，包括参数
    	（通过 joinPoint.getArgs() 获取）等等。 
    */
    @Before("pointcut()")
    public void before(JoinPoint joinPoint){
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
    
     @After("pointcut()")
    public void after(JoinPoint joinPoint){
        logger.info("====doAfter方法进入了====");  
        Signature signature = joinPoint.getSignature(); 
        String method = signature.getName();   
        logger.info("方法{}已经执行完", method); 
    }
    
    /*
    	@AfterReturning，属性 returning 的值必须要和参数保持一致，否则会检测不到。
    	该方法中的第二个入参就是被切方法的返回值，在方法中可以对返回值进行增强，可以根据业务需要做相应的封装。
    */
    @AfterReturning(pointcut = "pointcut()",returning = "result")
    public void afterReturning(JoinPoint joinPoint,Object result){
        Signature signature = joinPoint.getSignature();   
        String classMethod = signature.getName();   
        logger.info("方法{}执行完毕，返回参数为：{}", classMethod, result); 
        // 实际项目中可以根据业务做具体的返回值增强   
        logger.info("对返回参数进行业务上的增强：{}", result + "增强版"); 
    }
    
    /*
    	@AfterThrowing 注解是当被切方法执行时抛出异常时，会进入 @AfterThrowing 注解的方法中执行，
    	在该方法中可以做一些异常的处理逻辑。要注意的是 throwing 属性的值必须要和参数一致，否则会报错。
    	该方法中的第二个入参即为抛出的异常。
    */
    @AfterThrowing(pointcut = "pointcut()", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, Throwable ex) { 
        Signature signature = joinPoint.getSignature();  
        String method = signature.getName();    
        // 处理异常的逻辑   
        logger.info("执行方法{}出错，异常为：{}", method, ex);
    }
}
```



## 事务配置管理

导入了 mysql 依赖后，Spring Boot 会自动注入 DataSourceTransactionManager

我们不需要任何其 他的配置就可以用 @Transactional 注解进行事务的使用



**常见问题**

**异常并没有被 ”捕获“ 到**

Spring Boot 默认的事务规则是遇到运行异常（RuntimeException）和程序错误（Error）才会回滚，而抛出 SQLException 就无法回滚

针对非运行时异常，如果要进行事务回滚的话，可以在 @Transactional 注解中使用 rollbackFor 属性来指定异常

比如 @Transactional(rollbackFor = Exception.class) ，这样就没有问题了，所以在实际项目中，一定要指定异常

**异常被 ”吃“ 掉**

因为有try...catch，所以导致异常被 ”吃“ 掉，事务无法回滚。

这种怎么解决呢？直接往上抛，给上一层来处理即可，千万不要在事务中把异常自己 ”吃“ 掉。

**事务的范围** 

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

可以看到，因为要考虑并发问题，在业务层代码的方法上加了个 synchronized 关键字。

举个实际的场景，比如一个数据库中，针对某个用户，只有一条记录，下一个插入动作过来，会先判断该数据库中有没有相同的用户，如果有就不插入，就更新，没有才插入，所以理论上，数据库中永远就一个用户只有一条信息，不会出现同一数据库中插入了两条相同用户的信息

但是在压测时，就会出现上面的问题，数据库中确实有两条同一用户的信息，分析其原因，在于事务的范围和锁的范围问题

从上面方法中可以看到，方法上是加了事务的，那么也就是说，在执行该方法开始时，事务启动，执行完后，事务关闭。

但是 synchronized 没有起作用，其实根本原因是因为事务的范围比锁的范围大。 

也就是说，在加锁的那部分代码执行完之后，锁释放掉了，但是事务还没结束，此时另一个线程进来 了，事务没结束的话，第二个线程进来时，数据库的状态和第一个线程刚进来是一样的。

即由于mysql Innodb引擎的默认隔离级别是可重复读（在同一个事务里，SELECT的结果是事务开始时时间点的状态），线程二事务开始的时候，线程一还没提交完成，导致读取的数据还没更新。第二个线程也做了插 入动作，导致了脏数据。 

这个问题可以避免，第一，把事务去掉即可（不推荐）；第二，在调用该 service 的地方加锁，保证锁 的范围比事务的范围大即可。



## 拦截器

**自定义拦截器**

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
        logger.info("执行完方法之后进执行(Controller方法调用之后)，但是此时还没进行视图渲染");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
                                Exception ex) throws Exception {   
        logger.info("整个请求都处理完咯，DispatcherServlet也渲染了对应的视图咯，此时我可以做一些清理的工作了");
    }
}
```



**配置拦截器**

方式一

Spring Boot 2.0 之前，我们都是直接继承 WebMvcConﬁgurerAdapter 类，然后重写 addInterceptors 方法来实现拦截器的配置。

Spring Boot 2.0 之后，取而代之的是 WebMvcConﬁgurationSupport ，但是这种方式会导致SpringBoot的SpingMVC自动配置失效，并且静态资源会被拦截

```java
@Configuration
public class InterceptorConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/**");
    }

    //重写该方法，指定静态资源不被拦截
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/static/");
        super.addResourceHandlers(registry);
    }
}
```

方式二

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



**取消拦截操作**

定义一个注解，专门用来取消拦截操作

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UnInterception {
}
```

在不需要拦截的方法上面添加该注解即可

在拦截器中判断方法上是否有该注解，如果有，则不进行拦截

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception {
    HandlerMethod handlerMethod = (HandlerMethod) handler;
    Method method = handlerMethod.getMethod();
    String methodName = method.getName();   
    logger.info("====拦截到了方法：{}，在该方法执行之前执行====", methodName);
    if(method.getAnnotation(UnInterception.class)!=null){
        return true;
    }
    return true;
}
```



# 基础架构搭建

**统一的数据封装**

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

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```



**json的处理**

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



**持久层集成**

在application.yml中进行配置

```yaml
#服务端口号
server:
  port: 8080

#数据库地址
datasource:
  url: localhost:3306/blog_test

spring:
  datasource:  #数据库配置
    driver-class-name: com.mysql.cj.jdbc.Driver
    url:jdbc: mysql://${datasource.url}?useSSL=false&useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&autoReconnect=true&failOverReadOnly=false&maxReconnects=10\u00A0
    username: root
    password: 123456

mybatis:
#指定别名设置的包为所有entity
  type-aliases-package: com.hzb.entity
  configuration:
    map-underscore-to-camel-case: true  #驼峰命名规范
  mapper-locations: #mapper映射文件位置
      -classpath: mapper/*.xml
```



**拦截器**

自定义拦截器

```java
public class MyInterceptor implements HandlerInterceptor {
    private static final Logger logger = LoggerFactory.getLogger(MyInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        logger.info("执行方法之前执行(Controller方法调用之前)");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
                           ModelAndView modelAndView) throws Exception {   
        logger.info("执行完方法之后进执行(Controller方法调用之后)，但是此时还没进行视图渲染");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
                                Exception ex) throws Exception {   
        logger.info("整个请求都处理完咯，DispatcherServlet也渲染了对应的视图咯，此时我可以做一些清理的工作了");
    }
}
```

将自定义拦截器加入都拦截器配置中

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



**全局异常处理**

维护一个异常提示信息枚举类

```java
public enum CommonEnum{
    // 数据操作错误定义
    SUCCESS("200", "成功!"),
    BODY_NOT_MATCH("400","请求的数据格式不符!"),
    SIGNATURE_NOT_MATCH("401","请求的数字签名不匹配!"),
    NOT_FOUND("404", "未找到该资源!"),
    INTERNAL_SERVER_ERROR("500", "服务器内部错误!"),
    SERVER_BUSY("503","服务器正忙，请稍后再试!");

    /** 错误码 */
    private String code;
    /** 错误描述 */
    private String msg;

    private CommonEnum(String code, String msg) {
        this.code = code;
        this.msg = msg;
    }
    public String getCode() {
        return code;
    }
    public String getMsg() {
        return msg;
    }
}
```

全局统一异常处理类，一般对自定义的业务异常最先处理，然后去处理一些常见的系统异常，最后来一个一劳永逸(Exception异常)

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {
    private static final Logger LOGGER = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(MissingServletRequestParameterException.class)
    public JsonResult<String> handleHttpMessageNoteReadableException(MissingServletRequestParameterException e){
        LOGGER.error("缺少请求参数，{}", e.getMessage());
        return new JsonResult("400", "缺少必要的请求参数");
    }

    @ExceptionHandler(NullPointerException.class)
    public JsonResult<String> handleNullPointerException(NullPointerException e){
        LOGGER.error("空指针异常，{}", e.getMessage());
        return new JsonResult("500", "空指针异常");
    }

    @ExceptionHandler(BusinessErrorException.class)
    public JsonResult<String> handleBusinessErrorException(BusinessErrorException e){
       String code = e.getErrorCode();
       String msg = e.getErrorMsg();
       return new JsonResult<>(code,msg);
    }

    @ExceptionHandler(Exception.class)
    public JsonResult<String> handleException(Exception e){
        LOGGER.error("系统异常，{}", e.getMessage());
        return new JsonResult("500", "系统发生异常，请联系管理员");
    }
}
```



# SpringBoot集成其它框架

## 集成Thymeleaf

导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
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



**静态错误页面**

Spring Boot 中会自动识别模板目录（templates/）下的 404.html 和 500.html 文件。我们在 templates/ 目录下新建一个 error 文件夹，专门放置错误的 html 页面，然后分别打印些信息。







## 集成Swagger

导入依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
```



**Swagger2配置**

只要controller中接口的返回值中存在实体类，就会被扫描到Swagger的Model中

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    //配置多个分组，返回多个Docket实例即可
    
    public Docket docket(Environment environment){
        //设置要显示Swagger的环境
        Profiles profiles = Profiles.of("dev","test");
        //通过environment.acceptProfiles判断当前项目所处的环境
        boolean boo = environment.acceptsProfiles(profiles);
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                //enable是否启动swagger
                .enable(boo)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.hzb.controller"))
                //.paths(PathSelectors.ant("/sys/**"))
                .build();
    }

    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("Binboy")
                .version("v1.0")
                .contact("bin,648164883@qq.com")
                .description("first swagger")
                .build();
    }
}
```



**实体类注解**

@ApiModel 注解用于实体类，表示对类进行说明，用于参数用实体类接收。

@ApiModelProperty 注解用于类中属性，表示对 model 属性的说明或者数据操作更改。

```java
@ApiModel(value = "用户实体类")
public class User {
    @ApiModelProperty(value = "用户名")
    private String username;
    @ApiModelProperty(value = "用户密码")
    private String password;
}
```



**Controller相关注解**

@Api 注解用于类上，表示标识这个类是 swagger 的资源。 

@ApiOperation 注解用于方法，表示一个 http 请求的操作。 

@ApiParam 注解用于参数上，用来标明参数信息。

```java
@RestController 
@RequestMapping("/swagger") 
@Api(value = "Swagger2 在线接口文档") 
public class TestController {
    @GetMapping("/get/{id}")    
    @ApiOperation(value = "根据用户唯一标识获取用户信息")    
    public JsonResult<User> getUserInfo(@PathVariable @ApiParam(value = "用户唯一 标识") Long id) {        
        // 模拟数据库中根据id获取User信息        
        User user = new User(id, "倪升武", "123456");        
        return new JsonResult(user);    
    } 
}
```



总结：

可以通过Swagger给一些比较难理解的属性或者接口增加注释信息

接口文档实时更新

可以在线测试

在正式发布时，关闭Swagger



## 集成mybatis

引入依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
```



**配置信息**

**在 Spring Boot 启动类上添加 @MapperScan 注解，来扫描一个包下的所有 mapper**

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



## 集成Redis







## 集成Shiro

导入依赖

```xml
<!--  shiro跟spring的整合依赖  -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.1</version>
</dependency>
<!--  shiro跟thymeleaf的整合依赖  -->
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```



**自定义Realm**

```java
public class UserReaml extends AuthorizingRealm {
    @Resource
    private UserService userService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        //获取用户名
        String username = (String) principals.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        //给用户设置角色
        info.setRoles(userService.getRoles(username));
        //给用户设置权限
        info.setStringPermissions(userService.getPermissions(username));
        return info;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //根据token获取用户名       
        String username = (String) token.getPrincipal();
        //从数据库中查询用户
        User user = userService.getByUsername(username);
        if(null!=user){
            //把当前用户存到session中
            SecurityUtils.getSubject().getSession().setAttribute("user",user);
            //传入用户名和密码进行身份认证，并返回认证信息
            AuthenticationInfo info = new SimpleAuthenticationInfo(user.getUsername(),user.getPassword(),"userRealm");
            return info;
        }
        return null;
    }
}
```



**shiro配置**

在application.yml中配置shiro

```yaml
shiro:
  hash-algorithm-name: md5
  hash-iterations: 2
  anon-urls:
  - /index.html*
  - /sys/toLogin*
  - /login/login*
  - /resources/**
  login-url: /toLogin
  log-out-url: /login/logout*
  authc-ulrs:
  - /**
```

配置类

```java
@Configuration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass(value = { SecurityManager.class })
@ConfigurationProperties(prefix = "shiro")
public class ShiroConfig {
    private static final Logger logger = LoggerFactory.getLogger(ShiroConfig.class);

    private static final String SHIRO_DIALECT = "shiroDialect";
    private static final String SHIRO_FILTER = "shiroFilter";
    private String hashAlgorithmName = "md5";// 加密方式
    private int hashIterations = 2;// 散列次数
    private String loginUrl = "/index.html";// 默认的登陆页面

    private String[] anonUrls;
    private String logOutUrl;
    private String[] authcUlrs;

    //声明凭证匹配器
    @Bean("credentialsMatcher")
    public HashedCredentialsMatcher hashedCredentialsMatcher() {
        HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
        credentialsMatcher.setHashAlgorithmName(hashAlgorithmName);
        credentialsMatcher.setHashIterations(hashIterations);
        return credentialsMatcher;
    }

    //声明userRealm
    @Bean("userRealm")
    public UserRealm userRealm(CredentialsMatcher credentialsMatcher) {
        UserRealm userRealm = new UserRealm();
        // 注入凭证匹配器
        userRealm.setCredentialsMatcher(credentialsMatcher);
        logger.info("userRealm注册完成");
        return userRealm;
    }

    //配置SecurityManager
    @Bean("securityManager")
    public SecurityManager securityManager(UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        // 注入userRealm
        securityManager.setRealm(userRealm);
        logger.info("securityManager注册完成");
        return securityManager;
    }

    //配置shiro的过滤器
    @Bean(SHIRO_FILTER)
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
        // 设置安全管理器
        factoryBean.setSecurityManager(securityManager);
        //设置默认登录的url，身份认证失败会访问该url
        factoryBean.setLoginUrl(loginUrl);
        //设置认证成功之后要跳转的链接
        //factoryBean.setSuccessUrl("/success");
        //设置未授权界面，权限认证失败会访问该url
        factoryBean.setUnauthorizedUrl("/unauthorized");

        //LinkedHashMap是有序的，进行顺序拦截器配置
        Map<String,String> filterChainDefinitionMap = new LinkedHashMap<>();
        // 配置可以匿名访问的地址，可以根据实际情况自己添加，放行一些静态资源等，anon表示放行
        if (anonUrls != null && anonUrls.length > 0) {
            for (String anon : anonUrls) {
                filterChainDefinitionMap.put(anon, "anon");
            }
        }
        // 设置登出的路径
        if (null != logOutUrl) {
            filterChainDefinitionMap.put(logOutUrl, "logout");
        }
        // 设置拦截的路径
        if (authcUlrs != null && authcUlrs.length > 0) {
            for (String authc : authcUlrs) {
                filterChainDefinitionMap.put(authc, "authc");
            }
        }
        // “/user/student” 开头的需要角色认证，是“admin”才允许
        filterChainMap.put("/user/student*/**","roles[admin]");
        // “/user/teacher” 开头的需要权限认证，是“user:create”才允许
        filterChainMap.put("/user/teacher*/**","perms[\'user:create\']");

        Map<String, Filter> filters=new HashMap<>();
        //		filters.put("authc", new ShiroLoginFilter());
        //配置过滤器
        factoryBean.setFilters(filters);
        factoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        logger.info("shiroFilterFactoryBean注册完成");
        return factoryBean;
    }
    
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
    
    //注册shiro的委托过滤器，相当于之前在web.xml里面配置的
	@Bean
	public FilterRegistrationBean<DelegatingFilterProxy> delegatingFilterProxy() {
		FilterRegistrationBean<DelegatingFilterProxy> filterRegistrationBean = new FilterRegistrationBean<DelegatingFilterProxy>();
		DelegatingFilterProxy proxy = new DelegatingFilterProxy();
		proxy.setTargetFilterLifecycle(true);
		proxy.setTargetBeanName(SHIRO_FILTER);
		filterRegistrationBean.setFilter(proxy);
		return filterRegistrationBean;
	}

    /*
    	开启Shiro的注解(如@RequiresRoles,@RequiresPermissions)
    	需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
    	配置以下两个bean(DefaultAdvisorAutoProxyCreator(可选)和AuthorizationAttributeSourceAdvisor)即可实现此功能
    */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }
    @Bean
    @DependsOn({"lifecycleBeanPostProcessor"})
    public DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        advisorAutoProxyCreator.setProxyTargetClass(true);
        return advisorAutoProxyCreator;
    }

    //创建shiro跟themeleaf交互的ShiroDialect
    @Bean(name = SHIRO_DIALECT)
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }
}
```



**用户登录接口**

```java
@RequestMapping(value = "/tologin",method = RequestMethod.GET)
public String tologin()
{
    return "login";
}

@RequestMapping("login")
public String login(User user, HttpServletRequest request){
    //根据用户名和密码创建token
    UsernamePasswordToken token = new UsernamePasswordToken(user.getUsername(),user.getPassword());
    //获取subject认证主体
    Subject subject = SecurityUtils.getSubject();
    try {
        //开始认证，这一步会跳到自定义的Realm中
        subject.login(token);
        request.getSession().setAttribute("user",user);
        return "success";
    }catch (Exception ex){
        request.setAttribute("error",ex.getMessage());
        return "login";
    }
}
```



**需要做验证的接口**

```java
@RequiresPermissions("user:all")
@RequestMapping("/reportcheck/all/{status}")
public String reportcheckall(Model model, @PathVariable("status") String status)
{
    ...
}

//value中的值必须写成下面列表的形式,如果写错了权限无法赋予
//logical = Logical.OR 只要有其中一个权限就可以访问,and是两个需要同时具备
@RequiresPermissions(value = {"user:tstable","user:all"},logical = Logical.OR)
@RequestMapping("/reportcheck/ts/{status}")
public String reportcheckts(Model model, @PathVariable("status") String status)
{
    ...
}
```



**处理没有权限的异常**

```java
@ControllerAdvice
public class NoPermissionException {
    @ResponseBody
    @ExceptionHandler(UnauthorizedException.class)
    public String handleShiroException(Exception ex) {
        return "<h1>亲,你没有权限噢!</h1>";
    }
}
```











