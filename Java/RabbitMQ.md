# RabbitMQ 基础

## 1.概念

MQ:本质是个队列，FIFO先入先出，只不过队列中存放的内容是message而已，还是一种跨进程的通信机制，用于上下游传递消息



### 为什么要用MQ:

1. 流量消峰:使用消息队列做缓冲，把一秒内下的订单分散成一段时间来处理
2. 应用解耦:不同微服务之间的通信如果使用消息中间件，如果某个服务死机重启，别的服务传来的消息可以存储在消息队列里等待服务重启完毕
3. 异步处理:服务异步调用，解决调用的时候的消息传输问题，不需要主动监听，等待消息队列发消息就可以了。



### 主流MQ：

1. ActiveMQ:

   维护越来越少，高吞吐量场景较少使用

2. Kafka:

   适合产生大量数据的互联网服务的数据收集业务

3. RocketMQ

   可靠性非常高，适合高吞吐场景

4. RabbitMQ

   性能好时效性微秒级，社区活跃度也比较高



### 核心概念

生产者	交换机	队列	消费者

主要工作模式：

![image-20210914104331983](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914104331983.png)



### RabbitMQ 流程图

![image-20210914110212117](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914110212117.png)



Broker：接收和分发消息的应用，RabbitMQ Server就是Message Broker

Virtual host：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的namespace概念。当多个不同的用户使用同一个RabbitMQ server提供的服务时，可以划分出多个vhost，每个用户在自己的 vhost 创建 exchange／queue 等

Connection：publisher／consumer和broker之间的TCP连接

Channel：如果每一次访问 RabbitMQ 都建立一个Connection，在消息量大的时候建立 TCP Connection的开销将是巨大的，效率也较低。Channel是在connection内部建立的逻辑连接，如果应用程序支持多线程，通常每个thread创建单独的channel进行通讯，AMQP method包含了channel id 帮助客户端和message broker 识别 channel，所以channel之间是完全隔离的。Channel作为轻量级的Connection极大减少了操作系统建立TCP connection的开销

Exchange：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)

Queue：消息最终被送到这里等待consumer取走

Binding：exchange和queue之间的虚拟连接，binding中可以包含routing key，Binding信息被保存到exchange中的查询表中，用于message的分发依据



## 2. Hello World

![image-20210914111823923](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914111823923.png)

使用消息队列直接连接两个程序



## 3. Work Queues

![image-20210914111956036](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914111956036.png)

作用：避免立即执行资源密集型任务，而不得不等待它完成。相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。

1. 轮训分发消息

   直接绑定两个消费者到消息队列就会自动进行轮训分发消息

2. 消息应答

   RabbitMQ一旦向消费者传递了一条消息，便立即将该消息标记为删除。为了保证消息在发送过程中不丢失，rabbitmq引入消息应答机制，消息应答就是:消费者在接收到消息并且处理该消息之后，告诉rabbitmq它已经处理了，rabbitmq可以把该消息删除了。

   

   消息应答的方法:

   A.Channel.basicAck(用于肯定确认)
   RabbitMQ已知道该消息并且成功的处理消息，可以将其丢弃了

   B.Channel.basicNack(用于否定确认)

   C.Channel.basicReject(用于否定确认)
   与Channel.basicNack相比少一个参数 不处理该消息了直接拒绝，可以将其丢弃了

   

   ```java
   channel.basicAck(deliveryTag, true);
   //是否启用批量应答
   //启用代表，如果消息队列发送了十个消息，我确认了第十个，那么前十个会被自动确认应答
   ```

   

3. RabbitMQ持久化

   保障当RabbitMQ服务停掉以后消息生产者发送过来的消息不丢失：我们需要将队列和消息都标记为持久化。

   队列持久化：

   

   ![image-20210914122406881](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914122406881.png)

   消息实现持久化:

   ![image-20210914122443526](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914122443526.png)

   依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点

4. 不公平分发

   ![image-20210914124400179](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914124400179.png)

   告诉队列同时能处理多少个消息。

5. 预取值

   就是上面prefetchCount的值，100到300范围内的值通常可提供最佳的吞吐量。 这个数据存储在channel里。



## 4.发布确认

生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID,一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者，这就使得生产者知道消息已经正确到达目的队列了.

confirm模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果RabbitMQ因为自身内部错误导致消息丢失，就会发送一条nack消息，生产者应用程序同样可以在回调方法中处理该nack消息。

开启方法：

![image-20210914124929953](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914124929953.png)

1.单个发布确认

发布一个消息之后被确认发布之后再发布下一条信息:发布速度特别的慢。

```Java
//开启发布确认 
channel.confirmSelect();

//服务端返回false或超时时间内未返回，生产者可以消息重发 
boolean flag = channel.waitForConfirms();
if(flag){ System.out.println("消息发送成功"); }
```

2.批量确认发布

缺点：当发生故障导致发布出现问题时，不知道是哪个消息出现问题了，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息

```java
//开启发布确认 
channel.confirmSelect();
//批量确认消息大小 
int batchSize = 100; 
//未确认消息个数 
int outstandingMessageCount = 0;


outstandingMessageCount++;
if (outstandingMessageCount == batchSize) { 
    channel.waitForConfirms(); 
    outstandingMessageCount = 0;
}
```

3.异步确认发布

利用回调函数来达到消息可靠性

![image-20210914125829281](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914125829281.png)

速度比较：

![image-20210914130011276](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914130011276.png)

## 5.发布/订阅（交换机）

生产者生产的消息从不会直接发送到队列,生产者只能将消息发送到交换机。交换机再把这些内容发送到队列上。

四种类型的交换机：

直接(direct), 主题(topic) ,标题(headers) , 扇出(fanout)

Fanout: 使用fanout类型交换器，routingKey忽略。每个消费者定义生成一个队列并绑定到同一个
Exchange，每个消费者都可以消费到完整的消息。

Direct:根据routingkey来分配数据（可以多个队列有相同的绑定，这样就会在这些队列上进行广播）

![image-20210914150400486](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914150400486.png)

Topics：类似Direct，只不过绑定的不是routingkey的全部名称而是绑定的正则表达式。

![image-20210914151435873](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914151435873.png)



## 6.死信队列

由于特定的原因导致queue中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信



死信的来源：

消息TTL过期
队列达到最大长度(队列满了，无法再添加数据到mq中)
消息被拒绝(basic.reject或basic.nack)并且requeue=false.

图中蓝色部分就是死信交换机和死信队列，用于发送死信。

![image-20210914151850530](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914151850530.png)



## 7.延迟队列

延时队列中的元素是希望在指定时间到了以后或之前取出和处理,所以延迟队列是用来存放需要在指定时间被处理的元素的队列。



使用场景：

1.订单在十分钟之内未支付则自动取消
2.新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。
3.用户注册成功后，如果三天内没有登陆则进行短信提醒。
4.用户发起退款，如果三天内没有得到处理则通知相关运营人员。
5.预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议



两种方法设置TTL：

队列设置TTL

消息设置TTL

如果设置了队列的TTL属性，那么一旦消息过期，就会被队列丢弃(如果配置了死信队列被丢到死信队列中)，而第二种方式，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间。如果将TTL设置为0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。

下图给TLL设计了三个不同的延时队列

10秒延时，40秒延时，无延时，其中无延时队列可以使用消息设置TTL来判断具体延时。

![image-20210914152512391](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914152512391.png)

rabbitmq_delayed_message_exchange插件

因为RabbitMQ设置消息延时无法处理整个队列里的所有消息，所以需要使用插件来实现QC延时队列。

![image-20210914162322890](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914162322890.png)

```java
CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, args);
```

![image-20210914162513422](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914162513422.png)

## 8.发布确认高级

在生产环境中由于一些不明原因，导致rabbitmq重启，在RabbitMQ重启期间生产者消息投递失败，导致消息丢失，需要手动处理和恢复。

spring.rabbitmq.publisher-confirm-type=correlated

NONE
禁用发布确认模式，是默认值

CORRELATED

发布消息成功到交换器后会触发回调方法



SIMPLE

经测试有两种效果，其一效果和CORRELATED值一样会触发回调方法，
其二在发布消息成功后使用rabbitTemplate调用waitForConfirms或waitForConfirmsOrDie方法 等待broker节点返回发送结果，根据返回结果来判定下一步的逻辑，要注意的点是waitForConfirmsOrDie方法如果返回false则会关闭channel，则接下来无法发送消息到broker



使用方法：

建立回调接口。

![image-20210914164446176](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914164446176.png)

绑定

![image-20210914164501731](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914164501731.png)



回退消息:

在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的。通过设置mandatory参数可以在当消息传递过程中不可达目的地时将消息返回给生产者。

1. 回调接口

![image-20210914164950242](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914164950242.png)

2. 绑定

![image-20210914165013193](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914165013193.png)



备份交换机:

当为某一个交换机声明一个对应的备份交换机时，就是为它创建一个备胎，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。



![image-20210914165149416](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914165149416.png)



当mandatory和备份交换机一起开启的时候，备份交换机优先级高。



## 9. RabbitMQ其他知识点

1. 幂等性

   用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。

   MQ消费者的幂等性的解决一般使用全局ID 或者写个唯一标识比如时间戳 或者UUID 或者订单消费者消费MQ中的消息也可利用MQ的该id来判断，或者可按自己的规则生成一个全局唯一id，每次消费消息时用该id先判断该消息是否已消费过。

   

2. 优先级队列

   ![image-20210914173446805](C:\Users\Wang\OneDrive\Java全家桶\RabbitMQ 基础.assets\image-20210914173446805.png)

   

3. 惰性队列

   惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。

   在发送1百万条消息，每条消息大概占1KB的情况下，普通队列占用内存是1.2GB，而惰性队列仅仅占用1.5MB

```java
args.put("x-queue-mode", "lazy");
```



## 10.RabbitMQ集群

镜像队列

如果集群中的一个节点失效了，队列能自动地切换到镜像中的另一个节点上以保证服务的可用性。

Haproxy+Keepalive实现高可用负载均衡

...

Keepalived实现双机(主备)热备...

Federation Exchange...

Federation Queue...

Shovel...
