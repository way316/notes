# Spring Boot 2 学习笔记

## Spring Boot 2 核心组件

### Web开发

#### Web原生组件注入（Servlet、Filter、Listener）

##### 1. 使用原生Servlet API

@ServletComponentScan(basePackages = "com.wy") :在Spring主程序里需要使用此注解来扫描指定的包从而招到原生Servlet组件

@WebServlet(urlPatterns = "/my")：直接响应，没有经过Spring的拦截器

@WebFilter(urlPatterns={"/css/*","/images/*"}) 

@WebListener

##### 2.使用RegistrationBean

ServletRegistrationBean

FilterRegistrationBean

ServletListenerRegistrationBean

```java
@Configuration (proxyBeanMethod = true) //确保以来的组件始终是单实例的
public class MyRegistConfig {
	
    //注册一个severlet的Bean
    @Bean
    public ServletRegistrationBean myServlet(){
        MyServlet myServlet = new MyServlet();
        return new ServletRegistrationBean(myServlet,"/my","/my02");
    }


    @Bean
    public FilterRegistrationBean myFilter(){

        MyFilter myFilter = new MyFilter();
//      return new FilterRegistrationBean(myFilter,myServlet());
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/my","/css/*"));
        return filterRegistrationBean;
    }

    @Bean
    public ServletListenerRegistrationBean myListener(){
        MySwervletContextListener mySwervletContextListener = new MySwervletContextListener();
        return new ServletListenerRegistrationBean(mySwervletContextListener);
    }
}
```

#### 嵌入式Servlet容器

##### 1、切换嵌入式Servlet容器

###### 默认支持的webServer

​	Tomcat, Jetty, Undertow

​	ServletWebServerApplicationContext 容器启动寻找ServletWebServerFactory 并引导创建服务器

切换服务器：

```xml
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
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```



###### 原理

 1. SpringBoot应用启动发现当前是Web应用，web场景包里导入了tomcat（默认）

 2. web应用会创建一个web版的ioc容器 ServletWebServerApplicationContext

 3. ServletWebServerApplicationContext  启动的时候寻找 ServletWebServerFactory（Servlet 的web服务器工厂---> Servlet 的web服务器）

 4. SpringBoot底层默认有很多的WebServer工厂；

    TomcatServletWebServerFactory,

    JettyServletWebServerFactory

    UndertowServletWebServerFactory

5. Spring使用ServletWebServerFactoryAutoConfiguration来自动配置webserver工厂
6. ServletWebServerFactoryAutoConfiguration导入了ServletWebServerFactoryConfiguration
7. ServletWebServerFactoryConfiguration配置类 根据动态判断系统中到底导入了那个Web服务器的包。（默认是web-starter导入tomcat包），容器中就有 TomcatServletWebServerFactory
8. TomcatServletWebServerFactory 创建出Tomcat服务器并启动；TomcatWebServer 的构造器拥有初始化方法initialize---this.tomcat.start();
9. 内嵌服务器，就是手动把启动服务器的代码调用（tomcat核心jar包存在）

##### 2. 定制Servlet容器

- 实现  WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> 

  ​	把配置文件的值和ServletWebServerFactory 进行绑定，这个步骤在创建ServletWebServerFactory之后

- 修改配置文件 server.xxx

- 直接自定义 ConfigurableServletWebServerFactory(使用自定义的方法来代替默认方法)

```Java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
        server.setPort(9000);
    }

}
```

#### 定制化原理

maven里导入场景starter - xxxxAutoConfiguration - 导入xxx组件（@Bean） - 绑定xxxProperties -- 绑定配置文件项

##### 定制化方式

修改配置文件；

xxxxxCustomizer； 

编写自定义的配置类   xxxConfiguration；+ @Bean替换、增加容器中默认组件,视图解析器 

Web应用 编写一个配置类实现 WebMvcConfigurer 即可定制化web功能；+ @Bean给容器中再扩展一些组件

```Java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer
```

- @EnableWebMvc + WebMvcConfigurer —— @Bean  可以全面接管SpringMVC，所有规则全部自己重新配置； 实现定制和扩展功能

  - 1、WebMvcAutoConfiguration  默认的SpringMVC的自动配置功能类。静态资源、欢迎页.....
  - 2、一旦使用 @EnableWebMvc 、。会 @Import(**DelegatingWebMvcConfiguration**.class)

  - 3、**DelegatingWebMvcConfiguration** 的 作用，只保证SpringMVC最基本的使用
    - 把所有系统中的 WebMvcConfigurer 拿过来。所有功能的定制都是这些 WebMvcConfigurer  合起来一起生效
    - 自动配置了一些非常底层的组件。RequestMappingHandlerMapping、这些组件依赖的组件都是从容器中获取
    - public class **DelegatingWebMvcConfiguration** extends WebMvcConfigurationSupport

  - 4、WebMvcAutoConfiguration 里面的配置要能生效 必须  @ConditionalOnMissingBean(WebMvcConfigurationSupport.class) 
  - 5、@EnableWebMvc  导致了 WebMvcAutoConfiguration  没有生效

### 数据访问

#### SQL

##### 1.数据源的自动配置-HikariDataSource

###### 1.导入JDBC

```XML
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
```

![image-20210824181211567](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210824181211567.png)

SpringBoot不知道需要的驱动版本所以需要自己添加DB驱动

```xml
默认版本：<mysql.version>8.0.22</mysql.version>

想要修改版本
1、直接依赖引入具体版本（maven的就近依赖原则）
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
2、重新声明版本（maven的属性的就近优先原则）
    <properties>
        <mysql.version>5.1.49</mysql.version>
    </properties>
```

###### 2. 自动配置的类：

- DataSourceAutoConfiguration ： 数据源的自动配置

  - 修改数据源相关的配置：**spring.datasource**

  - **数据库连接池的配置，是自己容器中没有DataSource才自动配置的**

  - ```java
    	@Configuration(proxyBeanMethods = false)
    	@Conditional(PooledDataSourceCondition.class)
    	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
    	@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
    			DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.OracleUcp.class,
    			DataSourceConfiguration.Generic.class, DataSourceJmxConfiguration.class })
    	protected static class PooledDataSourceConfiguration
    ```

    

  - 底层配置好的连接池是：**HikariDataSource**

- DataSourceTransactionManagerAutoConfiguration： 事务管理器的自动配置
- JdbcTemplateAutoConfiguration： JdbcTemplate的自动配置，可以来对数据库进行crud
  - 可以修改这个配置项@ConfigurationProperties(prefix = "spring.jdbc") 来修改JdbcTemplate
  - @Bean@Primary    JdbcTemplate；容器中有这个组件

- JndiDataSourceAutoConfiguration： jndi的自动配置
- XADataSourceAutoConfiguration： 分布式事务相关的

###### 3. 配置项

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```



##### 2.使用Druid数据源

###### 1. 自定义方式

1. 引入Druid包

   ```xml
           <dependency>
               <groupId>com.alibaba</groupId>
               <artifactId>druid</artifactId>
               <version>1.1.17</version>
           </dependency>
   ```

2. 自己配置Druid

   ```java
   @Configuration
   public class MyDataSourceConfig() {
       //ConfigurationProperties绑定了配置文件里的配置数据
       @ConfigurationProperties("spring.datasource")
   	@Bean
   	public DataSource dataSource() {
   		DuridDataSource duridDataSource = new DuridDataSource();
            //加入监控功能
   		druidDataSource.setFilters("stat,wall");
   		druidDataSource.setMaxActive(10);
   		return duridDataSource;
   	}
   }
   ```

3. 配置StatViewServlet（用于监控Druid）

   ```java
       // 配置 druid的监控页功能
       @Bean
       public ServletRegistrationBean statViewServlet(){
           StatViewServlet statViewServlet = new StatViewServlet();
           ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<>(statViewServlet, "/druid/*");
   
           registrationBean.addInitParameter("loginUsername","admin");
           registrationBean.addInitParameter("loginPassword","123456");
   
   
           return registrationBean;
       }
   ```

4. 配置StatFilter（用于统计监控信息；如SQL监控、URI监控）

   ```Java
       //用于采集web-jdbc关联监控的数据。
       @Bean
       public FilterRegistrationBean webStatFilter(){
           WebStatFilter webStatFilter = new WebStatFilter();
   
           FilterRegistrationBean<WebStatFilter> filterRegistrationBean = new FilterRegistrationBean<>(webStatFilter);
           
           filterRegistrationBean.setUrlPatterns(Arrays.asList("/*"));
           
   filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
   
           return filterRegistrationBean;
    }
   ```


###### 2. 自动配置

```XML
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.17</version>
        </dependency>
```

2、分析自动配置

- 扩展配置项 **spring.datasource.druid**
- DruidSpringAopConfiguration.**class**,   监控SpringBean的；配置项：**spring.datasource.druid.aop-patterns**

- DruidStatViewServletConfiguration.**class**, 监控页的配置：**spring.datasource.druid.stat-view-servlet；默认开启**
-  DruidWebStatFilterConfiguration.**class**, web监控配置；**spring.datasource.druid.web-stat-filter；默认开启**

- DruidFilterConfiguration.**class**}) 所有Druid自己filter的配置

```Java
    private static final String FILTER_STAT_PREFIX = "spring.datasource.druid.filter.stat";
    private static final String FILTER_CONFIG_PREFIX = "spring.datasource.druid.filter.config";
    private static final String FILTER_ENCODING_PREFIX = "spring.datasource.druid.filter.encoding";
    private static final String FILTER_SLF4J_PREFIX = "spring.datasource.druid.filter.slf4j";
    private static final String FILTER_LOG4J_PREFIX = "spring.datasource.druid.filter.log4j";
    private static final String FILTER_LOG4J2_PREFIX = "spring.datasource.druid.filter.log4j2";
    private static final String FILTER_COMMONS_LOG_PREFIX = "spring.datasource.druid.filter.commons-log";
    private static final String FILTER_WALL_PREFIX = "spring.datasource.druid.filter.wall";
```

3.配置示例

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

    druid:
      aop-patterns: com.atguigu.admin.*  #监控SpringBean
      filters: stat,wall     # 底层开启功能，stat（sql监控），wall（防火墙）

      stat-view-servlet:   # 配置监控页功能
        enabled: true
        login-username: admin
        login-password: admin
        resetEnable: false

      web-stat-filter:  # 监控web
        enabled: true
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'


      filter:
        stat:    # 对上面filters里面的stat的详细配置
          slow-sql-millis: 1000
          logSlowSql: true
          enabled: true
        wall:
          enabled: true
          config:
            drop-table-allow: false
```



##### 3. 整合Mybatis

###### 1. 引入starter

```xml
      <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
```

###### 2.自动配置

- 全局配置文件
- SqlSessionFactory: 自动配置好了

- SqlSession：自动配置了 **SqlSessionTemplate 组合了SqlSession**
- @Import(**AutoConfiguredMapperScannerRegistrar**.**class**）自动扫描@Mapper；

- Mapper： 只要我们写的操作MyBatis的接口标准了 **@Mapper 就会被自动扫描进来**

###### 3.开发步骤

- 编写mapper接口。标准@Mapper注解

```java
@Mapper
public interface AccountMapper {
	public Account getAcct(Long id);
}
```

- 编写sql映射文件并绑定mapper接口
- 在application.yaml中指定Mapper配置文件的位置，以及指定全局配置文件的信息 （建议；**配置在mybatis.configuration**）

```xml
# 配置mybatis规则(ymal)
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml  #全局配置文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml  #sql映射文件位置
  configuration:
    map-underscore-to-camel-case: true //开启驼峰命名自动匹配

Mapper接口--->绑定Xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.admin.mapper.AccountMapper">
<!--    public Account getAcct(Long id); -->
    <select id="getAcct" resultType="com.atguigu.admin.bean.Account">
        select * from  account_tbl where  id=#{id}
    </select>
</mapper>
```



###### 3. 注解模式，去除mapper.xml文件

```java
@Mapper
public interface CityMapper {

    @Select("select * from city where id=#{id}")
    public City getById(Long id);

    @Insert("insert into city ('name','state','country') values (#{name},#{state},#{country}")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    public void insert(City city);

}
```

因为insert比较复杂，insert可以写在xml里面，所以支持注解+配置双模式。

可以在主程序使用*@MapperScan("com.atguigu.admin.mapper")*就不需要使用@mapper标签了



##### 4. Mybatis-Plus

###### 自动配置

- MybatisPlusAutoConfiguration 配置类，MybatisPlusProperties 配置项绑定。**mybatis-plus：xxx 就是对****mybatis-plus的定制**
- **SqlSessionFactory 自动配置好。底层是容器中默认的数据源**

- **mapperLocations 自动配置好的。有默认值。****classpath\*:/mapper/\**/\*.xml；任意包的类路径下的所有mapper文件夹下任意路径下的所有xml都是sql映射文件。  建议以后sql映射文件，放在 mapper下**
- **容器中也自动配置好了** **SqlSessionTemplate**

- **@Mapper 标注的接口也会被自动扫描；建议直接** @MapperScan(**"com.atguigu.admin.mapper"**) 批量扫描就行

Mybatis-Plus的优点

- 只需要我们的Mapper继承 **BaseMapper<Class>** 就可以拥有crud能力，BaseMapper里面写好了基本的CRUD。

- 不仅仅是dao/mapper层，连service都可以简化

  ```Java
  @Service
  public class UserServiceImpl extends ServiceImpl<UserMapper,User> implements UserService {
  
  
  }
  
  public interface UserService extends IService<User> {
  
  }
  ```

  ServiceImpl<需要操作的mapper,需要返回的类型>里面写了默认的增删改查方法，让service类继承。



##### 5.Redis

1. Redis自动配置

   引入自动配置包

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
   ```

   自动配置：

   - RedisAutoConfiguration 自动配置类。RedisProperties 属性类 --> **spring.redis.xxx是对redis的配置**
   - 连接工厂是准备好的。**Lettuce**ConnectionConfiguration、**Jedis**ConnectionConfiguration（两张客户端操作Redis）

   - **自动注入了RedisTemplate**<**Object**, **Object**> ： xxxTemplate；
   - **自动注入了StringRedisTemplate；<k：v>都是String**

   - **key：value**
   - **底层只要我们使用** **StringRedisTemplate、****RedisTemplate就可以操作redis**

//先跳过

2. Redis 与Lettuce

   ```java
       @Test
       void testRedis(){
           ValueOperations<String, String> operations = redisTemplate.opsForValue();
   
           operations.set("hello","world");
   
           String hello = operations.get("hello");
           System.out.println(hello);
       }
   ```

3. 切换到jedis

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
   
   <!--        导入jedis-->
           <dependency>
               <groupId>redis.clients</groupId>
               <artifactId>jedis</artifactId>
           </dependency>
   ```

   ```yaml
   spring:
     redis:
         host: r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com
         port: 6379
         password: lfy:Lfy123456
         client-type: jedis
         jedis:
           pool:
             max-active: 10
   ```



### 单元测试

#### 1.Junit 介绍

**JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**

**JUnit Platform**: Junit Platform是在JVM上启动测试框架的基础，不仅支持Junit自制的测试引擎，其他测试引擎也都可以接入。

**JUnit Jupiter**: JUnit Jupiter提供了JUnit5的新的编程模型，是JUnit5新特性的核心。内部 包含了一个**测试引擎**，用于在Junit Platform上运行。

**JUnit Vintage**: 由于JUint已经发展多年，为了照顾老的项目，JUnit Vintage提供了兼容JUnit4.x,Junit3.x的测试引擎

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

**SpringBoot 2.4 以上版本移除了默认对** **Vintage 的依赖。如果需要兼容junit4需要自行引入（不能使用junit4的功能 @Test****）**

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

SpringBoot整合Junit以后。

- 编写测试方法：@Test标注（注意需要使用junit5版本的注解）
- Junit类具有Spring的功能，@Autowired、比如 @Transactional 标注测试方法，测试完成后自动回滚

#### 2. JUnit5常用注解

- **@Test :**表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- **@ParameterizedTest :**表示方法是参数化测试，下方会有详细介绍

- **@RepeatedTest :**表示方法可重复执行，下方会有详细介绍
- **@DisplayName :**为测试类或者测试方法设置展示名称

- **@BeforeEach :**表示在每个单元测试之前执行
- **@AfterEach :**表示在每个单元测试之后执行

- **@BeforeAll :**表示在所有单元测试之前执行
- **@AfterAll :**表示在所有单元测试之后执行

- **@Tag :**表示单元测试类别，类似于JUnit4中的@Categories
- **@Disabled :**表示测试类或测试方法不执行，类似于JUnit4中的@Ignore

- **@Timeout :**表示测试方法运行如果超过了指定时间将会返回错误
- **@ExtendWith :**为测试类或测试方法提供扩展类引用(用来对接其他平台)

如果需要用spring的功能，需要使用@SpringBootTest

#### 3. 断言

**断言方法都是 org.junit.jupiter.api.Assertions 的静态方法**, 会设定预期来判断输出是否符合预期。

1. 简单断言

   | assertEquals    | 判断两个对象或两个原始类型是否相等   |
   | :-------------- | :----------------------------------- |
   | assertNotEquals | 判断两个对象或两个原始类型是否不相等 |
   | assertSame      | 判断两个对象引用是否指向同一个对象   |
   | assertNotSame   | 判断两个对象引用是否指向不同的对象   |
   | assertTrue      | 判断给定的布尔值是否为 true          |
   | assertFalse     | 判断给定的布尔值是否为 false         |
   | assertNull      | 判断给定的对象引用是否为 null        |
   | assertNotNull   | 判断给定的对象引用是否不为 null      |

   ```Java
   @Test
   @DisplayName("simple assertion")
   public void simple() {
        assertEquals(3, 1 + 2, "simple math");
        assertNotEquals(3, 1 + 1);
   
        assertNotSame(new Object(), new Object());
        Object obj = new Object();
        assertSame(obj, obj);
   
        assertFalse(1 > 2);
        assertTrue(1 < 2);
   
        assertNull(null);
        assertNotNull(new Object());
   }
   ```

   

2. 数组断言

   判断两个对象或原始类型的数组是否相等

   ```java
   @Test
   @DisplayName("array assertion")
   public void array() {
    assertArrayEquals(new int[]{1, 2}, new int[] {1, 2});
   }
   ```

   

3. 组合断言

   接受多个 org.junit.jupiter.api.Executable 函数式接口的实例作为要验证的断言，可以通过 lambda 表达式很容易的提供这些断言,必须两个都成功才会返回成功

   ```Java
   @Test
   @DisplayName("assert all")
   public void all() {
    assertAll("Math",
       () -> assertEquals(2, 1 + 1),
       () -> assertTrue(1 > 0)
    );
   }
   ```

   

4. 异常断言

   JUnit5提供了一种新的断言方式**Assertions.assertThrows()** ,配合函数式编程就可以进行使用,断定一定会出现异常，没异常才返回测试失败。

   ```java
   @Test
   @DisplayName("异常测试")
   public void exceptionTest() {
       ArithmeticException exception = Assertions.assertThrows(
              //扔出断言异常
               ArithmeticException.class, () -> System.out.println(1 % 0));
   }
   ```

   

5. 超时断言

   **Assertions.assertTimeout()** 为测试方法设置了超时时间

   ```java
   @Test
   @DisplayName("超时测试")
   public void timeoutTest() {
       //如果测试方法时间超过1s将会异常
       Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(500));
   }
   ```

   

6. 快速失败

   通过 fail 方法直接使得测试失败

   ```java
   @Test
   @DisplayName("fail")
   public void shouldFail() {
    fail("This should fail");
   }
   ```



#### 4.前置条件

不满足的**前置条件只会使得测试方法的执行终止**

assumeTrue 和 assumFalse 确保给定的条件为 true 或 false，不满足条件会使得测试执行终止。assumingThat 的参数是表示条件的布尔值和对应的 Executable 接口的实现对象。只有条件满足时，Executable 对象才会被执行；当条件不满足时，测试执行并不会终止

```JAVA
@Test
    void testAssumThat() {
        assumingThat(3>5,
                () -> {
                    //与上述方法不同的是，仅当前面假设成立时，才会执行这里面的语句！！！！
                    // 且只会影响到该lambda表达式中的代码
                    assertEquals(2, 2);
                });
        //此处的断言不受上述assumingThat限制，在所有情况下都会执行
        System.out.println("no effect");
        assertEquals("a string", "a string");
    }
```



#### 5.嵌套测试

有助于创建层次结构上下文，以将相关的单元测试组合在一起 在内部类上需要使用@Nested

```java
@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }
        }
    }
}
```

重点知识：内层的@Test会调用外层的@BeforeEach, 外层的不会调用内层的



#### 6.参数化测试

允许使用不同的参数进行多次测试

**@ValueSource**: 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型

**@NullSource**: 表示为参数化测试提供一个null的入参

**@EnumSource**: 表示为参数化测试提供一个枚举入参

**@CsvFileSource**：表示读取指定CSV文件内容作为参数化测试入参

**@MethodSource**：表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流)



```Java
@ParameterizedTest
@ValueSource(strings = {"one", "two", "three"})
@DisplayName("参数化测试1")
public void parameterizedTest1(String string) {
    System.out.println(string);
    Assertions.assertTrue(StringUtils.isNotBlank(string));
}


@ParameterizedTest
@MethodSource("method")    //指定方法名
@DisplayName("方法来源参数")
public void testWithExplicitLocalMethodSource(String name) {
    System.out.println(name);
    Assertions.assertNotNull(nam	e);
}

static Stream<String> method() {
    return Stream.of("apple", "banana");
}
```



#### 7.迁移至南

在进行迁移的时候需要注意如下的变化：

- 注解在 org.junit.jupiter.api 包中，断言在 org.junit.jupiter.api.Assertions 类中，前置条件在 org.junit.jupiter.api.Assumptions 类中。
- 把@Before 和@After 替换成@BeforeEach 和@AfterEach。

- 把@BeforeClass 和@AfterClass 替换成@BeforeAll 和@AfterAll。
- 把@Ignore 替换成@Disabled。

- 把@Category 替换成@Tag。
- 把@RunWith、@Rule 和@ClassRule 替换成@ExtendWith。



### 指标监控

#### 1. SpringBoot Actuator

SpringBoot抽取了Actuator场景，使得我们每个微服务快速引用即可获得生产级别的应用监控、审计等功能

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```



使用步骤：

- 引入场景

- 访问 http://localhost:8080/actuator/**
- 暴露所有监控信息为HTTP(默认使用JMX方式暴露信息)

```Yaml
management:
  endpoints:
    enabled-by-default: true #暴露所有端点信息
    web:
      exposure:
        include: '*'  #以web方式暴露
  endpoint: #对某个端点进行配置
  	health:
      show-details: always
```

Actuator Endpoint(监控端点)

| ID                 | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `auditevents`      | 暴露当前应用程序的审核事件信息。需要一个`AuditEventRepository组件`。 |
| `beans`            | 显示应用程序中所有Spring Bean的完整列表。                    |
| `caches`           | 暴露可用的缓存。                                             |
| `conditions`       | 显示自动配置的所有条件信息，包括匹配或不匹配的原因。         |
| `configprops`      | 显示所有`@ConfigurationProperties`。                         |
| `env`              | 暴露Spring的属性`ConfigurableEnvironment`                    |
| `flyway`           | 显示已应用的所有Flyway数据库迁移。 需要一个或多个`Flyway`组件。 |
| `health`           | 显示应用程序运行状况信息。                                   |
| `httptrace`        | 显示HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应）。需要一个`HttpTraceRepository`组件。 |
| `info`             | 显示应用程序信息。                                           |
| `integrationgraph` | 显示Spring `integrationgraph` 。需要依赖`spring-integration-core`。 |
| `loggers`          | 显示和修改应用程序中日志的配置。                             |
| `liquibase`        | 显示已应用的所有Liquibase数据库迁移。需要一个或多个`Liquibase`组件。 |
| `metrics`          | 显示当前应用程序的“指标”信息。                               |
| `mappings`         | 显示所有`@RequestMapping`路径列表。                          |
| `scheduledtasks`   | 显示应用程序中的计划任务。                                   |
| `sessions`         | 允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序。 |
| `shutdown`         | 使应用程序正常关闭。默认禁用。                               |
| `startup`          | 显示由`ApplicationStartup`收集的启动步骤数据。需要使用`SpringApplication`进行配置`BufferingApplicationStartup`。 |
| `threaddump`       | 执行线程转储。                                               |

WEB应用专用

| `heapdump`   | 返回`hprof`堆转储文件。                                      |
| ------------ | ------------------------------------------------------------ |
| `jolokia`    | 通过HTTP暴露JMX bean（需要引入Jolokia，不适用于WebFlux）。需要引入依赖`jolokia-core`。 |
| `logfile`    | 返回日志文件的内容（如果已设置`logging.file.name`或`logging.file.path`属性）。支持使用HTTP`Range`标头来检索部分日志文件的内容。 |
| `prometheus` | 以Prometheus服务器可以抓取的格式公开指标。需要依赖`micrometer-registry-prometheus`。 |

最常用的Endpoint

- **Health：监控状况**

  健康检查端点，我们一般用于在云平台，平台会定时的检查应用的健康状况，我们就需要Health Endpoint可以为平台返回当前应用的一系列组件健康状况的集合。

- **Metrics：运行时指标**

  提供详细的、层级的、空间指标信息，这些信息可以被pull（主动推送）或者push（被动获取）方式得到；

- **Loggers：日志记录**

  

##### 定制Endpoint

1. 定制 Health 信息

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}

构建Health
Health build = Health.down()
                .withDetail("msg", "error service")
                .withDetail("code", "500")
                .withException(new RuntimeException())
                .build();
```

另外一种写法：

```java
@Component
public class MyComHealthIndicator extends AbstractHealthIndicator {

    /**
     * 真实的检查方法
     * @param builder
     * @throws Exception
     */
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        //mongodb。  获取连接进行测试
        Map<String,Object> map = new HashMap<>();
        // 检查完成
        if(1 == 2){
//            builder.up(); //健康
            builder.status(Status.UP);
            map.put("count",1);
            map.put("ms",100);
        }else {
//            builder.down();
            builder.status(Status.OUT_OF_SERVICE);
            map.put("err","连接超时");
            map.put("ms",3000);
        }


        builder.withDetail("code",100)
                .withDetails(map);

    }
}
```

2. 定制info信息

   1. 配置文件

   ```xml
   info:
     appName: boot-admin
     version: 2.0.1
     mavenProjectName: @project.artifactId@  #使用@@可以获取maven的pom文件值
     mavenProjectVersion: @project.version@
   ```

   2. 编写InfoContributor

   ```java
   import java.util.Collections;
   
   import org.springframework.boot.actuate.info.Info;
   import org.springframework.boot.actuate.info.InfoContributor;
   import org.springframework.stereotype.Component;
   
   @Component
   public class ExampleInfoContributor implements InfoContributor {
   
       @Override
       public void contribute(Info.Builder builder) {
           builder.withDetail("example",
                   Collections.singletonMap("key", "value"));
       }
   
   }
   ```

3. 定制Metrics信息

   ```java
   class MyService{
       Counter counter;
       //需要在service方法里注册MeterRegistry
       public MyService(MeterRegistry meterRegistry){
            counter = meterRegistry.counter("myservice.method.running.counter");
       }
   
       public void hello() {
           //每次调用hello就加1
           counter.increment();
       }
   }
   
   
   //也可以使用下面的方式
   @Bean
   MeterBinder queueSize(Queue queue) {
       return (registry) -> Gauge.builder("queueSize", queue::size).register(registry);
   }
   ```

4. 定制Endpoint

   ```java
   @Component
   @Endpoint(id = "container")
   public class DockerEndpoint {
   
   
       @ReadOperation
       public Map getDockerInfo(){
           return Collections.singletonMap("info","docker started...");
       }
   
       @WriteOperation
       private void restartDocker(){
           System.out.println("docker restarted....");
       }
   
   }
   ```



### 原理解析

#### Profile 功能

- 默认配置文件  application.yaml；任何时候都会加载
- 指定环境配置文件  application-{env}.yaml

- 激活指定环境

- - 配置文件激活
  - 命令行激活：java -jar xxx.jar --**spring.profiles.active=prod  --person.name=haha**

- - - **修改配置文件的任意值，命令行优先**

- 默认配置与环境配置同时生效
- 同名配置项，profile配置优先

2.@Profile条件装配功能

​	

```Java
@Configuration(proxyBeanMethods = false)
@Profile("production") //只在production环境激活,可以放在方法上
public class ProductionConfiguration {

    // ...

}
```

3.profile分组

```
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq

使用：--spring.profiles.active=production  激活
```

这个时候proddb的参数会优先激活，prodmq的参数会补充proddb的参数。



#### 外部化配置

1、外部配置源

常用：**Java属性文件**、**YAML文件**、**环境变量**、**命令行参数**；

2、配置文件查找位置

(1) classpath 根路径

(2) classpath 根路径下config目录

(3) jar包当前目录

(4) jar包当前目录的config目录

(5) /config子目录的直接子目录

3、配置文件加载顺序：

1. 　当前jar包内部的application.properties和application.yml
2. 　当前jar包内部的application-{profile}.properties 和 application-{profile}.yml

1. 　引用的外部jar包的application.properties和application.yml
2. 　引用的外部jar包的application-{profile}.properties 和 application-{profile}.yml

4、指定环境优先，外部优先，后面的可以覆盖前面的同名配置项



#### 自定义Starters

1、starter启动原理

- starter-pom引入 autoconfigurer 包

![img](https://cdn.nlark.com/yuque/0/2020/png/1354552/1606995919308-b2c7ccaa-e720-4cc5-9801-2e170b3102e1.png)

- autoconfigure包中配置使用 **META-INF/spring.factories** 中 **EnableAutoConfiguration 的值，使得项目启动加载指定的自动配置类**

  ![image-20210825160136656](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210825160136656.png)

- **编写自动配置类 xxxAutoConfiguration -> xxxxProperties**

- - **@Configuration**
  - **@Conditional**

- - **@EnableConfigurationProperties**
  - **@Bean**

- - ......

**引入starter** **--- xxxAutoConfiguration --- 容器中放入组件 ---- 绑定xxxProperties ----** **配置项**

2、自定义starter

**atguigu-hello-spring-boot-starter（启动器）**

**atguigu-hello-spring-boot-starter-autoconfigure（自动配置包**)



AutoConfiguration

```Java
@Configuration
@EnableConfigurationProperties(HelloProperties.class)  //默认HelloProperties放在容器中
public class HelloServiceAutoConfiguration{

    @ConditionalOnMissingBean(HelloService.class)
    @Bean
    public HelloService helloService(){
        HelloService helloService = new HelloService();
        return helloService;
    }
}
```

HelloProperties

```Java
//绑定了配置文件里的信息
@ConfigurationProperties("atguigu.hello")
public class HelloProperties {

    private String prefix;
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

HelloService

```Java
public class HelloService {

    @Autowired
    HelloProperties helloProperties;

    public String sayHello(String userName){
        return helloProperties.getPrefix() + "："+userName+"》"+helloProperties.getSuffix();
    }
}
```

把文件打包，就可以在其他项目中调用自动装配了

![image-20210825161600546](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210825161600546.png)



#### SpringBoot底层原理

1. SpringBoot启动过程

   - 创建 **SpringApplication**

   - - 从spring.caftories中找一些信息保存起来，保存的信息是未来需要运行的服务，监听器，初始化器的信息。
     - 判定当前应用的类型: 使用ClassUtils判断，是web程序，运行Servlet

   - - **bootstrappers****：初始启动引导器（**List<Bootstrapper>**）：去spring.factories文件中找** org.springframework.boot.**Bootstrapper**
     - 找 **ApplicationContextInitializer**；去spring.factories找 **ApplicationContextInitializer**

   - - - List<ApplicationContextInitializer<?>> **initializers**

   - - **找** **ApplicationListener  ；应用监听器。**去**spring.factories找 **ApplicationListener

   - - - List<ApplicationListener<?>> **listeners**

   - 运行 **SpringApplication**

     //省略，后期自行总结

#### 自定义事件监听组件

//省略

