# RabbitMq学习

## 安装

### 1.安装依赖环境

```shell
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz

```

### 2.安装Erlang

```sh
rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm

```

- 注意gblic版本，如果版本过低，需要升级glibc环境

### 3.安装RabbitMq

```
# 安装
rpm -ivh socat-1.7.3.2-5.el7.lux.x86_64.rpm

# 安装
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm
```

### 4.开启管理界面及配置

```shell
# 开启管理界面
rabbitmq-plugins enable rabbitmq_management
# 修改默认配置信息
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app 
# 比如修改密码、配置等等，例如：loopback_users 中的 <<"guest">>,只保留guest
# 需要修改loopback_users 中的 <<"guest">>,只保留guest，否则无法登陆管理界面

```

### 5.启动mq

```sh
service rabbitmq-server start # 启动服务
service rabbitmq-server stop # 停止服务
service rabbitmq-server restart # 重启服务
```

### 6.设置配置文件

```sh
cd /usr/share/doc/rabbitmq-server-3.6.5/

cp rabbitmq.config.example /etc/rabbitmq/rabbitmq.config
```





## 常见模型

### WorkQueue模型

- 工作队列，可以提高消息处理速度，避免队列消息堆积

![1671526941452](../../1671526941452.jpg)



### 发布订阅模型

- 发布订阅模型运行同一消息发送给多个消费者。实现的方式是通过加入了交换机（Exchange）实现
- 常见交换机类型包括：
  - Fanout：广播，所有队列都会得到该消息。
  - Direct：路由，根据路由规则发送消息到队列。
  - Topic：话题，根据通配符发送消息到队列。
- Exchange只负责消息路由，不负责存储，路由失败则消息丢失。

![1671527481976](../../1671527481976.jpg)

#### Fanout模式

- 该模式下，Exchange将会把接收到的消息路由到每一个绑定的queue

![1671527562355](../../1671527562355.jpg)

#### Direct模式

- 该模式下，Exchange将会根据路由键规则将消息路由到指定的Queue

![1671540052643](../../1671540052643.jpg)



#### Topic模式

- 该模式和Direct模式相似，但是可以根据通配符更加灵活的路由。

![1671540003293](../../1671540003293.jpg)

## Spring Boot 集成RabbitMq

- 导入Maven依赖
- 配置yml属性

```yaml
spring:
  rabbitmq:
    host: 175.178.101.182 #mq地址
    port: 5672 #mq端口号
    virtual-host: / #虚拟主机
    username: guest #用户名
    password: guest #密码
```



### FanOut模式

#### Provider端：

- Provider端发送消息

```java
@RestController
public class SendMessage {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @GetMapping("/send")
    public String sendMessageToConsumer(){
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "test message, hello!";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String,Object> map=new HashMap<>(16);
        map.put("messageId",messageId);
        map.put("messageData",messageData);
        map.put("createTime",createTime);

        //指定绑定的交换机，fanout模式下路由键为null
        rabbitTemplate.convertAndSend("lml.fanout",null,map);
        return "ok";
    }
}
```

#### Consumer端消费

- consumer端配置注入

```java
@Configuration
public class RabbitMqConfig {
    //声明交换机
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("lml.fanout");
    }
    //声明队列1
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanoutQueue1");
    }
    //声明队列2
    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanoutQueue2");
    }
    //绑定队列1至交换机
    @Bean
    public Binding fanoutBindQueue1(){
        return  BindingBuilder.bind(fanoutQueue1()).to(fanoutExchange());
    }
    //绑定队列2至交换机
    @Bean
    public Binding fanoutBindQueue2(){
        return  BindingBuilder.bind(fanoutQueue2()).to(fanoutExchange());
    }
}
```

- 配置消费类

```java
@Component
public class MqConsumer {

    @RabbitListener(queues = "fanoutQueue1")
    @RabbitHandler
    public void accept(Map map){
        System.out.println("fanoutQueue1收到消息  : " + map.toString());
    }

    @RabbitListener(queues = "fanoutQueue2")
    @RabbitHandler
    public void accept1(Map map){
        System.out.println("fanoutQueue2收到消息  : " + map.toString());
    }
}
```

### Direct模式

- 下列代码演示不配置bean，直接在监听器上进行交换机以及队列等属性绑定
- 该模式下通过指定的路由来进行消息发送和接收

#### Provider端：

```java
  @GetMapping("/sendToOrder")
    public String sendMessageToOrder(){
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "test message, 我是订单模块的消息!";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String,Object> map=new HashMap<>(16);
        map.put("messageId",messageId);
        map.put("messageData",messageData);
        map.put("createTime",createTime);

        rabbitTemplate.convertAndSend("lml.direct","order",map);
        return "ok";
    }
    
    @GetMapping("/sendToGoods")
    public String sendMessageToGoods(){
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "test message, 我是商品模块的消息!";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String,Object> map=new HashMap<>(16);
        map.put("messageId",messageId);
        map.put("messageData",messageData);
        map.put("createTime",createTime);
        rabbitTemplate.convertAndSend("lml.direct","goods",map);
        return "ok";
    }
```



#### Consumer端：

```java
    
@RabbitListener(bindings = @QueueBinding(
            value = @Queue("direct.queue1"),//队列名称
            exchange = @Exchange(name = "lml.direct",type = ExchangeTypes.DIRECT),//配置交换机类型和名称。
            key = {"blue","red"} //配置路由key
    ))
    public void directAccept1(Map map){
        System.out.println("directQueue1收到消息"+map.toString());
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("direct.queue2"),
            exchange = @Exchange(name = "lml.direct",type = ExchangeTypes.DIRECT),
            key = {"blue","yellow"}
    ))
    public void directAccept2(Map map){
        System.out.println("directQueue1收到消息"+map.toString());
    }
```



### Topic模式

- 该模式与Direct模式相似，但是可以使用通配符灵活路由。

#### Consumer端

```java
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("topic.queue1"),
            exchange = @Exchange(name = "lml.topic",type = ExchangeTypes.TOPIC),
            key = {"Order.#"}
    ))
    public void topicAccept1(Map map){
        System.out.println("处理当前消息的线程是:"+Thread.currentThread().getName());
        System.out.println("topicQueue1收到消息"+map.toString());
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("topic.queue2"),
            exchange = @Exchange(name = "lml.topic",type = ExchangeTypes.TOPIC),
            key = {"Order.*"}
    ))
    public void topicAccept2(Map map){
        System.out.println("处理当前消息的线程是:"+Thread.currentThread().getName());
        System.out.println("topicQueue2收到消息"+map.toString());
    }
```

#### Procider端

```java
    @GetMapping("/sendToGoodsWithTopic")
    public String sendMessageToOrderWithTopic(){
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "test message, 我是订单模块中秒杀模块的消息!";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String,Object> map=new HashMap<>(16);
        map.put("messageId",messageId);
        map.put("messageData",messageData);
        map.put("createTime",createTime);

        String messageData2 = "test message, 我是订单模块中结算模块的消息!";
        Map<String,Object> map2=new HashMap<>(16);
        map2.put("messageId",messageId);
        map2.put("messageData",messageData2);
        map2.put("createTime",createTime);

        rabbitTemplate.convertAndSend("lml.topic","Order.seckill",map);
        rabbitTemplate.convertAndSend("lml.topic","Order.balance",map2);
        return "ok";
    }
```



### 消息转换器

- SpringBoot集成RabbitMq默认是用JDK进行序列化的，序列化效率较低，可以改用Jackson序列化
- 引入Jackson依赖
- 配置类注入

```java
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
```





## 保证消息可靠性

### 生产者端

- publisher-confirm 发送者确认
  - 消息成功投递到交换机，返回ACK
  - 消息未成功投递到交换机，返回NACK
- publisher-return 发送者回执
  - 消息投递到交换机了，但是没有路由到队列。返回ack及路由失败原因。

#### 代码实例：

##### 1.SpringBoot配置文件配置

```yaml
  rabbitmq:
    host: 175.178.101.182 #mq地址
    port: 5672 #mq端口号
    virtual-host: / #虚拟主机
    username: guest #用户名
    password: guest #密码
    #开启publisher-confirm  simple：同步等待confirm结果。 correlated：异步回调
    publisher-confirm-type: correlated #需定义confirmCallback
    publisher-returns: true #开启回执，也是异步回调。需定义returnCallback
    template:
      mandatory: true #定义路由失败策略，true：调用returnCallback,false:丢弃消息
```

##### 2.配置类重写回调策略。

```java
@Slf4j
@Configuration
public class RabbitMqReturnConfig implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        RabbitTemplate rabbitTemplate = applicationContext.getBean(RabbitTemplate.class);
        rabbitTemplate.setReturnsCallback(returned -> {
            log.error("消息发送失败,响应码:{},失败原因:{},交换机:{},路由key:{},消息:{}",
                    returned.getReplyCode(),returned.getReplyText(),returned.getExchange(),
                    returned.getRoutingKey(),returned.getMessage());
            //TODO:重发消息或记录
        });
    }
}
```

##### 3.生产者测试代码

```java
   @Test
    public void testSendMessage() throws InterruptedException {
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "test message!";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String,Object> map=new HashMap<>(16);
        map.put("messageId",messageId);
        map.put("messageData",messageData);
        map.put("createTime",createTime);
        CorrelationData correlationData = new CorrelationData();
        //异步回调，根据ack判断消息是否成功投递到交换机
        correlationData.getFuture().addCallback(result -> {
            if (result.isAck()){ //true，代表成功投递到交换机
                log.info("消息成功投递到交换机:{}",result); 
            }
            else {
                log.error("消息未投递到交换机:{}",correlationData.getId());
            }
                }
                ,
                ex -> log.error("消息发送失败:{}",ex));
        rabbitTemplate.convertAndSend("lml.direct","s",map,correlationData);
        Thread.sleep(200);
        //这里不sleep Spring测试类关闭资源回调函数执行不成功返回confirm  ack false
    }
```



### 消息持久化

- MQ默认是内存存储消息，开启持久化功能可以确保缓存在MQ中的消息不丢失。
- SpringAMQP默认将开启消息持久化，包括交换机、队列、消息持久化。

#### 手动开启

##### 1.交换机持久化

```java
    @Bean
    public DirectExchange simpleExchange(){
        //name:交换机名称 durable:是否持久化，默认为true，autoDelete:没有队列绑定时是否自动删除。
        return new DirectExchange("lml.simple.direct",true,false);
    }
```

##### 2.队列持久化

```java
    @Bean
    public DirectExchange simpleExchange(){
        //name:交换机名称 durable:是否持久化，默认为true，autoDelete:没有队列绑定时是否自动删除。
        return new DirectExchange("lml.simple.direct",true,false);
    }
```

##### 3.消息持久化

```java
        Message message = MessageBuilder.withBody("hello,i am test message".getBytes())
                .setDeliveryMode(MessageDeliveryMode.PERSISTENT)//persistent 持久的
                .build();
```



### 消费者端

#### 消费者确认机制

- RabbitMQ支持消费者确认机制。消费者处理消息后可以向MQ发送ACK回执，MQ收到ACK回执时才删除消息，SpringAMQP允许配置一下三种模式
  - manual：手动ack,需要在业务代码结束后，调用api发送ack。
  - auto：自动ack,由Spring检测listener代码是否出现异常，没有异常则返回ack，有异常则抛出异常返回nack（本质是使用到了AOP中的环绕通知）。
  - none：关闭ack，MQ会假定消费者获取消息后必定会成功处理，投递成功后消息立即被删除。

```yaml
spring:
  rabbitmq:
    host: 175.178.101.182 #mq地址
    port: 5672 #mq端口号
    virtual-host: / #虚拟主机
    username: guest #用户名
    password: guest #密码
    listener:
      simple:
        prefetch: 1
        acknowledge-mode: auto #设置消息确认模式
```

- 该模式下出现异常，消息会不断requeue，无限循环，将会给mq带来不必要的压力，因此需要设置spring的retry（重试）机制。消费者消费异常时利用本地重试，而不是无限requeue到mq队列。

```yaml
spring:
  rabbitmq:
    host: 175.178.101.182 #mq地址
    port: 5672 #mq端口号
    virtual-host: / #虚拟主机
    username: guest #用户名
    password: guest #密码
    listener:
      simple:
        prefetch: 1
        acknowledge-mode: auto #设置消息确认模式
        retry:
          enabled: true #开启消费者失败重试
          initial-interval: 1000 #初始失败等待时长
          multiplier: 2 #下次失败等待时长，:multiplier * 上次失败时长
          max-attempts: 3 #最大重试次数
          stateless: true #是否为有状态，true:无状态，false:有状态，如果包含事务，设置为false
```

#### 消费者失败消息处理策略

- 在开启重试模式后，重试次数耗尽，如果消息仍然消费失败，则需要有MessageRecoverer接口处理，它包括下面三种实现策略
  - RejectAndDontRequeueRecoverer：重试耗尽后，直接reject，丢弃消息。默认是这种策略。
  - ImmediateRequeueMessageRecoverer：重试耗尽后，返回nack，消息重新入队。
  - RepublishMessageRecoverer：重试耗尽后，将失败消息投递到指定交换机。

##### 测试RepublishMessageRecoverer处理模式

- 首先定义失败消息的交换机，队列以及绑定关系。

```java
  @Bean
    public DirectExchange errorExchange(){
        return new DirectExchange("error.direct",true,false);
    }

    @Bean
    public Queue errorQueue(){
        return new Queue("error.queue1");
    }

    @Bean
    public Binding bindingErrorExchange(){
        return BindingBuilder.bind(errorQueue()).to(errorExchange()).with("error");
    }
```

- 定义RepublishMessageRecoverer

```java
 @Bean
    public MessageRecoverer republishMessageRecoverer(RabbitTemplate rabbitTemplate){
        return new RepublishMessageRecoverer(rabbitTemplate,"error.direct","error");
    }
```



### 死信交换机

- 当一个队列中的消息满足下列情况之一时，可以成为死信
  - 消费者使用baisc.reject或basic.nack声明消息失败，就是不使用RepublishMessageRecoverer模式，且消息的requeue设置为false。
  - 消息是一个过期消息，超时无人消费。
  - 要投递的队列消息堆积满了，最早的消息可能成为死信。
- 如果队列配置了dead-letter-exchange属性，指定了一个交换机，那么该队列中的死信就会投递到交换机，这个交换机成为死信交换机。
