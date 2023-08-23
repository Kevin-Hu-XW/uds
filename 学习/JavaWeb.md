# JavaWeb
## springboot
##### 介绍
SMM框架 Spring SpringMVC Mybatis xml方式配置
SpringBoot快速开发框架，帮助我们快速的整合第三方框架特性：
1、去除xml配置
2、全部采用注解的方式配置
3、内嵌入Tomcat
SpringBoot框架默认情况下spring-boot-starter-web已经帮我们整合号springMvc框架
**SpringBoot和SpringCloud区别：**

- SpringBoot 是一个快速开发的框架,能够快速的整合第三方框架，简化XML配置，全部采用注解形式，内置Tomcat容器,帮助开发者能够实现快速开发，SpringBoot的Web组件 默认集成的是SpringMVC框架。

- SpringMVC是控制层。

- SpringCloud依赖与SpringBoot组件，使用SpringMVC编写Http协议接口，同时SpringCloud是一套完整的微服务解决框架。

**spring-boot-starter-parent作用：**
在pom.xml中引入spring-boot-start-parent,spring官方的解释叫什么stater poms,它可以提供dependency management,也就是说依赖管理，引入以后在申明其它dependency的时候就不需要version了，后面可以看到。
**spring-boot-starter-web作用:**
帮助整合spring、springmvc，帮助我们快速整合第三方框架的原理封装Maven依赖的方式实现
**@RestController**
在加上@RestController表示修饰该COntroller所有的方法返回JSON格式，直接可以编写
由springMVC提供，而不是springboot提供；相当于在每个方法上加上@ResponseBody注解，返回JSON格式，而不是每个方法上都加上@ResponseBody注解；而@Controller控制层注解spirngmvc接口映射，默认情况下返回页面跳转，如果需要返回JSON格式下需要@ResponseBody注解

```java
@SpringBootApplication
由@Configuration、@EnableAutoConfiguration、@ComponentScan组合而来，
@ComponentScan扫描包范围：当前启动类同级包或者子包下面
```

##### 静态资源访问
Spring Boot默认提供静态资源目录位置需置于classpath下，目录名需符合如下规则：
- /static
- /public
- /resources	
- /META-INF/resources

举例：我们可以在src/main/resources/目录下创建static，在该位置放置一个图片文件。启动程序后，尝试访问http://localhost:8080/D.jpg。如能显示图片，配置成功。
##### SpringBoot读取Resource下文件

```java
File file = ResourceUtils.getFile("classpath:token/token.txt");
InputStream inputStream = new FileInputStream(file);
BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
StringBuilder stringBuilder = new StringBuilder();
String line = null;
while ((line = reader.readLine()) != null) {
    log.info("###########token:"+line);
    stringBuilder.append(line);
}
```

##### SpringBoot实现访问本地磁盘电脑资源

```yml
#对外暴露的静态资源路径
file. upload.static-access-path= /uploads/**
#本地磁盘路径
file. upload.folder= E:/uploads/
```

这样前端可以通过"http://localhost:8080/uploads/"前缀进行访问

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
@Configuration
public class StaticResourcesConfig implements WebMvcConfigurer {
    /*
        本地磁盘路径
     */
    @Value("${filepath.downLoadPath}")
    private String downLoadPath;
    /*
        静态资源对外暴露的访问路径
     */
    @Value("${filepath.staticAccessPath}")
    private String staticAccessPath;
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //将本地磁盘的指定路径映射成Spring Boot的静态资源路径
        registry.addResourceHandler(staticAccessPath)
                .addResourceLocations("file:"+downLoadPath);
    }
}
```



##### 使用Freemarker模板引擎渲染web视图
```java
<!-- 引入freeMarker的依赖包. -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
    </dependency>
```
在src/main/resources/创建一个templates文件夹,后缀为*.ftlh
新建application.yml文件
```java
spring:
  freemarker:
        allow-request-override: false
        cache: false
        check-template-location: true
        content-type: text/html; charset=utf-8
        expose-request-attributes: false
        expose-session-attributes: false
        expose-spring-macro-helpers: false
        suffix: .ftlh

        template-loader-path: classpath:/templates
server:
  servlet:
    encoding:
      force: true
      charset: UTF-8
```
#####  整合mybatis
```java
<!--添加相关依赖-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>${mybatis.version}</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.25</version>
</dependency>
    
<!--application.yml配置-->
spring:
  datasource:
    url: jdbc:mysql://101.35.56.116:3306/saas?useSSL=true
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis:
  mapperLocations: classpath:mapping/*.xml
```
**编写mapper**

```java
@Mapper
public interface UserMapper {
    User queryUserById(Long id);
}
```

**编写xml**
```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace：必须与对应的接口全类名一致 id:必须与对应接口的某个对应的方法名一致 -->

<mapper namespace="com.java.web.mybatis.mapper.UserMapper">

    <select id="queryUserById" resultType="com.java.web.base.pojo.User">
        select * from user where id = #{id}
    </select>
</mapper>
```
**Service层和Controller层**
```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    public User queryUserById(Long id){
        return this.userMapper.queryUserById(id);
    }
}
@RestController
public class UserController {
    @Autowired
    private UserService userService;

    @RequestMapping("/test/{id}")
    public String queryUser(@PathVariable("id") Long id){
        return this.userService.queryUserById(id).getUserName();
    }
}
```
##### 整合配置文件
**在springboot整合配置文件，分成两大类：**
application.properties、application.yml或者是Bootstrap.properties、Bootstrap.yml
相对于来说yml文件格式写法更加精简，减少配置文件的冗余性。
**加载顺序：**
bootstrap.yml 先加载 application.yml后加载
bootstrap.yml 用于应用程序上下文的引导阶段。
bootstrap.yml 由父Spring ApplicationContext加载。
**区别：**
bootstrap.yml 和 application.yml 都可以用来配置参数。
bootstrap.yml 用来程序引导时执行，应用于更加早期配置信息读取。可以理解成系统级别的一些参数配置，这些参数一般是不会变动的。一旦bootStrap.yml 被加载，则内容不会被覆盖。
application.yml 可以用来定义应用级别的，应用程序特有配置信息，可以用来配置后续各个模块中需使用的公共参数等。

**Properties在线转换yml格式网址：https://www.toyaml.com/index.html**

##### 多环境配置
```json
spring:
  profiles:
    active: dev
```

##### 核心配置
```json 
server:
  port: 8081
  servlet:
  #配置上下文路径
    context-path: /kevin
```

##### 日志管理-Log4j

Springboot 已经默认帮你整合好了logback，日志输出文件在当前项目路径log文件夹下

可以通过添加lombok依赖，通过注解的方式进行日志输出

==注：如果log无法点出info方法时，需要在idea中安装lombok插件==

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```
创建logback.xml配置文件

```xml
<configuration>
    <!--本文主要输出日志为控制台日志，系统日志，sql日志，异常日志-->
    <!-- %m输出的信息,%p日志级别,%t线程名,%d日期,%c类的全名,,,, -->
    <!--控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d %p (%file:%line\)- %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--系统info级别日志-->
    <!--<File> 日志目录，没有会自动创建-->
    <!--<rollingPolicy>日志策略，每天简历一个日志文件，或者当天日志文件超过64MB时-->
    <!--encoder 日志编码及输出格式-->
    <appender name="fileLog"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>log/file/fileLog.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>log/file/fileLog.log.%d.%i</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- or whenever the file size reaches 64 MB -->
                <maxFileSize>64 MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>
                %d %p (%file:%line\)- %m%n
            </pattern>
            <charset>UTF-8</charset>
            <!-- 此处设置字符集 -->
        </encoder>
    </appender>

    <!--sql日志-->
    <appender name="sqlFile"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>log/sql/sqlFile.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>log/sql/sqlFile.log.%d.%i</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- or whenever the file size reaches 64 MB -->
                <maxFileSize>64 MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!--对记录事件进行格式化。负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。-->
        <encoder>
            <!--用来设置日志的输入格式-->
            <pattern>
                %d %p (%file:%line\)- %m%n
            </pattern>
            <charset>UTF-8</charset>
            <!-- 此处设置字符集 -->
        </encoder>
    </appender>


    <!--异常日志-->
    <appender name="errorFile"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>log/error/errorFile.log</File>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>log/error/errorFile.%d.log.%i</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- or whenever the file size reaches 64 MB -->
                <maxFileSize>64 MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!--对记录事件进行格式化。负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。-->
        <encoder>
            <!--用来设置日志的输入格式-->
            <pattern>
                %d %p (%file:%line\)- %m%n
            </pattern>
            <charset>UTF-8</charset>
            <!-- 此处设置字符集 -->
        </encoder>
        <!--
            日志都在这里 过滤出 error
            使用 try {}catch (Exception e){} 的话异常无法写入日志，可以在catch里用logger.error()方法手动写入日志
            -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!--  日志输出级别 -->
    <!--All\DEBUG\INFO\WARN\ERROR\FATAL\OFF-->
    <!--打印info级别日志，分别在控制台，fileLog，errorFile输出
        异常日志在上面由过滤器过滤出ERROR日志打印
    -->
    <root level="INFO">
        <appender-ref ref="fileLog" />
        <appender-ref ref="console" />
        <appender-ref ref="errorFile" />
    </root>

    <!--打印sql至sqlFile文件日志-->
    <logger name="com.dolphin.mapper" level="DEBUG" additivity="false">
        <appender-ref ref="console" />
        <appender-ref ref="sqlFile" />
    </logger>
</configuration>
```

==并在yml中配置logback.xml路径,指定读取logback配置文件==

```json
logging:
  #config: log/logback.xml 会直接从项目目录下找log目录读取logback.xml文件
  #config: classpath:log/logback.xml 会从resource下找log目录读取logback.xml文件
  config: classpath:log/logback.xml  
```
**日志级别**
| 级别  |                           含义                           |
| :---: | :------------------------------------------------------: |
| fatal |       指出每个严重的错误事件将会导致应用程序的退出       |
| error |       虽然发生错误事件，但仍然不影响系统的继续运行       |
| warn  |                 表明会出现潜在的错误情形                 |
| info  |       一般和在粗粒度级别上，强调应用程序的运行全程       |
| debug | 一般用于细粒度级别上，对调试应用程序非常有帮助。默认级别 |
| trace | 是程序追踪，可以用于输出程序运行中的变量，显示执行的流程 |

**Log4j 组件**
==Log4J 主要由Loggers (日志记录器)、Appenders输出端）和 Layout（日志格式化器）组成==。其中 Loggers 控制日志的输出级别与日志是否输出；Appenders 指定日志的输出方式（输出到控制台、文件等;Layout 控制日志信息的输出格式。

**Loggers**
==日志记录器，负责收集处理日志记录==

**Appenders**
==Appender 用来指定日志输出到哪个地方，可以同时指定日志的输出目的地。==Log4j 常用的输出目的地 有以下几种：

|      **输出端类型**      |                           **作用**                           |
| :----------------------: | :----------------------------------------------------------: |
|     ConsoleAppender      |                      将日志输出到控制台                      |
|       FileAppender       |                      将日志输出到文件中                      |
| DailyRollingFileAppender |     将日志输出到一个日志文件，并且每天输出到一个新的文件     |
|   RollingFileAppender    | 将日志信息输出到一个日志文件，并且指定文件的尺寸，当文件大 小达到指定尺寸时，会自动把文件改名，同时产生一个新的文件 |
|       JDBCAppender       |                   把日志信息保存到数据库中                   |

**Layouts**
布局器 Layouts用于控制日志输出内容的格式，让我们可以使用各种需要的格式输出日志。Log4j常用 的Layouts:

| **格式化器类型** |                           **作用**                           |
| :--------------: | :----------------------------------------------------------: |
|    HTMLLayout    |                 格式化日志输出为HTML表格形式                 |
|   SimpleLayout   |   简单的日志输出格式化，打印的日志格式为（info - message）   |
|  PatternLayout   | 最强大的格式化期，可以根据自定义格式输出日志，如果没有指定转换格式， 就是用默认的转换格式 |

==日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL==

##### springboot整合log4j
由于springboot默认整合了logback，所以在整合log4j前需要先排除logback

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
<!-- spring boot start -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <!-- 排除自带的logback依赖 -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- springboot-log4j -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j</artifactId>
    <version>1.3.8.RELEASE</version>
</dependency>
```

##### springboot 整合websocket



##### 使用aop统一处理web日志（或elk）

作用：在我们的方法的前后实现拦截 减少打印日志代码的冗余性的问题

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```java
@Aspect
@Component
@Slf4j
public class WebLogAspect {

	/**
	 * 第一个点* 表示 com.java.web.hello下的所有类
	 * 第二个点* 表示 com.java.web.hello下的所有方法
	 * (..)   表示 com.java.web.hello下的所有方法下的参数
	 */
	@Pointcut("execution(public * com.java.web.hello.*.*(..))")
	public void webLog() {
	}

	/**
	 * 前置通知，请求方法之前做拦截
	 * @param joinPoint
	 * @throws Throwable
	 */
	@Before("webLog()")
	public void doBefore(JoinPoint joinPoint) throws Throwable {
		// 接收到请求，记录请求内容
		ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
		HttpServletRequest request = attributes.getRequest();
		// 记录下请求内容
		log.info("URL : " + request.getRequestURL().toString());
		log.info("HTTP_METHOD : " + request.getMethod());
		log.info("IP : " + request.getRemoteAddr());
		Enumeration<String> enu = request.getParameterNames();
		while (enu.hasMoreElements()) {
			String name = (String) enu.nextElement();
			log.info("name:{},value:{}", name, request.getParameter(name));
		}
	}

	/**
	 * 目标方法请求之后，响应的信息
	 * @param ret
	 * @throws Throwable
	 */
	@AfterReturning(returning = "ret", pointcut = "webLog()")
	public void doAfterReturning(Object ret) throws Throwable {
		// 处理完请求，返回内容
		log.info("RESPONSE : " + ret);
	}
}

```

##### 打包运行
mvn clean package  打包
java -jar 包名 

##### springboot通过外部的方式加载yml配置文件
java -jar uds-saas-lan-1.0.0-SNAPSHOT.jar --spring.config.location=F:/bootstrap.yml

## springcloud
### 微服务架构的演变
==传统单体架构->分布式架构->SOA面向服务架构->微服务架构模式==

- ==传统架构模式==

  传统的架构，也就是为单点应用，也就是大家在早期所学习的JavaEE知识SSH或者SSM架构模式，会采用分层架构模式：数据库访问层、业务逻辑层、控制层，从前端到后台所有的代码都是一个开发者去完成。

  controller-视图层

  service-业务逻辑层

  dao-数据库访问层

  将项目代码都放到同一个项目中，部署到同一个tomcat中

  该模式的优缺点：

  ​		优点：开发简单，运维简单

  ​		缺点：该架构模式没有对业务逻辑进行拆分，所有代码都写都同一个项目中，适合于小团队或者个人形式的开   					发，不适合团队协同模式开发工作。如果相同某个模块出现了不可用的情况，会导致整个系统无法使用。

  应用场景：政府项目、管理系统、OA适合于小团队开发

- ==分布式架构模式==

  分布式架构模式是基于传统的架构模式演变过来，将传统的单点项目根据业务模块实现拆分、会拆分为会员系统、订单系统、支付系统、秒杀系统等。 从而降低我们项目的耦合度，这种架构模式开始慢慢的适合于互联网公司开发团队。 

- ==SOA面向服务架构==

  SOA架构模式也称作为：面向服务架构模式、俗称面向与接口开发，将共同存在的业务逻辑抽取成一个共同的服务，提供给其他的服务接口实现调用、服务与服务之间通讯采用rpc远程调用技术。

  服务只有接口，没有视图层

  能够解决什么问题：代码冗余性，把共同的代码抽取成一个服务被其它的服务就行调用

  特点：

  1、SOA架构模式采用soap协议（http/https+xml）实现传输，在高并发的情况下该传输协议存在大量的允余性的传输，而且非常占带宽，后来微服务采用JSON代替xml

  2、soa架构模式实现方案webservice或者ESB企业服务总线，底层采用soap协议传输

  WebService架构模式：wsdl

  Wsdl表示：接口的信息、方法、调用地址、参数

  ![](C:\Users\Admin\Desktop\工作\uds\学习\webservice.png)

- ==微服务架构模式==

  **soa架构模式的缺点：**

  1、依赖中心化服务发现机制

  2、使用Soap通讯协议，通常使用XML格式来序列化通讯数据，xml格式非常喜欢重，比较占宽带传输。

  3、服务化管理和治理设施不完善

  4、不适合前后端分离（前后端分离就是对控制层和业务逻辑层进行区分，前端专注与页面跳转和数据渲染，后端专注  	  与业务逻辑）

  **微服务架构基本概念**


​	   1、微服务架构模式是从SOA架构模式演变过来， 比SOA架构模式粒度更加精细，让专业的人去做专业的事情（专注）， 	   目的是提高效率，每个服务与服务之间互不影响，微服务架构中每个服务必须独立部署、互不影响，微服务架构模式	   体现轻巧、轻量级、适合于互联网公司开发模式。
​		2、微服务架构倡导应用程序设计程多个独立、可配置、可运行和可微服务的子服务。
​		3、服务与服务通讯协议采用Http协议，使用restful风格API形式来进行通讯，数据交换格式轻量级json格式通讯，整个传输过程中，采用二进制，所以http协议可以跨语言平台，并且可以和其他不同的语言进行相互的通讯，所以很多开放平台都采用http协议接口。

<img src="C:\Users\Admin\Desktop\工作\uds\学习\微服务.PNG" style="zoom: 65%;" />

​		**微服务架构与SOA架构的不同**

​		1、通讯协议

​				微服务架构基于 SOA架构 演变过来，继承 SOA架构的优点，在微服务架构中去除 SOA 架构中的 ESB 企业服务				总线，采用 http+json（restful）进行传输。
​				**注：**ESB企业服务总线：解决多系统之间跨语言无法实现通讯的问题，对我们数据协议实现转换，可以提供可靠						的消息传输，第三方框架实现。一般情况下都是采用Http+JSON格式传输，所以没有必要使用ESB企业服务						总线。

​		2、服务拆分粒度
​			  微服务架构比 SOA 架构粒度会更加精细，让专业的人去做专业的事情（专注），目的提高效率，每个服务于服务			  之间互不影响，每个服务都是单独的数据库，redis连接，MQ，并且微服务架构中每个服务必须独立部署，微服 			  务架构更加轻巧，轻量级。
​		3、迭代
​			  项目体现特征微服务架构比 SOA 架构更加适合与互联网公司敏捷开发、快速迭代版本，因为粒度非常精细。

​       4、专业度
​             SOA 架构中可能数据库存储会发生共享，微服务强调独每个服务都是单独数据库，保证每个服务于服务之间互不			 影响。

​       **微服务架构会产生那些问题**

​        分布式事务解决方案(rabbitmq/rocketmq/lcn(已经淘汰)/ Seata)

​		分布式任务调度平台(XXL-Job、阿里Scheduler)

​		分布式日志采集系统ELJ+Kafka

​		分布式服务注册中心 eureka、Zookeeper、consule、nacos等

​		分布式服务追踪与调用链Zipkin等

​		非常重要：独立部署、可配置、动态化

​		**为什么我们要使用SpringCloud**

​		SpringCloud并不是rpc远程调用框架，而是一套全家桶的微服务解决框架，理念就是解决我们在微服务架构中遇到的		任何问题。		

​		==rcp调用框架：==httpclient、dubbo、grpc、rest、openfeign

​		**SpringCloud第一代与第二代的区别**

​		SpringCloud 第一代实际上都是用的Netflix开源的组件整合微服务解决方案。

​		SpringCloud第二代实际就是自己研发和国内SpringCloudAlibaba的优秀的微服务解决框架实现整合。

​		<img src="C:\Users\Admin\Desktop\工作\uds\学习\springcloud版本区别.png" style="zoom:60%;" />

​		**SpringCloud第一代：**

​		SpringCloud Config 分布式配置中心

​		SpringCloud Netflix 核心组件：

​				Eureka:服务治理 

​				Hystrix:服务保护框架

​				Ribbon:客户端负载均衡器

​				Feign：基于ribbon和hystrix的声明式服务调用组件

​				Zuul: 网关组件,提供智能路由、访问过滤等功能。

​		**SpringCloud第二代（自己研发）和优秀的组件组合：**

​		Spring Cloud Gateway 网关

​		Spring Cloud Loadbalancer 客户端负载均衡器

​		Spring Cloud r4j(Resilience4J) 服务保护

​		Spring Cloud Alibaba Nacos 服务注册

​		Spring Cloud Alibaba Nacos 分布式配置中心

​		Spring Cloud Alibaba Sentinel服务保护 

​		SpringCloud Alibaba Seata分布式事务解决框架

​		Alibaba Cloud OSS 阿里云存储

​		Alibaba Cloud SchedulerX 分布式任务调度平台

​		Alibaba Cloud SMS 分布式短信系统

​		**springcloud和springcloud的区别？**
​		SpringCloidAlibaba 实际上对我们的 SpringCloud实现了扩展组件,能够完美整合到SpringCloud rpc远程调用整合。
​		1.nacos分布式注册中心、分布式配置中心 SpringCloudEureka+Config组合。
​		2.目的为了推广阿里云产品，如果使用了 SpringCloudAlibaba 建议最好使用alibaba整个体系的产品。
​		Mq rocketmg分布式任务调度X
​		==总结：==SpringCloudAlibaba实际上对SpringCloud做了扩展组件开发nacos、setata分布式解决框架、ScheldulerX、   					Alibaba OSS等，目的是为了推广阿里云产品

### 服务注册与发现Nacos

#### Nacos产生的背景

在rpc远程调用中，服务的url的治理

==rcp调用框架：==httpclient、dubbo、grpc、rest、openfeign

**传统的rpc远程调用存在的问题？**

1、超时问题

2、安全问题

3、服务与服务间的url地址管理

在我们的微服务架构通讯，服务之间依赖关系非常大，如果通过传统的方式管理我们服务的url地址的情况下，一旦地址发生变化的情况下，还需要人工修改rpc远程调用地址。

**服务治理概念： **

在RPC远程调用过程中，服务与服务之间依赖关系非常大，服务Url地址管理非常复杂，所以这时候需要对我们服务的url实现治理，通过服务治理可以实现服务注册与发现、负载均衡、容错等。

每次调用该服务如果地址直接写死的话，一旦接口发生变化的情况下，这时候需要重新发布版本才可以该接口调用地址，所以需要一个注册中心统一管理我们的服务注册与发现。

注册中心：就是存放整个微服务的地址信息，能够实现动态的感知地址信息

注册中心：zookeeper、nacos、Eureka、Consul、redis、数据库

**服务注册：**我们生产者项目启动的时候，会将当前服务自己的信息地址注册到注册中心。

**服务发现: **消费者从我们的注册中心上获取生产者调用的地址（集合），在使用负载均衡的策略获取集群中某个地址实现本地rpc远程调用。

==微服务注册中心实现原理：==

生产者：提供接口被其他服务调用

消费者：调用生产者接口实现消费

<img src="C:\Users\Admin\Desktop\工作\uds\学习\服务注册与发现.png" style="zoom:60%;" />

1、生产者启动的时候 key=服务的名称value ip_和端口号 注册到我们的微服务注册中心

kevin-member 192.168.212.110:8080.

kevin-member 192.168.212.110:8081.

2、注册存放服务地址列表类型：key唯一，列表是list集合。

Map<Key,List(String)>.

{kevin-member:[“192.168.212.110:8080" " 192.168.212.110:8081"]}

3、我们的消费者从我们注册中心上根据服务名称查询服务地址列表（集合）

4、消费者获取到集群列表后，采用负载均衡器选择一个地址进行rpc远程调用

#### Nacos的基本的介绍

Nacos可以实现分布式服务注册与发现/分布式配置中心框架

安装环境

默认账号和密码是nacos/nacos

#### Nacos注册中心

##### springboot整合nacos注册中心

nacos版本2.0.4、spring-cloud.version 2021.0.1、spring-cloud-alibaba.version 2021.0.4.0

添加maven依赖

```xm
<dependencies>
    <!--  springboot 整合web组件-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
</dependencies>
```

修改yml配置

```yaml
spring:
  application:
    ##服务名称
    name: springboot-mybatis
  cloud:
    nacos:
      discovery:
        ##服务的注册地址
        server-addr: 101.35.56.116:8848
```

springcloud、springcloudAlibaba、nacos对应的版本关系

 https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E

==注意：==

Nacos 2.X版本比1.x版本新增grpc的通信方式，需要新增两个端口9848,9849

##### 本地负载均衡器
本地负载均衡器基本的概念：我们的消费者服务从我们的注册中心获取到集群地址列表，缓存到本地，让后本地采用负载均衡策略（轮训、随机、权重等），实现本地的rpc远程的调用

**本地负载均衡器与Nginx 的区别：**

Nginx是客户端所有的请求统一都交给我们的Nginx处理，让后在由Nginx实现负载均衡转发，属于服务器端负载均衡器。

本地负载均衡器是从注册中心获取到集群地址列表，本地实现负载均衡算法，既本地负载均衡器。

应用场景的：

Nginx属于服务器负载均衡，应用于Tomcat/Jetty服务器等，

而我们的本地负载均衡器，应用于在微服务架构中rpc框架中，rest、openfeign、dubbo。

==本地负载均衡器与Nginx 的区别详解：==https://www.jianshu.com/p/22b2c362b973 

SpringCloud rest 和openfeign客户端默认使用ribbon实现负载均衡

Spring Cloud Nacos 2021 放弃Ribbon后 使用LoadBalancer + Nacos做负载均衡 ，因此需要引入另一个jar包

```xml
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```
##### openfeign客户端

SpringCloud第一代采用feign（是Netflix研发），第二代采用openfeign（是springcloud研发）

Openfeign客户端作用：是一个Web声明式的http客户端远程调用工具，底层封装httpclient，默认支持负载均衡

==注：Nacos服务注册中微服务的名称不能有下划线==

#### Nacos分布式配置中心
##### 为什么要使用配置中心？
**在没有配置中心之前，传统应用配置的存在以下痛点：**
（1）采用本地静态配置，无法保证实时性：修改配置不灵活且需要经过较长的测试发布周期，无法尽快通知到客户端，还有些配置对实时性要求很高，比方说主备切换配置或者碰上故障需要修改配置，这时通过传统的静态配置或者重新发布的方式去配置，那么响应速度是非常慢的，业务风险非常大
（2）易引发生产事故：比如在发布的时候，容易将测试环境的配置带到生产上，引发生产事故。
（3）配置散乱且格式不标准：有的用properties格式，有的用xml格式，还有的存DB，团队倾向自造轮子，做法五花八门。
（4）配置缺乏安全审计、版本控制、配置权限控制功能：谁？在什么时间？修改了什么配置？无从追溯，出了问题也无法及时回滚到上一个版本；无法对配置的变更发布进行认证授权，所有人都能修改和发布配置。

**而配置中心区别于传统的配置信息分散到系统各个角落的方式，对系统中的配置文件进行集中统一管理，而不需要逐一对单个的服务器进行管理。那这样做有什么好处呢？**
  （1）通过配置中心，可以使得配置标准化、格式统一化
  （2）当配置信息发生变动时，修改实时生效，无需要重新重启服务器，就能够自动感知相应的变化，并将新的变化统一发送到相应程序上，快速响应变化。比方说某个功能只是针对某个地区用户，还有某个功能只在大促的时段开放，使用配置中心后只需要相关人员在配置中心动态去调整参数，就基本上可以实时或准实时去调整相关对应的业务。
  （3）通过审计功能还可以追溯问题
##### springboot整合nacos配置中心

1、添加 nacos 配置中心的 maven 依赖

```yml
<!-- SpringCloud Ailibaba Nacos Config -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2、在 bootstrap.yml 文件中添加 nacos 配置中心相关配置

```json
 ## 服务配置中心
 config:
    server-addr: 101.35.56.116:8848
    file-extension: yaml # 配置文件类型
        # namespace: public # namespace不写时去默认名称空间获取
        # group: DEFAULT_GROUP # group不写时去默认组获取
        # prefix: nacos-config-a # 表示读取哪个配置文件？不写时默认使用${spring.application.name}
```

3、nacos控制台建立配置集, 分布式配置中心(Spring Cloud Config)遇到的坑

![](C:\Users\Admin\Desktop\工作\uds\学习\nacosbug.PNG)

是因为在springcloud 2020.0.2版本中把bootstrap的相关依赖从spring-cloud-starter-config中移除了，所以现在在2020.02 以后需要单独引入spring-cloud-starter-bootstrap 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
    <version>3.0.3</version>
</dependency>
```

##### Nacos核心概念

###### Data ID 

Data ID 是配置集的唯一标识，一个应用可以包含多个配置集，每个配置集都需要被一个有意义的名称标识。那么 Data ID 怎么取值呢？格式通俗一点就是 “前缀-环境-扩展名”，如下所示：

${spring.cloud.nacos.config.prefix}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}

① prefix：前缀，默认是 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix 来配置。

```xml
# 若不指定，默认采用应用名的方案
spring.application.name=springboot-nacos
 
# 也可以手动指定配置的dataID前缀标识
# spring.cloud.nacos.config.prefix=springboot-nacos-config
```

② active：配置运行环境，即为当前环境对应的 profile。

注意：

当 spring.profiles.active 为空时，对应的连接符 ”-“ 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}

```xml
# dev表示开发环境
spring.profiles.active=dev
```

③ file-exetension：配置文件的类型，默认是 properties，也可以通过配置项 spring.cloud.nacos.config.file-extension 来配置，目前支持的类型有 TEXT、JSON、XML、YAML、HTML、Properties

```xm
# 指定配置文件类型为yaml文件
spring.cloud.nacos.config.file-extension=yaml
```

④ 最终配置：

​    经过前面三个步骤，我们最终在nacos配置中心的控制台新增配置文件就是：springboot-nacos-dev.yaml

<img src="C:\Users\Admin\Desktop\工作\uds\学习\命名规则.PNG" style="zoom: 50%;" />

###### 环境隔离-命名空间Namespace：

Nacos 引入命名空间 Namespace 的概念来进行多环境配置和服务的管理及隔离。例如，你可能存在本地开发环境dev、测试环境test、生产环境prod 三个不同的环境，那么可以创建三个不同的 Namespace 区分不同的环境。创建方式如下：

<img src="C:\Users\Admin\Desktop\工作\uds\学习\nacos-namespace.png" style="zoom:80%;" />

成功创建新命名空间后，就可以在 springboot 的配置文件配置命名空间的 id 切换到对应的命名空间，并获取对应空间下的配置文件，但在没有指定命名空间配置的情况下，默认的配置都是在 public 空间中，指定命名空间的方式如下：

```xml
# 对应创建的命名空间的ID，此处对应的是dev命名空间
cloud.nacos.config.namespace=dev
```
###### 业务隔离-Group分组：

Group 也可以实现环境隔离的功能，但 Group 设计的目的主要是做同一个环境中的不同服务分组，把不同的微服务的配置文件划分到同一个分组里面去，Nacos 如果不指定 Group，则默认的分组是 DEFAULT_GROUP。

        如果没有 Group，试想一下这个场景：有两个微服务，一个是订单系统，一个是用户系统，但是他们有着相同的配置，比如 datasource-url，那么如何区分呢？这时候 Group 就派上用场了。上述场景中订单系统、用户系统可以单独分为一个组，比如 ORDER_GROUP、USER_GROUP，当然这是比较细粒度的分组，根据企业的业务也可以多个微服务分为一组。
接下来在 application.properties 文件分组：

![](C:\Users\Admin\Desktop\工作\uds\学习\命名规则1.PNG.jpg)

```xml
spring.cloud.nacos.config.group=kevin
```

###### 总结：

Nacos 实现配置管理和动态配置刷新很简单，总结如下步骤：

① 添加对应 spring-cloud-starter-alibaba-nacos-config 依赖，修改bootstrap.yml配置
② 使用原生注解 @Value() 导入配置
③ 使用原生注解 @RefreshScope 刷新配置
④ 根据自己业务场景做好多环境配置隔离(Namespace)、不同业务配置隔离(Group)
### SpringCloud Gateway

#### 什么是微服务网关？
微服务网关是整个微服务API请求的入口

作用：可以实现日志拦截、权限控制、解决跨域问题、限流、熔断、负载均衡、黑名单与白名单拦截、授权等。

不采用网关的缺点：

- 微服务中的架构模式采用前后端分离，前端调用接口的地址都可以被抓包分析到

- 传统方式我们可以使用过滤器拦截用户的会话信息，这个过程所有的服务都要写入验证登录的代码，造成代码允余

![](C:\Users\Admin\Desktop\工作\uds\学习\gateway.PNG)

#### 过滤器与网关的区别？

过滤器适合于单个服务实现过滤请求（局部拦截）

网关主要用于拦截整个微服务的过滤请求，能够解决整个微服务的允余代码（全局拦截）

#### Zuul与Gateway的区别？
Zuul网关属于netfix公司开源的产品属于第一代微服务网关
Gateway属于SpringCloud自研发的第二代微服务网关，相比来说SpringCloudGateway性能比Zuul性能要好。

**注意：**

Zuul基于Servlet实现的，阻塞式的Api， 不支持长连接。依赖Springboot-Web

SpringCloudGateway基于Spring5构建，能够实现响应式非阻塞式的Api，支持长连接，能够更好的整合Spring体系的产品。依赖Springboot-WebFux

#### SpringCloud GateWay搭建

**添加相关依赖**

**注意**：一定要排除掉 spring-boot-starter-web 依赖，否则启动报错

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>3.1.1</version>
</dependency>
```

##### Spring Cloud Gateway 配置项的说明：

**路由 Route：**由ID、目标URI、断言集合和过滤器集合组成。如果聚合断言结果为真，则转发到该路由。

Route 主要由 路由id、目标uri、断言集合和过滤器集合组成，那我们简单看看这些属性到底有什么作用。

（1）id：路由标识，要求唯一，名称任意（默认值 uuid，一般不用，需要自定义）

（2）uri：请求最终被转发到的目标地址

（3）order： 路由优先级，数字越小，优先级越高

（4）predicates：断言数组，即判断条件，如果返回值是boolean，则转发请求到 uri 属性指定的服务中

（5）filters：过滤器数组，在请求传递过程中，对请求做一些修改

**断言 Predicate:**Predicate 来自于 Java8 的接口。Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）

Predicate 可以用于接口请求参数校验、判断新老数据是否有变化需要进行更新操作

11种断言这里就不再介绍如何配置了，官方文档写的很清楚：https://docs.spring.io/spring-cloud-gateway/docs/2.2.9.RELEASE/reference/html/

下面就以最后一种权重断言为例介绍一下如何配置。配置如下：

```xml
spring:
  cloud:
    gateway:
      # 路由数组：指当请求满足什么样的断言时，转发到哪个服务上
      routes:
        # 路由标识，要求唯一，名称任意
        - id: gateway-provider_1
		  # 请求最终被转发到的目标地址
          uri: http://localhost:9024
          # 设置断言
          predicates:
            # Path Route Predicate Factory 断言，满足 /gateway/provider/** 路径的请求都会被路由到 http://localhost:9024 这个uri中
            - Path=/gateway/provider/**
            # Weight Route Predicate Factory 断言，同一分组按照权重进行分配流量，这里分配了80%
            # 第一个group1是分组名，第二个参数是权重
            - Weight=group1, 8
          # 配置过滤器（局部）
          filters:
            # StripPrefix：去除原始请求路径中的前1级路径，即/gateway
            - StripPrefix=1            
            
        - id: gateway-provider_2
          uri: http://localhost:9025
          # 设置断言
          predicates:
            - Path=/gateway/provider/**
            # Weight Route Predicate Factory，同一分组按照权重进行分配流量，这里分配了20%
            - Weight=group1, 2
		  # 配置过滤器（局部）
          filters:
            # StripPrefix：去除原始请求路径中的前1级路径，即/gateway
            - StripPrefix=1            
```

