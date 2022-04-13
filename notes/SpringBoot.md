# SpringBoot

## 快速入门

- springboot在创建项目时，使用 jar的打包方式
- springboot的引导类，是项目入口，运行 main方法就可以启动项目
- 使用 springboot和 spring构建的项目，业务代码编写方式完全一样

## 起步依赖原理分析

- 在 `spring-boot-starter-parent`中定义了各种技术的版本信息，组合了一套最优搭配的技术版本
- 在各种 starter中，定义了完成该功能需要的坐标合集，其中大部分版本信息来自于父工程
- 我们的工程继承 parent，引入 starter后，通过依赖传递，就可以简单方便获得需要的 jar包，并且不会存在版本冲突等问题

## 配置

### 配置文件分类

- springboot提供了2中配置文件类型： `properties`和`yml/yaml`
- 默认配置文件名称：application
- 在同一级目录下优先级为：properties > yml > yaml

### yaml基本语法

- 大小写敏感
- 数据值前边必须有空格，作为分隔符
- 使用缩进表示层级关系
- 缩进时不允许使用 tab键，只允许使用空格（防止 tab对应空格数不统一）
- 缩进的空格数目不重要，只要同一层级的元素左侧对齐即可
- `#`表示注释，从这个字符到行尾，都会被解析器忽略

```yaml
server:
	port: 8081
	address: 127.0.0.1

name: abd
```

### yaml数据格式

1. 配置文件类型
   - `properties`: 和以前一样
   - `yml/yaml`: 注意空格
2. yaml：简洁，以数据为核心
   - 基本语法
     - 大小写敏感
     - 数据值前边必须有空格，作为分隔符
     - 使用空格缩进表示层级关系，相同缩进表示同一级
   - 数据格式
     - 对象
     - 数组：使用 `-`表示数组每一元素
     - 纯量
   - 参数引用
     - `${key}`

### 读取配置内容

- `@Value`
- `Environment`
- `@ConfigurationProperties`

### Profile

1. profile是用来完成不同环境下，配置动态切换功能的
2. profile配置方式
   - 多 profile文件方式：提供多个配置文件，每个代表一种环境
     - `application-dev.properties/yml` 开发环境
     - `application-test.properties/yml`测试环境
     - `application-pro.properties/yml`生产环境
   - yml多文档方式
     - 在yml中使用 `---`分隔不同配置
3. profile激活方式
   - 配置文件
   
     ```properties
     # 在配置文件中配置
     spring.profiles.active=dev
     ```
   
   - 虚拟机参数
   
     ```
     在 vm options指定
     -Dspring.profiles.active=dev
     ```
   
   - 命令行参数

		```shell
		--spring.profiles.active=pro
		
		java -jar xxx.jar --spring.profiles.active=pro
		```

### 内部配置加载顺序

springboot程序启动时，会从以下位置加载配置文件：

1. `file:./config/` 当前项目下的 /config目录下
2. `file:./` 当前项目的根目录
3. `classpath:/config/` classpath的 /config目录
4. `classpath:/` classpath的根目录

加载顺序为上文的排列顺序，高优先级配置的属性会生效

### 外部配置加载顺序

通过官网查看外部属性加载顺序

[https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

## 整合其他框架

### 整合 Junit

#### 实现步骤

1. 搭建 springboot工程
2. 引入 `spring-boot-starter-test`起步依赖
3. 编写测试类
4. 添加测试相关注解
   - `@RunWith(SpringRunner.class)` 新版不需要
   - `@SpringBootTest(classes=启动类.class)`
5. 编写测试方法

### 整合 Redis

#### 实现步骤

1. 搭建 springboot工程
2. 引入 redis起步依赖
3. 配置 redis相关属性
4. 注入 `RedisTemplate`模板
5. 编写测试方法，测试

### 整合 MyBatis

#### 实现步骤

1. 搭建 springboot工程
2. 引入 mybatis起步依赖，添加 mysql驱动
3. 编写 DataSource和 MyBatis相关配置
4. 定义表和实体类
5. 编写 dao和 mapper文件/纯注解开发
6. 测试

## Spring原理分析

### SpringBoot自动配置

#### Condition

##### 简介

`Condition`类是在 Spring4.0增加的条件判断功能，通过这个功能可以实现选择性的创建 **Bean**操作

##### 自定义条件

- 自定义条件类：自定义类实现 Condition接口，重写 matches方法，在 matches方法中进行逻辑判断，返回 boolean值。matches方法两个参数：
  - `context` 上下文对象，可以获取属性值，获取类加载器，获取 BeanFactory等
  - `metadata` 元数据对象，用于获取注解属性
- 判断条件：在初始化 Bean时，使用 `@Conditional(条件类.class)`注解

##### SpringBoot提供的常用注解

- `ConditionalOnProperty` 判断配置文件中是否有对应属性和值才初始化 Bean
- `ConditionalOnClass` 判断环境中是否有对应字节码文件才初始化 Bean
- `ConditionalMissingBean` 判断环境中没有对应 Bean才初始化 Bean

#### 切换内置 web服务器

##### 简介

SpringBoot的 web环境中默认使用 tomcat作为内置服务器，其实 SpringBoot提供了 4个内置服务器供我们选择，我们可以很方便的进行切换

![内置服务器](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/6439b9537ff9f336095f4998c25780a58dc66415a30b685f4a759e5ce738930b-20220413101907541-2022-04-13-b43a26eff9a276c1989e5f79883f1a5b-00cc43.png)

##### 配置

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  	<!-- 排除 tomcat依赖 -->
    <exclusions>
        <exclusion>
            <groupId>spring-boot-starter-tomcat</groupId>
            <artifactId>org.springframework.boot</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 引入 jetty依赖 -->
<dependency>
    <groupId>spring-boot-starter-jetty</groupId>
    <artifactId>org.springframework.boot</artifactId>
</dependency>
```

#### @Enable*注解

##### 简介

SpringBoot中提供了很多 Enable开头的注解，这些注解都是用于动态启用某些功能的。而其底层原理是使用 `@Import`注解导入一些配置类，实现 **Bean**的动态加载

##### 配置

```java
// 在三方包中制作引用注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(UserConfig.class)
public @interface EnableUser {}

// 在主入口使用注解
@SpringBootApplication
@EnableUser
public class BackendApplication {
    public static void main(String[] args) {
        ConfigurationApplicationContext context = SpringApplication.run(BackendApplication.class, args);
      	Object user = context.getBean("user");
      	System.out.println(user);
    }
}
```

#### @Import注解

##### 简介

`@Enable*`注解底层依赖于 `@Import`注解导入一些类，使用 `@Import`导入的类会被 Spring加载到 IOC容器中。而 `@Import`注解提供了 4种用法：

- 导入**Bean**
- 导入配置类
- 导入 `ImportSelector`实现类。一般用于加载配置文件中的类
- 导入 `ImportBeanDefinitionRegistrar`实现类

#### @EnableAutoConfiguration注解

- `@EnableAutoConfiguration`注解内部使用 `@Import(AutoConfigurationImportSelector.class)`来加载配置类
- 配置文件位置：**META-INF/spring.factories**，该配置文件中定义了大量的配置类，当 SpringBoot应用启动时，会自动加载这些配置类，初始化 **Bean**
- 并不是所有的 **Bean**都会被初始化，在配置类中使用 Condition来加载满足条件的 **Bean**

#### 自定义配置

##### 需求

自定义 redis-starter。要求当导入 redis坐标时，SpringBoot自动创建 Jedis的 Bean

##### 实现步骤

1. 创建 redis-spring-boot-autoconfigure模块
2. 创建 redis-spring-boot-starter模块，依赖 redis-spring-boot-autoconfigure的模块
3. 在 redis-spring-boot-autoconfigure模块中初始化 jedis的 bean，并定义 META-INF/spring.factories文件
4. 在测试模块中引入自定义的 redis-starter依赖，测试获取 jedis的 bean，操作 redis

### SpringBoot监听机制

#### Java监听机制

SpringBoot的监听机制，其实就是对 java提供的时间监听机制的封装

java中的事件监听机制定义了以下几个角色：

- 事件：Event，继承 java.util.EventObject类的对象
- 事件源：Source，任意对象 Object
- 监听器：Listener，实现 java.util.EventListener接口的对象

#### SpringBoot监听机制

SpringBoot在项目启动时，会对几个监听器进行回调，我们可以实现这些监听器接口，在项目启动时完成一些操作

- ApplicationContextInitializer
- SpringApplicationRunListener
- CommandLineRunner
- ApplicationRunner

### SpringBoot启动流程分析

![启动流程图](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/78ad7cb5a32fea2296b59bc4e14e9fde223a21c46cf2613861d149d06b7ce727-20220413101914496-2022-04-13-6b09c55026baa21a5e4d296d3f6db23b-9223c6.png)

## SpringBoot监控

### Actuator

#### 简介

SpringBoot自带监控功能 **Actuator**，可以帮助实现对程序内部运行情况监控，比如监控状况、Bean加载情况、配置属性、日志信息等

#### 监控使用

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. 访问 [http://localhost:8080/acruator](http://localhost:8080/acruator)

### SpringBootAdmin

#### 简介

- SpringBootAdmin是一个开源社区项目，用于管理和监控 SpringBoot应用程序
- SpringBootAdmin有两个角色，客户端（Client）和服务端（Server） 
- 应用程序作为 SpringBootAdmin Client向为 SpringBootAdmin Server注册
- SpringBootAdmin Server的 UI界面将 SpringBootAdmin Client的 Actuator EndPoint上的一些监控信息

#### 使用步骤

##### admin-server

- 创建 admin-server模块
- 导入依赖坐标 admin-starter-server
- 在引导类上启动监控功能 `@EnableAdminServer`

##### admin-client

- 创建 admin-client模块
- 导入依赖坐标 admin-starter-client
- 配置相关信息：server地址等
- 启动 server和 client服务，访问 server

## SpringBoot项目部署

SpringBoot项目开发完成后，支持两种方式部署到服务器：

1. jar包（官方推荐）
2. war包
