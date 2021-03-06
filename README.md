参考：  
https://www.rabbitmq.com/documentation.html  
[服务端手册](https://www.rabbitmq.com/admin-guide.html)  
[客户端手册](https://www.rabbitmq.com/clients.html)  
[Java API Reference](https://rabbitmq.github.io/rabbitmq-java-client/api/current/)  

目录  
[TOC]  

## 消息队列简介

选择衡量指标：  
1）服务性能  
2）数据存储  
3）集群架构  

几种常见消息队列的对比：    
+ ActiveMQ  
功能强劲但是并发性能不够好，不适用于高并发的复杂的项目。  
架构模式：

+ Kafka  
刚开始是为了收集和传输日志，追求高吞吐量（性能很高）；但是缺点是不支持事务，不会对消息的重复、丢失、错误进行严格要求。  
架构模式：
 
+ RocketMQ  
纯Java开发，起源于Kafka, 针对Kafka的缺陷（不支持事务，传输不可靠）做了优化；具有高吞吐、高可用，适合大规模分布式系统应用的特点。
但是是收费的。

+ RabbitMQ  
性能不比Kafka，但是其他方面很好，高可用，稳定，数据可靠。  
RabbitMQ高可用负载均衡集群架构：  
 
选择RabbitMQ的原因：    
1）开源、性能优秀、稳定性保障  
2）提供可靠性消息投递模式、返回模式  
3）与SpringAMQP完美整合，API丰富  
4）集群模式丰富，表达式配置，HA模式，镜像队列模型  
5）保证数据不丢失的前提下做到高可靠性、可用性   

## RabbitMQ安装与使用

#### Window
https://www.rabbitmq.com/install-windows.html
1. 安装Erlang
    去官网下载安装即可，然后配置环境变量将安装目录添加到Path。  
    测试Erlang是否安装配置成功：
    ```
    > erl   # 正常的话会输出版本号
    ```
2. 安装RabbitMQ Server
    默认安装会将RabbitMQ安装为后台服务，开机自动启动。  
    如果服务正常启动可以看到 任务管理器 > 服务 > RabbitMQ

    进入安装目录的sbin, 可通过命令行执行如下操作：
    ```
    rabbitmq-service start    # 启动服务
    rabbitmq-service stop     # 停止服务
    rabbitmq-service remove   # 移除服务
    rabbitmq-service install  # 安装为服务
    ```

    开启及关闭管理拓展插件
    ```
    rabbitmq-plugins enable rabbitmq_management
    rabbitmq-plugins disable rabbitmq_management
    rabbitmq-plugins list           #显示可选插件
    ```
    
    新建用户及设置用户权限  
    RabbitMQ有五种用户类型：超级管理员, 监控者, 策略制定者, 普通管理者以及其他；  
    默认创建了guest用户，密码guest；  
    默认有一个"/"的虚拟主机；  
    web管理页面：http://localhost:15672/#
    ```
    # rabbitmqctl 更多操作：https://www.rabbitmq.com/man/rabbitmqctl.8.html
    rabbitmqctl add_user <username> <password>
    rabbitmqctl set_user_tags <username> administrator  # 设置用户为管理员
    rabbitmqctl add_vhost <vhostname>   #添加虚拟主机
    rabbitmqctl set_permissions -p <vhostname> <username> .* .* .*
    rabbitmqctl delete_user <username>
    rabbitmqctl change_password <username> <new_password>
    rabbitmqctl list_users
    ```
3. 自定义RabbitMQ环境变量
    有些时候可能需要自定义（或者说客制化）环境变量，有两种方法：  
    - Start > Settings > Control Panel > System > Advanced > Environment Variables 中创建和编辑环境变量的名称和值；  
    - 创建和编辑 rabbitmq-env-conf.bat 用于定义环境变量，这个文件的路径地址由 RABBITMQ_CONF_ENV_FILE 环境变量指定。   
    注意：环境改变后，需要重新安装服务。

    Rabbit环境变量配置项：  
    ```
    RABBITMQ_NODE_IP_ADDRESS  
    RABBITMQ_NODE_PORT  
    RABBITMQ_DIST_PORT  
    RABBITMQ_NODENAME  
    RABBITMQ_CONFIG_FILE  
    RABBITMQ_ADVANCED_CONFIG_FILE  
    RABBITMQ_CONF_ENV_FILE  
    RABBITMQ_USE_LONGNAME  
    RABBITMQ_SERVICENAME  
    RABBITMQ_CONSOLE_LOG  
    RABBITMQ_CTL_ERL_ARGS  
    RABBITMQ_SERVER_ERL_ARGS
    RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS
    RABBITMQ_SERVER_START_ARGS  
    ...  
    ```
    详细参考：  
    https://www.rabbitmq.com/configure.html#configuration-file  

#### Mac
```
$ brew udpate
$ brew install rabbitmq
```
安装之后默认会将rabbitmq的命令行工具放在 /usr/local/sbin


#### Docker(推荐)
https://hub.docker.com/_/rabbitmq


## RabbitMQ 概念基础

RabbitMQ 可用于数据投递，非阻塞操作或推送通知。以及实现发布／订阅，异步处理，或者工作队列。所有这些都属于消息系统的模式。

RabbitMQ可靠性高，拥有灵活的路由；支持同局域网的集群和聚合；在集群中队列可以镜像到多个服务器实现高可用；
支持多种通信协议，提供了可视化管理工具，异常行为追踪系统，还附带有各种插件可对功能进行拓展。

#### AMQP通信协议  
AMQP(Advanced Message Queueing Protocol)协议模型：  
![](https://img-blog.csdn.net/20180226153824467?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE2MzQ0NjE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

核心概念：
+ Server  

+ Connection  
    相关类：  
    ConnectionFactory:连接工厂  
    使用 new ConnectionFactory()初始化的默认配置：  
    ```
    connectionFactory = {ConnectionFactory@791} 
     virtualHost = "/"
     host = "localhost"
     port = -1
     requestedChannelMax = 2047
     requestedFrameMax = 0
     requestedHeartbeat = 60
     connectionTimeout = 60000
     handshakeTimeout = 10000
     shutdownTimeout = 10000
     _clientProperties = {HashMap@794}  size = 6
      0 = {HashMap$Node@804} "product" -> "RabbitMQ"
      1 = {HashMap$Node@805} "copyright" -> "Copyright (c) 2007-2018 Pivotal Software, Inc."
      2 = {HashMap$Node@806} "capabilities" -> " size = 6"
      3 = {HashMap$Node@807} "information" -> "Licensed under the MPL. See http://www.rabbitmq.com/"
      4 = {HashMap$Node@808} "version" -> "5.5.0"
      5 = {HashMap$Node@809} "platform" -> "Java"
     socketFactory = null
     saslConfig = {DefaultSaslConfig@795} 
      mechanism = "PLAIN"
     sharedExecutor = null
     threadFactory = {Executors$DefaultThreadFactory@796} 
      group = {ThreadGroup@789} "java.lang.ThreadGroup[name=main,maxpri=10]"
      threadNumber = {AtomicInteger@828} "1"
      namePrefix = "pool-1-thread-"
     shutdownExecutor = null
     heartbeatExecutor = null
     socketConf = {SocketConfigurators$lambda@797} 
     exceptionHandler = {DefaultExceptionHandler@798} 
     credentialsProvider = {DefaultCredentialsProvider@799} 
      username = "guest"
      password = "guest"
     automaticRecovery = true
     topologyRecovery = true
     topologyRecoveryExecutor = null
     networkRecoveryInterval = 5000
     recoveryDelayHandler = null
     metricsCollector = null
     nio = false
     frameHandlerFactory = null
     nioParams = {NioParams@800} 
      readByteBufferSize = 32768
      writeByteBufferSize = 32768
      nbIoThreads = 1
      writeEnqueuingTimeoutInMs = 10000
      writeQueueCapacity = 10000
      nioExecutor = null
      threadFactory = null
      socketChannelConfigurator = {SocketChannelConfigurators$lambda@836} 
      sslEngineConfigurator = {NioParams$lambda@837} 
      connectionShutdownExecutor = null
      byteBufferFactory = {DefaultByteBufferFactory@838} 
       allocator = {DefaultByteBufferFactory$lambda@840} 
      writeQueueFactory = {NioParams$lambda@839} 
     sslContextFactory = null
     channelRpcTimeout = 600000
     channelShouldCheckRpcResponseType = false
     errorOnWriteListener = null
     workPoolTimeout = -1
     topologyRecoveryFilter = null
     connectionRecoveryTriggeringCondition = null
     topologyRecoveryRetryHandler = null
     trafficListener = {TrafficListener$1@801} 
    ```
    ```
    params = {ConnectionParams@816} 
     credentialsProvider = {DefaultCredentialsProvider@798} 
     consumerWorkServiceExecutor = null
     heartbeatExecutor = null
     shutdownExecutor = null
     virtualHost = "/"
     clientProperties = {HashMap@793}  size = 6
     requestedFrameMax = 0
     requestedChannelMax = 2047
     requestedHeartbeat = 60
     handshakeTimeout = 10000
     shutdownTimeout = 10000
     saslConfig = {DefaultSaslConfig@794} 
     networkRecoveryInterval = 5000
     recoveryDelayHandler = null
     topologyRecovery = true
     topologyRecoveryExecutor = null
     channelRpcTimeout = 600000
     channelShouldCheckRpcResponseType = false
     errorOnWriteListener = null
     workPoolTimeout = -1
     topologyRecoveryFilter = null
     connectionRecoveryTriggeringCondition = null
     topologyRecoveryRetryHandler = null
     exceptionHandler = {DefaultExceptionHandler@797} 
     threadFactory = {Executors$DefaultThreadFactory@795} 
     trafficListener = {TrafficListener$1@800} 
    ```
    Connection：通过连接工厂创建的连接  
      
    
    
+ Channel  
    Channel： 通过连接创建数据通信信道，用于发送和接收消息；  
         
+ Message  
    传输的数据，由Properties和Payload（Body）组成。  
    常用的属性：  
    delivery mode  
    headers（添加自定义属性）  
    content_type  
    content_encoding  
    priority  
    correlation_id  
    reply_to   
    expiration  
    message_id  
    timestamp  
    type  
    user_id  
    app_id  
    cluster_id  
    
+ Virtual Host（在Broker上建立的虚拟机） 
 
+ Exchange(只是一张映射表)  

    创建之后将会一直存在，除非手动删除或重启rabbitmq-server

    功能：  
        交换机用于接收消息，并根据路由键(Routing Key)转发消息到绑定的队列；  
        交换机可能从多个client接收消息，并转发给多个队列，具体由交换机类型决定； 
         
    类型：  
        Direct：此模式可以使用RabbitMQ自带的Exchange,不需要绑定操作，直接通过RouteKey完全匹配队列传递；  
        Topic：此模式可以进行模糊匹配（#匹配0或多个词，*匹配一个词）  
        fanout：（广播）不通过路由键过滤消息(性能最好)，直接将队列绑定到交换机上，只要有消息发送到交换机都会转发到与改交换机绑定的队列；    
        headers：经过消息头路由，不常用。  
        ...  
    
    参数：  
    Durability(持久化)：    是否需要持久化；  
    Auto Delete：    交换机上无绑定队列时，自动删除此交换机；  
    Internal：   当前Exchange是否用于RabbitMQ内部使用(拓展插件，拓展参数等)，默认为false。  
        
+ Binding  

+ Routing Key   
    路由键，相当于过滤器，交换机与队列即使是绑定的如果路由键不同也不会接收。    

+ Queue  
    Queue：具体的消息存储队列；  

+ Producer & Consumer  
    生产和消费者； 

实际应用中最复杂的通信流程结构图：  


#### Rabbit核心原理与生产问题

RabbitMQ是一个消息代理，核心功能就是接收和发送消息。

+ RabbitMQ架构模型

+ 高性能的原因

+ RabbitMQ消息流转流程

+ 消息生产与消费  

#### 常用的功能

+ 工作队列

+ 发布/订阅

+ 路由（有选择地订阅）

+ RPC

    调用方和被调用方互为消息的生产者和消费者。

+ Confirm确认消息，Return返回消息（Producer到Broker范畴）  

    保证消息可靠性传递的基础，Producer除了投递消息，还要创建一个ConfirmListener用于监听
    MQ Broker返回消息确认应答（Producer和Broker网络连通的话就会返回Ack，并不是说要将消息成功存入队列才返回Ack）。  
    Return Listener 用于处理路由不可达的消息（即因为 Exchange不存在 或指定的 RouteingKey 路由不到
    而导致的不知道消息应该传送到哪里）
    
    Confirm确认消息：  
    1）channel 上开启确认模式 channel.confirmSelect()  
    2）在channel上添加监听： addConfirmListener， 监听成功和失败的返回结果，
    根据具体的结果对消息进行重新发送或记录日志等处理。
    
    Return返回消息：  
    1）配置 Mandatory 属性为true，使MQ Broker监听器会接收路由不可达的消息并返回给Producer，
    然后由 Producer ReturnListener 接收；  
    此属性为 false 的话，broker端会自动删除该消息。  
    
    Question：  
    1）如果发送一条消息，有多个队列接收呢（topic/fanout），如果确保每个队列都成功接收？  
    
+ 消费端ACK与NACK（Broker到Consumer范畴）
    ```
    channel.basicConsume() autoAck属性控制Producer 是否自动发送消费成功或失败的确认消息；
    //手动发送消息消费失败确认信息
    channel.basicNack(envelope.getDeliveryTag(),
                                false,      //multiple true to reject all messages up to and including
                                               //the supplied delivery tag; false to reject just the supplied
                                               //delivery tag 拒绝指定了 deliveryTag 的所有未确认的消息
                                true);      //重新入队列，实际使用中很少使用
    //手动发送消息消费成功确认信息
    channel.basicAck(envelope.getDeliveryTag(), false);                       
    ```
    
+ 自定义消费者

+ TTL（Time to Live）

    消息生存时间，有针对消息本身的生存时间和针对队列消息生存时间，
    针对消息本身的生存时间通过配置消息的属性AMQP.BasicProperties设置，
    针对队列的消息生存时间（消息入列之后算起）通过（API ？）设置。  
    
    可以通过指定消息或队列的TTL实现消息自动删除，避免消息堆积。
    ```
    //指定队列的生存时间
    Map<String, Object> args = new HashMap<>();
    args.put("x-message-ttl", 6000);        //6s后队列被删除，消息放到DLX
    channel.queueDeclare("TEST_TTL_QUEUE", false, false, false, args);
    //指定消息的生存时间
    AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
        .expiration("10000")
        .build();
    ```

+ 死信队列（DLX，Dead-Letter-Exchange， 死信交换机）  

    死信：没有消费者消费的消息。  
    死信队列机制： 当一个消息在队列中变成死信之后，它能被重新publish到另一个Exchange,
    这个Exchange就是DLX。  
    
    死信的判断与常见出现情况：  
    1）消息被拒绝(basic.reject/basic.nack)，并且被拒绝回列(requeue=false)  
    2）消息TTL过期  
    3）队列塞满，先进入队列的消息没来的及被消费被后面的顶出，进入死信队列（注意是先进入的进死信队列） 
    ```
    //设置队列长度 x-max-length
    Map<String, Object> args = new HashMap<>();
    args.put("x-max-length", 100);
    channel.queueDeclare("TEST_TTL_QUEUE", false, false, false, args);
    ```
    
    Producer ---> Default Exchange ---> A Queue --(满足上面条件之一)-> DLX_Exchange -> DLX_Queue
    
    可以设置读取死信队列，从死信队列读取消息，可用于记录异常或其他操作。
    
    代码实现：  
    
+ 消息按优先级投递

    ```
    AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
            .priority(5)    //指定消息优先级，默认5，队列属性x-max-priority指定消息最大优先级
            .build();
    chanel.basicPublish("", QUEUE_NAME, properties, msg.getBytes());
    ```
    
+ 消息延迟投递

    两种实现方式：
    1）使用延迟队列插件
    2）使用死信队列TTL投递特性，待消息超时发到死信队列后，消费者从死信队列取消息处理。
    ```
    
    ```

#### RabbitMQ Java API Guide  

+ Exchange 和 Queue 被动声明    

+ 不等待无返回的队列声明  
    效率更高但是操作是不安全的，往往需要配合心跳机制检测失败操作。  
    
+ 删除实体(如Queue)与清除消息  
    直接删除；  
    删除空的实体；   
    删除无Consumer的实体；  
    清除实体数据。  

+ 发布消息  
    消息属性：  
        内容文本类型  
        传输模式  
        优先级  
        用户Id  
        消息头  
        过期时间  

+ Channel并发注意事项（线程安全）  
    https://www.rabbitmq.com/api-guide.html#concurrency  
    根据经验，应该避免多个线程共享同一个Channel，更好的做法是每个线程使用一个 Channel  
    
+ 检索单个消息

+ 处理无法路由的消息  
    设置 mandatory 标志， 并添加 ReturnListener，将消息的信息返回给Producer。
    
+ 关闭协议  
    Client端关闭流程  
    停机情况的信息  
    
+ Channel / Connection isOpen() 方法在开发测试环境中的使用  

+ 高级连接选项  

    - 消费者线程池  
        默认就启用了消费者线程池，支持最大16个线程。  
        可以使用自定义的线程池。    
        ```
        ExecutorService es = Executors.newFixedThreadPool(20);
        Connection conn = factory.newConnection(es);
        ```
        只有在有证据表明消费者回调处理存在严重瓶颈时，才应考虑使用此功能。
        如果没有执行消费者回调，或者很少，则默认分配绰绰有余
        
        使用自定义线程池的注意事项：  
        When the connection is closed a default ExecutorService will be shutdown(), 
        but a user-supplied ExecutorService (like es above) will not be shutdown(). 
        Clients that supply a custom ExecutorService must ensure it is shutdown eventually (by calling its shutdown() method), 
        or else the pool’s threads may prevent JVM termination.
        
        The same executor service may be shared between multiple connections, 
        or serially re-used on re-connection but it cannot be used after it is shutdown().
        
    - 使用 Hosts 列表   
    
    - 使用 AddressResolver 接口实现服务发现  
    
    - 心跳超时  
        * 定制线程工厂  
    
    - 支持Java 非阻塞IO（NIO）

+ 网络异常自动恢复  
    - 连接恢复
        连接恢复触发时机  
    - RecoveryListener  
    
    - 连接中断与恢复对消息发布的影响以及解决  
    
    - 拓扑恢复  
    
    - 异常检测与恢复限制 
    
    - 手动确认与自动恢复  
    
    - Channel生命周期与拓扑恢复  
    
+ 未处理异常  
    
+ 度量和监控  

+ RPC特征（请求/回复）  

+ TLS支持
        
        
#### RabbitMQ Server

##### RabbitMQ Server 配置
+ 配置  
    RabbitMQ提供几种通用的方式定制Server：配置文件、环境变量、rabbitmqctl、rabbitmq-plugins、
    运行时参数和策略。
    
    配置文件路径：  
    Debian - /etc/rabbitmq/
    Mac OSX (Homebrew) - ${install_prefix}/etc/rabbitmq/, the Homebrew prefix is usually /usr/local  
    Windows - %APPDATA%\RabbitMQ\  
    ```
    $ echo $APPDATA
    C:\Users\Lee\AppData\Roaming
    ```
    核心配置项：
    https://www.rabbitmq.com/configure.html#config-items
    
    
+ 文件和目录位置
+ 日志
+ 策略和运行时参数
+ 客户端连接心跳
+ 内部节点连接心跳
+ 队列和消息生存周期拓展

##### RabbitMQ 命令行工具
+ rabbitmqctl  
    https://www.rabbitmq.com/man/rabbitmqctl.8.html
    ```
    rabbitmqctl list_queues
    rabbitmqctl list_vhosts
    rabbitmqctl status
    ```    
+ rabbitmq-plugins
+ rabbitmqadmin

##### 访问控制（认证与授权）

##### 网络和TLS

##### 监控、审计、故障排除

##### 分布式RabbitMQ

##### Guidelines（）
+ 生产清单
+ 备份和恢复
+ 可靠的消息传递
+ 升级
+ Blue-green deployment-based upgrade

##### 消息存储和资源管理
+ 内存使用
+ 内存管理
+ 资源警报
+ 空磁盘空间报警
+ 流控制
+ 消息存储配置
+ 队列和消息生命周期拓展
+ 队列长度限制
+ 懒队列

##### STOMP, MQTT, WebSockets
[一文读懂MQTT协议](https://www.jianshu.com/p/5c42cb0ed1e9)

安装STOMP插件之后就可以使用STOMP协议进行消息的传递；  
MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于发布/订阅（publish/subscribe）模式的“轻量级”通讯协议，该协议构建于TCP/IP协议上，由IBM在1999年发布。
MQTT最大优点在于，可以以极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务。

这些插件提供了实现长连接的方案。

STOMP、MQTT协议的优缺点？怎么选择？

#### RabbitMQ Client

##### 下载和安装

##### API指南

##### API参考手册

##### JMS指南

##### JMS手册

##### 命令行工具

## 常用API方法总结
```
queueDeclare(String queue, 
            boolean durable, 
            boolean exclusive, 
            Map<String, Object> arguments);
queue: 队列名称
durable： 是否持久化, 队列的声明默认是存放到内存中的，如果rabbitmq重启会丢失，如果想重启之后还存在就要使队列持久化，保存到Erlang自带的Mnesia数据库中，当rabbitmq重启之后会读取该数据库
exclusive：是否排外的，有两个作用，一：当连接关闭时connection.close()该队列是否会自动删除；二：该队列是否是私有的private，如果不是排外的，可以使用两个消费者都访问同一个队列，没有任何问题，
如果是排外的，会对当前队列加锁，其他通道channel是不能访问的，如果强制访问会报异常：com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=405, reply-text=RESOURCE_LOCKED - cannot obtain exclusive access to locked queue 'queue_name' in vhost '/', class-id=50, method-id=20)一般等于true的话用于一个队列只能有一个消费者来消费的场景
autoDelete：是否自动删除，当最后一个消费者断开连接之后队列是否自动被删除，可以通过RabbitMQ Management，查看某个队列的消费者数量，当consumers = 0时队列就会自动删除
arguments： 

队列中的消息什么时候会自动被删除？
Message TTL(x-message-ttl)：
    设置队列中的所有消息的生存周期(统一为整个队列的所有消息设置生命周期), 也可以在发布消息的时候单独为某个消息指定剩余生存时间,单位毫秒, 类似于redis中的ttl，生存时间到了，消息会被从队里中删除，注意是消息被删除，而不是队列被删除， 特性Features=TTL, 单独为某条消息设置过期时间AMQP.BasicProperties.Builder properties = new AMQP.BasicProperties().builder().expiration(“6000”); 
    channel.basicPublish(EXCHANGE_NAME, “”, properties.build(), message.getBytes(“UTF-8”));
Auto Expire(x-expires): 
    当队列在指定的时间没有被访问(consume, basicGet, queueDeclare…)就会被删除,Features=Exp
Max Length(x-max-length): 
    限定队列的消息的最大值长度，超过指定长度将会把最早的几条删除掉， 类似于mongodb中的固定集合，例如保存最新的100条消息, Feature=Lim
Max Length Bytes(x-max-length-bytes): 
    限定队列最大占用的空间大小， 一般受限于内存、磁盘的大小, Features=Lim B
Dead letter exchange(x-dead-letter-exchange)： 
    当队列消息长度大于最大长度、或者过期的等，将从队列中删除的消息推送到指定的交换机中去而不是丢弃掉,Features=DLX
Dead letter routing key(x-dead-letter-routing-key)：
    将删除的消息推送到指定交换机的指定路由键的队列中去, Feature=DLK
Maximum priority(x-max-priority)：
    优先级队列，声明队列时先定义最大优先级值(定义最大值一般不要太大)，在发布消息的时候指定该消息的优先级， 优先级更高（数值更大的）的消息先被消费,
Lazy mode(x-queue-mode=lazy)： 
    Lazy Queues: 先将消息保存到磁盘上，不放在内存中，当消费者开始消费的时候才加载到内存中
Master locator(x-queue-master-locator)
```


## RabbitMQ 官方Demo
这部分全参考自官网教程   
https://www.rabbitmq.com/getstarted.html
https://github.com/rabbitmq/rabbitmq-tutorials (github上的源码)

+ Hello World  
    最简单的RabbitMQ消息队列模型是  
    ![](https://www.rabbitmq.com/img/tutorials/python-one.png)  
    一个生产者发送消息给队列，队列将消息再发给一个消费者。  
    多个生产者可以将消息发送给同一个消息队列，多个消费者也可以从同一个消息队列获取消息。  
    
    此Demo演示一个生产者发送一个消息给队列，一个消费者从队列获取消息并将结果打印出来。
    里面只是涉及RabbitMQ API最简单的使用。  
    
    依赖（amqp-client slf4j-api slf4j-simple）
    ```
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>LATEST</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>LATEST</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>LATEST</version>
    </dependency>
    ```
    
    实现步骤：
    1) 通过ConnectionFactory创建新的连接Connection，并为新的连接创建Channel;  
        Connection是socket连接的抽象，负责协议版本协商和认证等等；
    2) 声明使用的队列[是幂等的,其任意多次执行所产生的影响均与一次执行的影响]（包括名称、服务重启队列是否保留、是否是创建专属队列、是否自动删除，队列构造参数）
        若此名称队列已存在则不会新创建。
    3) 发布消息
        可以指定交换机exchange,消息传给哪个队列，消息其他的一些属性（如：路由头部等等）    
    4) 消费者端同上1、2步，创建Channel声明关注的队列  
    5) 创建消费者实例，重写handleDelivery方法处理消息  
        handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body)
        consumerTag: ? 服务器自动生成的消费者的标志
        envelope:   信封很形象，指定一些传输地址信息
        properties: 指定contentHeader（内容类型、编码格式、传输模式、优先级、时间戳、正文大小等等，类似http头）
        body:       正文
    6) 使用创建的消费者实例监听队列
    
    查看创建的消息队列  
    ```
    rabbitmqctl.bat list_queues
    # 结果如下
    # Timeout: 60.0 seconds ...
    # Listing queues for vhost / ...
    # hello   0           # hello是队列名 0表示队列里面的消息条数
    ```
    
+ Work queues    
    一个生产者将消息发给一个queue，（如果有多个消息）再将消息以轮回或公平分发的策略发给多个消费者，
    后面的几个场景也是如此，也可以指定多个消费者。
    ![](https://www.rabbitmq.com/img/tutorials/python-two.png)

    涉及内容：  
    - 消息确认(当消息处理异常时，将消息重新传给另一个消费者进行处理，这个功能是默认打开的)  
        消费者接收处理完消息后会返回确认信息，然后消息才会从队列中删除，如果消费者线程处理异常导致connection和channel关闭，
        RabbitMQ会立即将消息传给其他在线的消费者进行处理。
        消息没有超时的概念，即使一个消息处理花费的时间很长很长，只要连接正常就不会转发。
        ```
        channel.basicAck(envelope.getDeliveryTag(), false); //返回确认信息(multiple为false表示只返回delivery tag)
        channel.basicConsume(TASK_QUEUE_NAME, false, consumer);     //send a proper acknowledgment from the worker
        ```
    - 消息持久性（为确保即使RabbitMQ退出或崩溃后消息仍然不会丢失）  
       ```
       channel.queueDeclare("task_queue", durable, false, false, null);  //第二个参数为true,队列为持久化的队列，否则为非持久化队列    
       
       channel.basicPublish("", "task_queue",
                   MessageProperties.PERSISTENT_TEXT_PLAIN,
                   message.getBytes());
       ```
    - 发布确认(相对于上面两种策略更加强大的一种保护措施)
        参考： https://www.rabbitmq.com/confirms.html
    - 轮回分发  
        轮流将队列中的消息分配给几个消费者（如有四个消息A\B\C\D，两个工作线程W1\W2，则A\C分配给W1,B\D分配给W2）
    - 公平分发  
        不会为消费者分发新的消息直到处理完成前一个消息并返回确认信息。  
        这样还是有遗留问题的，如果所有的消费者（工作线程）都是忙状态，队列可能会被填满，这时就需要添加更多工作者线程或者使用其他策略。
        ```
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);
       ```
       
+ Publish/Subscribe  
    发布/订阅 exchange通过广播的方式将一个消息发送给与exchange绑定的queue，然后再发给一个或多个消费者。 
    
    RabbitMQ更通用的消息模型  
    ![](https://www.rabbitmq.com/img/tutorials/python-three-overall.png)  
    生产者并不将消息直接放入工作队列，而是交给exchange(交换机)，exchange决定将消息放在哪个工作队列或哪些工作队列或是否丢弃；
    这些规则取决于exchange的类型。
    
    - fanout：将收到的消息广播给所有与其相关联的队列。  
    - direct： 与fanout相比添加了通过routingKey选择的功能，只传递给routingKey匹配的队列而非所有关联的队列  
    - topic： 与direct相比更加灵活的模式，通过点分割的单词进行匹配，这样可以匹配一个（这时就像direct一样）或多个特征，direct只有一个特征值；
        而且当一个queue通过#作为bindingKey时，可以接收exchange所有消息，就像fanout一样  
        ```
            * (star) 匹配一个单词（连在一起的一个或多个字符）
            # (hash) 匹配零到多个单词
        ```
    - headers：   
    
    channel.basicPublish("", "hello", null, message.getBytes());这种用法其实是使用了默认的无名exchange。
    消息被路由到routingKey指定名称的队列（如果队列存在的话）。
    
    这个实例是一个简单的日志系统，由两个程序组成，一个发送日志消息，一个接收消息并打印。  
    
    ```
    # 查看exchange的命令
    rabbitmqctl.bat list_exchanges
    # 查看exchange和queue的绑定
    rabbitmqctl.bat list_bindings
    ```
    
+ Routing  
    只订阅某些消息，生产者按routing key发送消息，exchange通过direct模式和routing key匹配接收消息的队列，
    将消息发给匹配的队列。
   
    channel.queueBind(queueName, EXCHANGE_NAME, "black");
    队列绑定到交换机可以额外指定一个routingKey参数（如上），对不同的exchange类型有不同的意义。  
    fanout 类型会直接无视此参数，将消息传递给所有绑定的队列；  
    direct 类型只会将与queueBind中routingKey值匹配的消息传递给绑定的队列。
     
    如此demo所示  
    ![](https://www.rabbitmq.com/img/tutorials/python-four.png)  
    上图对应代码：
    ```
    # Productor
    channel.exchangeDeclare(EXCHANGE_NAME, "direct");
    String[] routingKeys = {"info", "error", "warning"}; 
    for(String routingKey : routingKeys) {
        channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes());
    }
    # Customer
    channel.exchangeDeclare(EXCHANGE_NAME, "direct");
    channel.queueBind(queue1Name, EXCHANGE_NAME, "error");
    for(String severity : argv){
        channel.queueBind(queue2Name, EXCHANGE_NAME, severity);
    }
    ```
   
+ Topics  
    生产者不只按固定的routing key发送消息，而是按字符串正则表达式“匹配”发送，接收端同样如此。
    
    ![](https://www.rabbitmq.com/img/tutorials/python-five.png)
    
+ Headers  
    Headers类型的exchange使用的比较少，它也是忽略routingKey的一种路由方式。是使用Headers来匹配的。
    Headers是一个键值对，可以定义成Hashtable。发送者在发送的时候定义一些键值对，接收者也可以在绑定时候传入一些键值对，
    两者匹配的话，则对应的队列就可以收到消息。
    
    匹配有两种方式all和any。这两种方式是在接收端必须要用键值"x-mactch"来定义。
    all代表exchange消息的header键值对必须包含接收端定义的多个键值对，而any则代表只要包含接收端定义的键值对的任意一个就可以了。
    
    fanout，direct，topic exchange的routingKey都需要要字符串形式的，而headers exchange则没有这个要求，
    因为键值对的值可以是任何类型。
    
+ RPC  
    通过RabbitMQ实现的一个简单的RPC, Client和Server都既是生产者又是消费者。
    
    ![](https://www.rabbitmq.com/img/tutorials/python-six.png)
    

## RabbitMQ 实际应用

+ 异步处理  
    可以异步处理的任务（相互之间不影响无依赖），都可以使用消息队列的工作者线程（Consumer是通过线程池创建的，多个Consumer可能共用一个线程）进行处理。
    
+ 应用解耦  
    两个模块系统（如订单和库存模块）之间需要尽量低耦合，使用消息队列可以使用消息代替接口调用从而降低耦合。

+ 流量削峰  
    抢购活动中一瞬间并发请求很高，很容易导致应用挂掉，所以在服务器收到请求后先写入消息队列（然后从队列一个个地取并处理），
    消息队列写满后其余的用户请求直接抛弃或跳转到错误页面。

## Spring 框架整合 RabbitMQ

#### Spring AMQP 应用集成 RabbitMQ 
SpringAMQP 对AMQP协议接口做了重新封装，使得接口更加易用。  

SpringAMQP 声明Exchange\Queue\Binding是通过使用SpringAMQP对应构造方法构造，然后@Bean注入的方式。  
声明好之后，启动应用，应用会扫描这些Bean及其配置，然后通过调用RabbitMQ的方法进行实际的初始化。  
所以需要在配置文件中提前声明好Exchange\Queue\Binding。

RabbitTemplate（消息模板）  
是与SpringAMQP整合的时候进行消息发送的关键类。  
提供了丰富的消息发送方法，包括可靠性投递消息方法、回调监听消息接口ConfirmCallback，返回值确认接口
ReturnCallback等等。同样需要先注入到Spring容器中，然后直接使用。  

SimpleMessageListenerContainer（简单消息监听容器）  
1）监听队列（多个队列），自动启动，自动声明功能；    
2）支持事务（设置事务特性，事务管理器，事务属性，事务容量[并发]，是否开启事务，回滚消息等）；  
3）设置消费者数量，如最大最小数量，批量消费；
4）设置消息确认和自动确认模式，是否重回队列，异常捕获handler函数；  
5）设置消费者标签生成策略，是否独占模式，消费者属性等；  
6）设置具体的监听器，消息转换器等。
使用 SimpleMessageListenerContainer 可以动态地修改配置。  


#### Spring Boot 应用集成 RabbitMQ

#### Spring Cloud 应用集成 RabbitMQ

## RabbitMQ集群架构

## 互联网大厂SET架构
   
#### SET架构（单元化架构）
针对不同的业务线，分别使用不用的集群。

对于业务体量庞大的单个大型分布式集群，会面临的问题：  
1）容灾问题  
    
2）资源拓展问题  

3）大集群拆分问题  
 
架构演变  
同城双活 -> 两地三中心 -> SET化

SET架构方案目标  
1）业务：解决业务遇到的拓展性和容灾等需求，支持业务的高速发展；  
2）通用性： 架构侧形成统一通用的解决方案，方便各业务线接入使用。   

SET化架构策略示意图  


## 一线大厂MQ基础组件封装和架构设计


