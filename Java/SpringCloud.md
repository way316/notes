

# SpringCloud 学习笔记



## 微服务测试平台搭建

创建两个model，如果需要model A 调用 model B的服务，使用如下方法：

创建RestTemplate Bean.

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}

```

使用restTemplate.postForObject调用其他微服务

```Java
	public static final String PAYMENT_URL= "http://CLOUD-PAYMENT-SER VICE"; //这里是目标服务名称 spring.application.name
											//"http://localhost:8001";
    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/create")
    public CommonResult<Payment> create(Payment payment) {
        return restTemplate.postForObject(PAYMENT_URL+"/payment/create",payment,CommonResult.class);
    }
```



## 1. Eureake

### 1. 介绍

Eureka采用了CS的设计架构，Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka的客户端连接到 Eureka Server并维持心跳连接。

Eureka包含两个组件：Eureka Server和Eureka Client



### 2. 构建步骤 （Eureka 集群）

1. 新建eureka model *2

2. 导入Maven

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
           </dependency>
   ```

3. 修改 yaml

   ```yaml
   eureka:
     instance:
       hostname: eureka7001.com #eureka服务端的实例名称
     client:
       register-with-eureka: false     #false表示不向注册中心注册自己。
       fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
       service-url:
         defaultZone: http://eureka7002.com:7002/eureka/  #指向另外一个Eureka,两个服务互相调用
   ```

4. SpringBoot主程序下面加标签@EnableEurekaServer

5. 回到微服务，为微服务添加Maven包

   ```xml
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>
   ```

6. 将服务发布到集群配置中

   ```yaml
   eureka:
     client:
       #表示是否将自己注册进EurekaServer默认为true。
       register-with-eureka: true
       #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
       fetchRegistry: true
       service-url:
         #defaultZone: http://localhost:7001/eureka
         defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
   ```

6. 在微服务SpringBoot主程序下面加标签@EnableEurekaClient
7. 先启动EurekaServer 再启动服务提供者provider,最后启动消费者



### 3. 负载均衡

在RestTemplate创建bean的configuration里使用@LoadBalanced开启负载均衡

```java
@Configuration
public class ApplicationContextBean
{
    @Bean
    @LoadBalanced //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
    public RestTemplate getRestTemplate()
    {
        return new RestTemplate();
    }
}
```



### 4.actuator微服务信息完善

1. :服务名称修改

   在provider的yaml中添加 eureka.instance.instance-id: 服务名字

2. IP信息提示

   在provider的yaml中添加 eureka.instance.prefer-ip-address: true



### 5.服务发现Discovery

1. 修改Controller

       @Resource
       private DiscoveryClient discoveryClient;

2. 修改主启动类@EnableDiscoveryClient

3. 

### 6. Eureka自我保护

默认启动自我保护，某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存，也就是不会注销任何微服务。好处是不会因为网络波动导致服务因为心跳机制断线。

关闭：

在Eureka端:

```properties
eureka.server.enable-self-preservation:false
#关闭自我保护机制，保证不可用服务被及时踢除
```

在provider端：

```properties
#Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
eureka.instance.lease-renewal-interval-in-seconds=30
#Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
eureka.instance.lease-expiration-duration-in-seconds=90
```



## 2.Zookeeper



## 3.Consul

Consul 使用了go语言书写，提供了微服务系统中的服务治理、配置中心、控制总线等功能。

主要功能：

![image-20210902184010351](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210902184010351.png)



通过以下地址可以访问Consul的首页：http://localhost:8500



操作步骤：

1. 在cmd打开Consul服务器：consul agent -dev

2. 在provider里添加：

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-consul-discovery</artifactId>
           </dependency>
   ```

3. YML里配置

   ```Yaml
   ###consul服务端口号
   server:
     port: 8006
   
   spring:
     application:
       name: consul-provider-payment
   ####consul注册中心地址
     cloud:
       consul:
         host: localhost
         port: 8500
         discovery:
           #hostname: 127.0.0.1
           service-name: ${spring.application.name}
   ```

4. 创建主启动类添加注解

   ```
   @SpringBootApplication
   @EnableDiscoveryClient //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
   ```

5. 运行服务即可



## 三种注册中心的区别

![image-20210902203301344](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210902203301344.png)

![image-20210902203100263](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210902203100263.png)

AP:

当网络分区出现后，为了保证可用性，系统B可以返回旧值，保证系统的可用性。

结论：违背了一致性C的要求，只满足可用性和分区容错，即AP

![image-20210902203400735](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210902203400735.png)

CP:

当网络分区出现后，为了保证一致性，就必须拒接请求，否则无法保证一致性
结论：违背了可用性A的要求，只满足一致性和分区容错，即CP

![image-20210902203410455](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210902203410455.png)



简单来说，如果服务断线，AP返回旧数据，不会报错（防止报错），而CP认为服务出问题了直接报错，不返回数据（在乎数据正确）。



## 4.Ribbon

Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。（ 不需要服务端）

LB负载均衡(Load Balance)是什么
简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）。



Ribbon本地负载均衡客户端 VS Nginx服务端负载均衡区别
 Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是由服务端实现的。Nginx属于集中式负载均衡（在服务的消费方和提供方之间使用独立的LB设施进行负载均衡）

 Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。Ribbon属于属于进程内LB。



### 1.Ribbion 使用

1. 引入库（spring-cloud-starter-netflix-eureka-client自带了spring-cloud-starter-ribbon）

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

2. 使用RestTemplate

   RestTemplate有两种方法，第一种getForObject返回对象

   ![image-20210903112046405](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210903112046405.png)

   第二种方法getForEntity返回ResponseEntity对象,包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等

   ![image-20210903112051285](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210903112051285.png)

3. 在RestTemplate里加上@LoadBalanced

### 2.负载均衡

![image-20210903112407041](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210903112407041.png)



1. 在不同的包添加config文件（不能和springBoot的application在同一个包下，不能被@ComponentScan扫描）

   ```java
   @Configuration
   public class MySelfRule
   {
       @Bean
       public IRule myRule()
       {
           return new RandomRule();//定义为随机
       }
   }
   ```

2. SpringBoot主启动类添加

   ```Java
   @RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration=MySelfRule.class)
   ```

### 3.负载均衡算法原理

1. 获取服务列表，加入list

2. 有几个服务器就设置一个和服务器数量相等的index

3. 连接次数 % index选择服务器。

   

手写实战

1. 创建接口

   ```java
   public interface LoadBalancer
   {
       ServiceInstance instances(List<ServiceInstance> serviceInstances);
   }
   ```

2. 写实现类

   ```java
   @Component
   public class MyLB implements LoadBalancer
   {
       private AtomicInteger atomicInteger = new AtomicInteger(0);
   
       public final int getAndIncrement()
       {
           int current;
           int next;
           do
           {
               current = this.atomicInteger.get();
               next = current >= 2147483647 ? 0 : current + 1;
           } while(!this.atomicInteger.compareAndSet(current, next));
           System.out.println("*****next: "+next);
           return next;
       }
   
   
       @Override
       public ServiceInstance instances(List<ServiceInstance> serviceInstances)
       {
           int index = getAndIncrement() % serviceInstances.size();
           return serviceInstances.get(index);
       }
   }
   ```

3. 在controller里实现负载均衡

   ```java
       @Resource
       private DiscoveryClient discoveryClient;
       @Resource
       private LoadBalancer loadBalancer;
       
       @GetMapping("/consumer/payment/lb")
       public String getPaymentLB()
       {
           List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
   
           if(instances == null || instances.size()<=0) {
               return null;
           }
           ServiceInstance serviceInstance = loadBalancer.instances(instances);
           URI uri = serviceInstance.getUri();
   
           return restTemplate.getForObject(uri+"/payment/lb",String.class);
       }
       
       
   ```

   



## 5.OpenFeign

Feign是一个声明式的Web服务客户端，让编写Web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可

Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用(RestTemplate)。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

Feign整合了Ribbon



### 1. OpenFeign 使用

1. 导入依赖

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
           </dependency>
   ```

2. 设置yml

   ```yaml
   eureka:
     client:
       register-with-eureka: false
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
   ```

3. 主启动类加上@EnableFeignClients

4. 编写Feign接口，需要加上@FeignClient(value = "xxxx")

   ```java
   @Component
   @FeignClient(value = "CLOUD-PAYMENT-SERVICE")
   public interface PaymentFeignService
   {
       //这边相当于直接调用了CLOUD-PAYMENT-SERVICE里的controller的方法
       @GetMapping(value = "/payment/get/{id}")
       CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
   }
   
```java

5. 在controller里调用PaymentFeignService

   @RestController
   public class OrderFeignController
   {
       @Resource
       private PaymentFeignService paymentFeignService;
   
       @GetMapping(value = "/consumer/payment/get/{id}")
       public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id)
       {
           return paymentFeignService.getPaymentById(id);
       }
   }
```

6. Feign自带负载均衡配置项



### 3. OpenFeign超时控制

OpenFeign默认等待1秒钟，超过后报错 

因为OpenFeign底层集合了ribbon所以使用ribbon的配置来修改设置：

```yaml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
#指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
#指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

### 4.OpenFeign日志打印功能

使用方法

1. 配置日志bean

   ```java
   @Configuration
   public class FeignConfig
   {
       @Bean
       Logger.Level feignLoggerLevel()
       {
           return Logger.Level.FULL;
       }
   }
   ```

   

2. YML文件里需要开启日志的Feign客户端

   ```yaml
   logging:
     level:
       # feign日志以什么级别监控哪个接口
       com.wy.springcloud.service.PaymentFeignService: debug
       
       
       日志级别：
       NONE：默认的，不显示任何日志；
    
   	BASIC：仅记录请求方法、URL、响应状态码及执行时间；
    
   	HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；
    
   	FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据。
   ```

   



## 6.Hystrix 断路器

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。当某个服务出现错误时，因为这个服务也调用了其他服务，所以可能会造成微服务占用越来越多的系统次元进而引起系统崩溃。

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。



主要功能：

服务降级: 当达到指定条件时（服务器繁忙）不让客户端等待，返回一个友好提示

服务熔断：类似保险丝，当服务达到最大服务访问后，直接拒绝访问，调用一个友好提示给客户端。

服务限流: 秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行

实时监控

### 1. 服务降级使用方法

#### 方法1：服务端添加

服务端添加：

```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
```

Service添加：在需要降级的方法上添加@HystrixCommand 并且创建服务降级的方法。

```java
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {@HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")})
    public String paymentInfo_TimeOut(Integer id)
    {
        //此处为正常业务逻辑
    }
	
	//此处为服务降级方法
    public String paymentInfo_TimeOutHandler(Integer id){
        return "/(ㄒoㄒ)/调用支付接口超时或异常：\t"+ "\t当前线程池名字" + Thread.currentThread().getName();
    }
```

主启动类添加：@EnableCircuitBreaker



#### 方法2：客户端添加

1.配置文件开启hystrix

```yaml
feign:
  hystrix:
    enabled: true
```

2.主启动类添加：@EnableHystrix

3.业务类与上面的方法完全一样



#### 优化：

1..每个方法上加注解太复杂

在类上加注解：@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")

在需要开启断路器的方法上添加：

@HystrixCommand //加了@DefaultProperties属性注解，并且没有写具体方法名字，就用统一全局的



2.解决业务逻辑和非业务逻辑重叠导致混乱

新建一个类实现feigng功能调用的接口，重写方法

例子：

```
@Component 
public class PaymentFallbackService implements PaymentFeignClientService
{
    @Override
    public String getPaymentInfo(Integer id)
    {
        return "服务调用失败，提示来自：cloud-consumer-feign-order80";
    }
}
 
```

在接口中添加：@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallbackService.class)



### 2.服务熔断的使用方法

熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，
会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。



#### 使用方法：

1.在需要使用熔断器的地方添加一下注解：

```java
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),//是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"), //请求总数阀值：如果窗口期时间内低于这个数那么不打开断路器
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), //快照时间窗
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),//错误百分比阀值：当请求超过请求总数阀值，开始计算失败率，当失败率超过指定数据时，开启熔断
    })
```

详细配置：可以跳过

```java
@HystrixCommand(fallbackMethod = "str_fallbackMethod",
        groupKey = "strGroupCommand",
        commandKey = "strCommand",
        threadPoolKey = "strThreadPool",

        commandProperties = {
                // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离
                @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
                // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）
                @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
                // 配置命令执行的超时时间
                @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
                // 是否启用超时时间
                @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
                // 执行超时的时候是否中断
                @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
                // 执行被取消的时候是否中断
                @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
                // 允许回调方法执行的最大并发数
                @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
                // 服务降级是否启用，是否执行回调函数
                @HystrixProperty(name = "fallback.enabled", value = "true"),
                // 是否启用断路器
                @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
                // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，
                // 如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。
                @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
                // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过
                // circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50,
                // 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。
                @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
                // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，
                // 会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，
                // 如果成功就设置为 "关闭" 状态。
                @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
                // 断路器强制打开
                @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
                // 断路器强制关闭
                @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
                // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
                @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
                // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据
                // 设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。
                // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
                @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
                // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false, 那么所有的概要统计都将返回 -1。
                @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
                // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。
                @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
                // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。
                @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
                // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，
                // 就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，
                // 那么该 “桶” 中只保留 最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
                @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
                // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间。
                @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
                // 是否开启请求缓存
                @HystrixProperty(name = "requestCache.enabled", value = "true"),
                // HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
                @HystrixProperty(name = "requestLog.enabled", value = "true"),
        },
        threadPoolProperties = {
                // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
                @HystrixProperty(name = "coreSize", value = "10"),
                // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，
                // 否则将使用 LinkedBlockingQueue 实现的队列。
                @HystrixProperty(name = "maxQueueSize", value = "-1"),
                // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。
                // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue
                // 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
                @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
        }
)
```



#### 熔断器原理

![image-20210906113650205](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210906113650205.png)



![image-20210906113707956](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210906113707956.png)

![image-20210906113943467](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210906113943467.png)



熔断发生以后调用fallback方法。



### 3. Hystrix工作流程

![image-20210906114328398](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210906114328398.png)

1	创建 HystrixCommand（用在依赖的服务返回单个操作结果的时候） 或 HystrixObserableCommand（用在依赖的服务返回多个操作结果的时候） 对象。

2	命令执行。其中 HystrixComand 实现了下面前两种执行方式；而 HystrixObservableCommand 实现了后两种执行方式：execute()：同步执行，从依赖的服务返回一个单一的结果对象， 或是在发生错误的时候抛出异常。queue()：异步执行， 直接返回 一个Future对象， 其中包含了服务执行结束时要返回的单一结果对象。observe()：返回 Observable 对象，它代表了操作的多个结果，它是一个 Hot Obserable（不论 "事件源" 是否有 "订阅者"，都会在创建后对事件进行发布，所以对于 Hot Observable 的每一个 "订阅者" 都有可能是从 "事件源" 的中途开始的，并可能只是看到了整个操作的局部过程）。toObservable()： 同样会返回 Observable 对象，也代表了操作的多个结果，但它返回的是一个Cold Observable（没有 "订阅者" 的时候并不会发布事件，而是进行等待，直到有 "订阅者" 之后才发布事件，所以对于 Cold Observable 的订阅者，它可以保证从一开始看到整个操作的全部过程）。

3	若当前命令的请求缓存功能是被启用的， 并且该命令缓存命中， 那么缓存的结果会立即以 Observable 对象的形式 返回。

4	检查断路器是否为打开状态。如果断路器是打开的，那么Hystrix不会执行命令，而是转接到 fallback 处理逻辑（第 8 步）；如果断路器是关闭的，检查是否有可用资源来执行命令（第 5 步）。

5	线程池/请求队列/信号量是否占满。如果命令依赖服务的专有线程池和请求队列，或者信号量（不使用线程池的时候）已经被占满， 那么 Hystrix 也不会执行命令， 而是转接到 fallback 处理逻辑（第8步）。

6	Hystrix 会根据我们编写的方法来决定采取什么样的方式去请求依赖服务。HystrixCommand.run() ：返回一个单一的结果，或者抛出异常。HystrixObservableCommand.construct()： 返回一个Observable 对象来发射多个结果，或通过 onError 发送错误通知。

7	Hystrix会将 "成功"、"失败"、"拒绝"、"超时" 等信息报告给断路器， 而断路器会维护一组计数器来统计这些数据。断路器会使用这些统计数据来决定是否要将断路器打开，来对某个依赖服务的请求进行 "熔断/短路"。

8	当命令执行失败的时候， Hystrix 会进入 fallback 尝试回退处理， 我们通常也称该操作为 "服务降级"。而能够引起服务降级处理的情况有下面几种：第4步： 当前命令处于"熔断/短路"状态，断路器是打开的时候。第5步： 当前命令的线程池、 请求队列或 者信号量被占满的时候。第6步：HystrixObservableCommand.construct() 或 HystrixCommand.run() 抛出异常的时候。

9	当Hystrix命令执行成功之后， 它会将处理结果直接返回或是以Observable 的形式返回。



### 4. 服务监控

1. 创建新的module

2. 加入依赖

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
           </dependency>
   ```

3. 启动项加入@EnableHystrixDashboard

4. 被监控的项需要加入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

5. 有个坑，需要在被监视的端的启动项加入：

   ```java
   /**
    *此配置是为了服务监控而配置，与服务容错本身无关，springcloud升级后的坑
    *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
    *只要在自己的项目里配置上下面的servlet就可以了
    */
   @Bean
   public ServletRegistrationBean getServlet() {
       HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
       ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
       registrationBean.setLoadOnStartup(1);
       registrationBean.addUrlMappings("/hystrix.stream");
       registrationBean.setName("HystrixMetricsStreamServlet");
       return registrationBean;
   }
   ```

6. http://localhost:9001/hystrix

7. 填写监控地址http://localhost:8001/hystrix.stream

   ![image-20210906115020702](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210906115020702.png)

8. 查看监控

   ![image-20210906115056950](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210906115056950.png)



## 7. zuul路由网关

Zuul是Netflix出品的一个基于JVM路由和服务端的负载均衡器。

实现了代理+路由+过滤三大功能



zuul已经很少使用了



## 8. Gateway网关

SpringCloud Gateway 使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架,使用非阻塞 API

![image-20210907144819140](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210907144819140.png)

Route(路由):  路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

Predicate(断言):  参考的是Java8的java.util.function.Predicate 开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数)，如果请求与断言相匹配则进行路由

Filter(过滤):  指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。



### 工作流程：

![image-20210907145115531](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210907145115531.png)

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。

Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

Filter在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，
在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。



### 配置方法：

1.添加依赖

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

2.配置YML（无特殊配置，只需要注册进eureka就可以）

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

3.主启动类

```
@SpringBootApplication
@EnableEurekaClient
```

4.新增网关设置

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_routh  #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2     #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
```

此时启动注册中心，驱动服务端，就可以通过调用此类来运行服务端。

添加网关前:http://localhost:8001/payment/get/31

添加网关后:http://localhost:9527/payment/get/31

不使用配置设置：

```java
@Configuration
public class GateWayConfig
{
    /**
     * 配置了一个id为route-name的路由规则，
     * 当访问地址 http://localhost:9527/guonei时会自动转发到地址：http://news.baidu.com/guonei
     * @param builder
     * @return
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder)
    {
        RouteLocatorBuilder.Builder routes = builder.routes();

        routes.route("path_route_atguigu", r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();

        return routes.build();
    }
}
```



### 通过微服务名实现动态路由:

默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

```



### Predicate设置：

- After=2020-02-05T15:10:03.685+08:00[Asia/Shanghai]         # 断言，路径相匹配的进行路由

- Before=2020-02-05T15:10:03.685+08:00[Asia/Shanghai]         # 断言，路径相匹配的进行路由

- Between=2020-02-02T17:45:06.206+08:00[Asia/Shanghai],2020-03-25T18:59:06.206+08:00[Asia/Shanghai]

- Cookie=username,zzyy  访问的时候需要带上cookie curl http://localhost:9527/payment/lb --cookie "username=zzyy"

- Host=**.atguigu.com  http://localhost:9527/payment/lb -H "Host: www.atguigu.com"

- Method=GET

- Path=/payment/lb/**

- Query=username, \d+  # 要有参数名username并且值还要是整数才能路由 http://localhost:9527/payment/lb?username=31

- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式  

   curl http://localhost:9527/payment/lb -H "X-Request-Id:123"

  

### Filter的使用:

1. yaml添加

   ```yaml
   spring:
     cloud:
       gateway:
         routes:
             filters:
               - AddRequestParameter=X-Request-Id,1024 #过滤器工厂会在匹配的请求头加上一对请求头，名称为X-Request-Id值为1024
   ```

2. 自定义(如果不带用户名就无法通过)

   ```java
   @Component //必须加，必须加，必须加
   public class MyLogGateWayFilter implements GlobalFilter,Ordered
   {
       @Override
       public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)
       {
           System.out.println("time:"+new Date()+"\t 执行了自定义的全局过滤器: "+"MyLogGateWayFilter"+"hello");
           String uname = exchange.getRequest().getQueryParams().getFirst("uname");
           if (uname == null) {
               System.out.println("****用户名为null，无法登录");
               exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
               return exchange.getResponse().setComplete();
           }
           return chain.filter(exchange);
       }
   
       @Override
       public int getOrder()
       {
           return 0;
       }
   }
   ```

   http://localhost:9527/payment/lb?uname=z3



## 9. Config

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。



### 使用方法：

1. 创建modul

2. 引入以来

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-config-server</artifactId>
           </dependency>
   ```

3. 配置yaml

   ```yaml
   server:
     port: 3344
   
   spring:
     application:
       name:  cloud-config-center #注册进Eureka服务器的微服务名
     cloud:
       config:
         server:
           git:
             uri: https://github.com/way316/springcloud-config.git #GitHub上面的git仓库名字
           ####搜索目录
             search-paths:
               - springcloud-config
         ####读取分支
         label: main
   
   #服务注册到eureka地址
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

4. 主启动类添加@EnableConfigServer

5. 使用：http://config-3344.com:3344/master/config-dev.yml 从github获取配置

6. 配置读取规则:

   ![image-20210907151421609](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210907151421609.png)

## 

### Config客户端配置:

1. 引入依赖

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-config</artifactId>
           </dependency>
   ```

2. 创建bootstrap.yml （次yml优先级高于application.ymal 所以可以用这个ymal读取github上的配置文件来配置启动）

   ```yaml
   server:
     port: 3355
   
   spring:
     application:
       name: config-client
     cloud:
       #Config客户端配置
       config:
         label: master #分支名称
         name: config #配置文件名称
         profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
         uri: http://localhost:3344 #配置中心地址k
   
   #服务注册到eureka地址
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

   

3. 主启动类

   ```
   @EnableEurekaClient
   @SpringBootApplication
   ```

   

动态刷新问题：

1. 客户端pom引入actuator监控

   ```
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. ymal添加

   ```yaml
   management:
     endpoints:
       web:
         exposure:
           include: "*"
   ```

3. 在controller类添加@RefreshScope

4. 使用   curl -X POST "http://localhost:3355/actuator/refresh" 刷新配置



## 10. Bus 消息总线

实现config的自动刷新，就是config更新之后只需要一条指令就可以更新所有的微服务。

支持Kafka和RabbitMQ

总线： 使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。



使用步骤：

1. 安装RabbitMQ
2. 添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

3. 在配置中心yml中添加：

   ```yaml
   ##rabbitmq相关配置,暴露bus刷新配置的端点 （只需要在config server添加）
   management:
     endpoints: #暴露bus刷新配置的端点
       web:
         exposure:
           include: 'bus-refresh'
   #（每个config client和server都需要添加）     
   rabbitmq:
       host: localhost
       port: 5672
       username: guest
       password: guest
   ```

4. 更新配置

   ```commonlisp
   curl -X POST "http://localhost:3344/actuator/bus-refresh"
   //指定具体某一个实例生效而不是全部 
   http://localhost:配置中心的端口号/actuator/bus-refresh/{destination}
   http://localhost:3344/actuator/bus-refresh/config-client:3355
   ```

   

## 11. Stream 消息驱动

解决各个消息中间件之间的通信问题,通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。

通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离。

![image-20210915200013638](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210915200013638.png)

编码API和常用注解:

![image-20210915200615051](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210915200615051.png)



消息生产者

1. 导入依赖

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
           </dependency>
   ```

2. 添加配置

   ```yaml
   spring:
     application:
       name: cloud-stream-provider
     cloud:
         stream:
           binders: # 在此处配置要绑定的rabbitmq的服务信息；
             defaultRabbit: # 表示定义的名称，用于于binding整合
               type: rabbit # 消息组件类型
               environment: # 设置rabbitmq的相关的环境配置
                 spring:
                   rabbitmq:
                     host: localhost
                     port: 5672
                     username: guest
                     password: guest
           bindings: # 服务的整合处理
             output: # 这个名字是一个通道的名称
               destination: studyExchange # 表示要使用的Exchange名称定义
               content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
               binder: defaultRabbit # 设置要绑定的消息服务的具体设置
               
   eureka:
     client: # 客户端进行Eureka注册的配置
       service-url:
         defaultZone: http://localhost:7001/eureka
     instance:
       lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
       lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
       instance-id: send-8801.com  # 在信息列表时显示主机名称
       prefer-ip-address: true     # 访问的路径变为IP地址
   
   ```

3. 实现类

   ```java
   @EnableBinding(Source.class) // 可以理解为是一个消息的发送管道的定义
   public class MessageProviderImpl implements IMessageProvider
   {
       @Resource
       private MessageChannel output; // 消息的发送管道
   
       @Override
       public String send()
       {
           String serial = UUID.randomUUID().toString();
           // 创建并发送消息
           this.output.send(MessageBuilder.withPayload(serial).build()); 
           System.out.println("***serial: "+serial);
           return serial;
       }
   }
   ```

4. Controller

   ```java
   @RestController
   public class SendMessageController
   {
       @Resource
       private IMessageProvider messageProvider;
   
       @GetMapping(value = "/sendMessage")
       public String sendMessage()
       {
           return messageProvider.send();
       }
   }
   ```



消费者：

1. 依赖

```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    </dependency>
```

2. 配置

   ```yaml
   server:
     port: 8802
   
   spring:
     application:
       name: cloud-stream-consumer
     cloud:
         stream:
           binders: # 在此处配置要绑定的rabbitmq的服务信息；
             defaultRabbit: # 表示定义的名称，用于于binding整合
               type: rabbit # 消息组件类型
               environment: # 设置rabbitmq的相关的环境配置
                 spring:
                   rabbitmq:
                     host: localhost
                     port: 5672
                     username: guest
                     password: guest
           bindings: # 服务的整合处理
             input: # 这个名字是一个通道的名称
               destination: studyExchange # 表示要使用的Exchange名称定义***********************************************************************
               content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
               binder: defaultRabbit # 设置要绑定的消息服务的具体设置
   
   eureka:
     client: # 客户端进行Eureka注册的配置
       service-url:
         defaultZone: http://localhost:7001/eureka
     instance:
       lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
       lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
       instance-id: receive-8802.com  # 在信息列表时显示主机名称 *******
       prefer-ip-address: true     # 访问的路径变为IP地址
    
   ```

   3. 业务类

      ```java
      @Component
      @EnableBinding(Sink.class)
      public class ReceiveMessageListener
      {
          @Value("${server.port}")
          private String serverPort;
      
          @StreamListener(Sink.INPUT)
          public void input(Message<String> message)
          {
              System.out.println("消费者1号，------->接收到的消息：" + message.getPayload()+"\t port: "+serverPort);
          }
      }
      ```



实现分组消费与持久化：

如果不进行分组，默认使用广播形式传递消息。

添加分组：

```yaml
        bindings: # 服务的整合处理
          input: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义***********************************************************************
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置
            group: atguiguA

```

如果是不同的分组，也会对消息进行广播，如果设置两个服务是相同的组就会对消息进行轮播

当使用分组的时候，消息就会进行持久化保存，从而实现持久化操作。如果服务断线，上线之后推送给他的消息会自动推送。



## 12. SpringCloud Sleuth 分布式请求链路跟踪

用来查看微服务之间的关系

使用方法：

1. 下载https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/

2. 运行

3. 查看服务http://localhost:9411/zipkin/

4. 业务提供者添加依赖：

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-zipkin</artifactId>
           </dependency>
   ```

5. 添加设置

   ```yml
   spring:
     zipkin:
       base-url: http://localhost:9411
     sleuth:
       sampler:
         #采样率值介于 0 到 1 之间，1 则表示全部采集
        probability: 1
   ```

6. 调用方也添加依赖

   ```xml
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-zipkin</artifactId>
           </dependency>
   ```

7. 添加设置

   ```yaml
   spring:
       zipkin:
         base-url: http://localhost:9411
       sleuth:
         sampler:
           probability: 1
   ```

   在调用法添加resttemplate调用提供者的function，结束之后可以再网站查看调用情况

   ![image-20210915202415266](C:\Users\Wang\OneDrive\Java全家桶\SpringCloud 学习笔记.assets\image-20210915202415266.png)

