# Nginx

## 基本概念

### 概念

Nginx 是高性能的 HTTP 和反向代理的服务器，处理高并发能力是十分强大的，有报告表明能支持高达 50,000 个并发连接数。

#### 正向代理：

需要在客户端配置代理服务器进行指定网站访问

![image-20211017110712374](C:\Users\Wang\OneDrive\Java全家桶\Nginx.assets\image-20211017110712374.png)

#### 反向代理：

暴露的是代理服务器地址，隐藏了真实服务器的IP地址。

#### 负载均衡：

增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的
情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负
载均衡

![image-20210828161413325](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828161413325.png)



#### 动静分离：

![image-20210828161442457](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828161442457.png)



### 命令

```
1.查看版本号
nginx -v
2. 启动 nginx
nginx
3. 停止 nginx
nginx s stop
4. 重新加载 nginx
nginx s reload
```



### Nginx 配置文件

nginx/conf/nginx.conf

包含三部分内容
1 ）全局块：配置服务器整体运行的配置指令
比如 worker_processes 1; 处理并发数的配置

2 events 块 ：影响 Nginx 服务器与用户的网络连接
比如 worker_connections 1024; 支持的最大连接数为 1024

3 http 块
还包含两部分：
http 全局块
server 块



## 负载均衡实现

在配置文件中修改：

![image-20210828170257691](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828170257691.png)



分配服务器策略：

1、轮询

默认策略，每个请求按时间顺序逐⼀分配到不同的服务器，如果某⼀个服务器下线，能⾃动剔除

```properties
upstream lagouServer{
server 111.229.248.243:8080;
server 111.229.248.243:8082;
}
location /abc {
proxy_pass http://lagouServer/;
}
```

2、weight

weight代表权重，默认每⼀个负载的服务器都为1，权重越⾼那么被分配的请求越多（⽤于服务器性能不均衡的场景）

```properties
upstream lagouServer{
server 111.229.248.243:8080 weight=1;
server 111.229.248.243:8082 weight=2;
}
```

3、ip_hash

 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。 例如：

![image-20210828171846834](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828171846834.png)

4、fair（第三方）	按后端服务器的响应时间来分配请求，响应时间短的优先分配。

![image-20210828171855641](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828171855641.png)



## 动静分离实例

两种实现

1.纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案

2.动态跟静态文件混合在一起发布，通过 nginx 来分开



## 搭建高可用集群

1. 主从模式

   ![image-20210828173523708](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828173523708.png)

   1 ）需要两台 nginx 服务器
   2 ）需要 keepalived
   3 ）需要虚拟 ip

2. 双主模式

   ![image-20210828175604523](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828175604523.png)



## Nginx 原理

![image-20210828181147731](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828181147731.png)

![image-20210828181228123](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20210828181228123.png)

当client发送请求时，worker会争抢这个请求,然后执行反向代理。



Master进程的作用：

主要是管理worker进程，⽐如：

接收外界信号向各worker进程发送信号(./nginx -s reload)
监控worker进程的运⾏状态，当worker进程异常退出后Master进程会⾃动重新启动新的worker进程等

worker进程:

worker进程具体处理⽹络请求。多个worker进程之间是对等的，他们同等竞争来⾃客户端的请
求，各进程互相之间是独⽴的。⼀个请求，只可能在⼀个worker进程中处理，⼀个worker进程，
不可能处理其它进程的请求。worker进程的个数是可以设置的，⼀般设置与机器cpu核数⼀致。





一个 master 和多个 woker 的好处

每个worker进程都是独⽴的，不需要加锁，节省开销
每个worker进程都是独⽴的，互不影响，⼀个异常结束，其他的照样能提供服务
多进程模型为reload热部署机制提供了⽀撑



worker的数量

Nginx同 redis 类似都采用了 io 多路复用机制，每个 worker 都是一个独立的进程，但每个进程里只有一个主线程，通过异步非阻塞的方式来处理请求，每个worker的线程可以把cpu性能发挥到极致，所以worker数量=cpu核心数。



worker_connection

发送请求，占用了 woker 的几个连接数？
2 或者 4 个 （访问静态资源的时候是2个，只有client和worker之间来回，如果需要后端申请数据，那么就需要4个）

nginx 有一 个 master ，有四个 woker ，每个 woker 支持最大的连接数 1024 ，支持的
最大并发数是多少？
普通的静态访问最大并发数是： worker_connections * worker_processes /2
而如果是 HTTP 作 为反向代理来说，最大并发数量应该是 worker_connections *
worker_processes/4

