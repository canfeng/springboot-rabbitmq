## RabbitMQ简单介绍:
RabbitMQ是一个开源的消息代理和队列服务器，用来通过普通协议在完全不同的应用之间传递数据，RabbitMQ是使用Erlang语言来编写的，并且RabbitMQ是基于AMQP协议的。
## 特点：
1. RabbitMQ底层使用Erlang语言编写，传递效率高，延迟低
2. 开源、性能优秀、稳定性较高
3. 与SpringAMQP完美的整合、API丰富
4. 集群模式丰富、表达式配置、HA模式、镜像队列模式
5. 保证数据不丢失的情况下，做到高可用
6. AMQP全称：Advanced Message Queuing Protocol
7. AMQP翻译:高级消息队列协议
## RabbitMQ安装与配置
### 安装RabbitMQ需先安装erlang和socat
1. 安装依赖环境
```
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel
```
2. 下载软件包及其依赖包：
```
wget www.rabbitmq.com/releases/erlang/erlang-18.3-1.el7.centos.x86_64.rpm
wget http://repo.iotti.biz/CentOS/7/x86_64/socat-1.7.3.2-5.el7.lux.x86_64.rpm
wget www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-3.6.5-1.noarch.rpm
```
3. 执行安装
```
依次执行
rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm
rpm -ivh socat-1.7.3.2-5.el7.lux.x86_64.rpm
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm
```
4. 配置
```
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app
```
loopback_users 中的 <<"guest">>,只保留guest
5. 服务启动和停止：
```
启动 rabbitmq-server start &
停止 rabbitmqctl app_stop
```
6. 验证是否启动成功
```
lsof -i:5672
```
7. 插件管理

查看插件列表
```
rabbitmq-plugins list
```
启动management插件
```
rabbitmq-plugins enable rabbitmq_management
```
8. 至此已经安装配置完成，访问管控台查看信息
```
http://YourIp:15672/
```
> 如果无法访问，可能是防火墙的问题，可以关闭防火墙试试

```
service firewalld stop  
```

## AMQP核心概念
- Server：又称Broker，接收客户端的连接，实现AMQP实体服务
- Connection：连接，应用程序与Broker的网络连接
- Channel：网络信道，几乎所有的操作都在Channel中进行，包括定义Queue、定义Exchange、绑定Queue与Exchange、发布消息等。Channel是进行消息读写的通道。客户端可以建立多个Channel，每个Channel代表一个会话任务。
- Message：消息，服务器和应用程序之间传送的数据，由Properties和Body组成。Properties可以对消息进行修饰，比如消息的优先级、延迟等高级特性；Body就是消息体内容。
- Virtual host：虚拟地址，用于进行逻辑隔离，最上层的消息路由。一个Virtual host可以有若干个Exchange和Queue，同一个Virtual host里面不能有相同的Exchange和Queue
- Exchange：交换机，接收消息，根据路由键转发消息到绑定的队列
- Binding：Exchange和Queue之间的虚拟连接，binding中可以包含routing key
- Routing key：一个路由规则，虚拟机可用它来确定如何路由一个特定消息
- Queue：也称为Message Queue，消息队列，保存消息并将它们转发给消费者，多个消费者可以订阅同一个Queue，这时Queue中的消息会被平均分摊给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理。
- Prefetch count：如果有多个消费者同时订阅同一个Queue中的消息，Queue中的消息会被平摊给多个消费者。这时如果每个消息的处理时间不同，就有可能会导致某些消费者一直在忙，而另外一些消费者很快就处理完手头工作并一直空闲的情况。我们可以通过设置prefetchCount来限制Queue每次发送给每个消费者的消息数，比如我们设置prefetchCount=1，则Queue每次给每个消费者发送一条消息；消费者处理完这条消息后Queue会再给该消费者发送一条消息。

## 消息流转流程图
![image](https://github.com/suxiongwei/springboot-rabbitmq/blob/master/springboot-producer/src/main/resources/static/RabbitMQ1.jpg)

## RabbitMQ整合SpringBoot2.x,消息可靠性传递方案100%的实现
### 方案流程图
![image](https://github.com/suxiongwei/springboot-rabbitmq/blob/master/springboot-producer/src/main/resources/static/RabbitMQ2.jpg)
### 代码实现
1. 引入相关依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
2. 配置application.yml
```
## producer配置
spring:
  rabbitmq:
    addresses: 192.168.221.128:5672
    username: guest
    password: guest
    virtual-host: /
  datasource:
    url: jdbc:mysql://localhost:3306/rabbitmq?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull
    username: root
    password: root
    driverClassName: com.mysql.jdbc.Driver
server:
  port: 8001
  servlet:
    context-path: /
```

```
## consumer配置
## springboot整合rabbitmq的基本配置
spring:
  rabbitmq:
    addresses: 192.168.221.128:5672
    username: guest
    password: guest
    virtual-host: /
## 消费端配置
    listener:
      simple:
        concurrency: 5
        acknowledge-mode: manual
        max-concurrency: 10
        prefetch: 1
  datasource:
      url: jdbc:mysql://localhost:3306/rabbitmq?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull
      username: root
      password: root
      driverClassName: com.mysql.jdbc.Driver
server:
  port: 8002
  servlet:
    context-path: /
```
## 完整实例代码[springboot-rabbitmq](https://github.com/suxiongwei/springboot-rabbitmq)

## 数据库文件
```sql
-- ----------------------------
-- Table structure for broker_message_log
-- ----------------------------
DROP TABLE IF EXISTS `broker_message_log`;
CREATE TABLE `broker_message_log` (
  `message_id` varchar(255) NOT NULL COMMENT '消息唯一ID',
  `message` varchar(4000) NOT NULL COMMENT '消息内容',
  `try_count` int(4) DEFAULT '0' COMMENT '重试次数',
  `status` varchar(10) DEFAULT '' COMMENT '消息投递状态 0投递中，1投递成功，2投递失败',
  `next_retry` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' ON UPDATE CURRENT_TIMESTAMP COMMENT '下一次重试时间',
  `create_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' ON UPDATE CURRENT_TIMESTAMP,
  `update_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`message_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for t_order
-- ----------------------------
DROP TABLE IF EXISTS `t_order`;
CREATE TABLE `t_order` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `message_id` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2018091102 DEFAULT CHARSET=utf8;
```

## 演示步骤:
1. run ConsumerApplication 来开启消费者服务
2. run ProducerApplication 来开启生产者服务
3. run SpringbootProducerApplicationTests 中的testSend方法来发送消息进行测试
## 代码实例及学习参考内容来自慕课网课程[RabbitMQ消息中间件极速入门与实战](https://www.imooc.com/learn/1042)


## 其它
## RPC和MQ各自适合的应用场景
RPC比较适合- 客户端调用哪个服务器比较明确 
> 调用需要立即得到返回结果 <br/>
架构简单 在一个由多个微服务构成的大系统中，某些关键服务间的调用应当在较短的时间内返回，而且各个微服务的专业化程度较高，同一个请求的关注者只有一个。这个时候就应该用RPC。 比如在一个ERP系统中，有一个管理仓储的微服务，以及一个负责订单的微服务。新建订单时需要查知当前的存货是否充足，如果不充足就通知用户；提交订单时预订指定数量的货物，如果此时货物不错，也要终止订单的提交，并通知用户。显然在这种场景下是不允许较大的延迟，否则会影响用户体验。所以应该使用RPC，及时返回仓储情况。 

MQ比较适合 
> 消息的发送者和消费者需要解耦的情况  <br/>
发送者并不明确谁是消费者  <br/>
发送者并不关心谁来消费消息  <br/>
各个消费者可以从不同的角度入手处理消息  <br/>
消费者的处理结果也不返回给发送者  <br/>
消息的发送和处理是异步的  <br/>
消息的关注者不止一个 <br/>
在一个由多个微服务构成的大系统中，会有一些非关键服务，用来执行一些不需要立刻得到结果的计算。而且它们的计算结果并不会返回给消息的发送者。这个时候就应该使用MQ。


### [RabbitMQ：消息发送确认与消息接收确认（ACK）](https://www.jianshu.com/p/2c5eebfd0e95)