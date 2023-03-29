# RabbitMQ 

# 作用 
- 异步处理，通过消息调用，类似于多线程的Future、CompletableFuture
  - 注册后直接成功返回，发送通知短信，邮件交由MQ执行
- 应用解耦
  - 原来调用的接口需要调用别的系统方法 可以通过MQ进行，省略代码的修改
- 流量控制 
  - 秒杀业务的流量控制，削峰

# 概述 
消息服务的两个重要概念： 
- 消息代理
- 目的地

当消息发送者发送消息后，将由消息代理接管，消息代理保证消息传递到执行的目的地  

消息队列的两种目的地形式  
- 队列 queue 点对点消息通信 
  - 最终只能有一个点接受消息，可能有多个点在等消息
- 主题 topic 发布 publish / 订阅 subscribe 消息通信
  - 多个接收者都能接受到消息，类似广播模式

JMS `Java Message Service`    
基于JVM消息代理的规范,ActiveMQ,HornetMQ是JMS的实现

AMQP `Advanced Message Queuing Protocol`
高级消息队列协议，兼容JMS   
RabbitMQ是AMQP的实现   

# Spring的支持
- spring-jms 提供对JMS的支持
- spring-rabbit提供了对AMQP的支持
- 需要`ConnectionFactory`的实现来连接消息代理
- 提供`JmsTemplate` `RabbitTemplate`来发送消息
- `@JmsListener` `@RabbitListener` 注解在方法上监听消息代理发布的消息  

# Spring Boot自动配置
- JmsAutoConfiguration
- RabbitAutoConfiguration


# 主要概念流程
- `Message`消息由 消息头 + 消息体 而消息头则由一系列的可选属性组成， 这些属性包括 `routing-key` （路由键）、 `priority`（相对于其他消息的优先权）、`delivery-mode`（指出该消息可 能需要持久性存储）等  

- `Publisher`消息的生产者，也是一个向交换器发布消息的客户端应用程序  

- `Exchange` 交换机，用来接收生产者发送的消息，指定消息规则并将这些消息路由给服务器中的队列。    
  - `Exchange`有4种类型：direct(默认)，fanout,topic,和~~headers~~，不同类型的Exchange转发消息的策略有所区别  
  - `direct` 绑定`路由键`，只有对应的路由键能收到,一对一    
  - `fanout` 广播,不处理路由键，所有队列都会收到消息，一对全部    
  - `topic` 订阅发布模式，通配符 `*/#` 可以对应一个或多个队列   

- `Queue`消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走

- `Binding`  绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。   

- `Connection`网络连接

- `Channel`信道，多路复用一个连接中的一条双向数据流通道，信道是建立在真实的TCP连接内的虚拟连接，AMQP命令都是通过信道发送。   
引入信道就是为了复用TCP连接    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/RabbitMQ概念图.png)


# 安装 

使用Docker安装

```

docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
```

虚拟机启动注意事项：  
- docker 启动后不能再操作firewall(关闭或修改配置)
- 要同步服务器时间 
- 注意使用带Management的tag，不然无法访问web管理


访问localhost:15672 登录 账号密码`guest/guest`


# Exchange类型说明

**direct** 根据路由键的名称精确匹配，一个消息只能到一个队列中，称为路由键的完全匹配，单播模式，点对点匹配    

**fanout** 每个发送到交换机的消息，都会被转发到与该交换机绑定的`所有队列`上,很像子网广播，每台子网内的主机都获得了一份复制的消息   

**topic** 发布订阅模式由路由键`routing key`和绑定键`binding key`组成 消息发送靠路由键，绑定键识别路由键，符合规则的可以获取消息   
例：`routingkey` 是 `usr.name` 那么`bindingkey` 是`usr.name`或`usr.#`或者`usr.*`或`#.name`或`*.name`的队列都可以收到消息    
`#`代表一个或多个单词，`*`代表一个单词，二者可以配合同时使用，前后中都可以

# 使用
- 接入 starter-amqp
- 自动配置了 RabbitTemplate 、 AmqpAdmin、 CachinConnectionFactory 、RabbitMessagingTemplate  
- `@ConfigurationProperties(prefix="spring.rabbitmq")`绑定配置
- `@EnableRabbit`
- 监听消息使用 `@RabbitListener`

## 发送、绑定示例
- `AmqpAdmin`主要进行创建队列、交换机，绑定操作，删除。。。  
- `RabbitTemplate` 主要是发送消息
  - 发送对象消息需要实现序列化`Serializable`接口，默认的Converter是`SimpleMessageConverter`
  - 使用`Json格式`需要在容器中注入消息转换器`MessageConverter`，返回`Jackson2JsonMessageConverter`

```java
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MessageConverter.png)

```java
@Slf4j
@SpringBootTest
class GulimallOrderApplicationTests {
    @Autowired
    AmqpAdmin amqpAdmin;
    @Autowired
    RabbitTemplate rabbitTemplate;

    @Test
    void sendMsg(){
        OrderReturnReasonEntity entity = new OrderReturnReasonEntity();
        entity.setId(1L);
        entity.setName("1231");
        entity.setCreateTime(new Date());
        //发送的消息是对象 就会序列化对象
        rabbitTemplate.convertAndSend("hello-java-exchange","hello.java",entity);
    }

    @Test
    void contextLoads() {

        DirectExchange directExchange = new DirectExchange("hello-java-exchange",true,false);

        amqpAdmin.declareExchange(directExchange);

        log.info("exchange {} 创建成功",directExchange.getName());
    }

    @Test
    public void createQueue(){
        Queue queue = new Queue("hello-java-queue",true,false,false);
        amqpAdmin.declareQueue(queue);

        log.info("exchange {} 创建成功",queue.getName());
    }

    @Test
    public void binding(){
        Binding binding = new Binding("hello-java-queue", Binding.DestinationType.QUEUE,
                "hello-java-exchange","hello.java",null);
        amqpAdmin.declareBinding(binding);
    }

}
```


## 接收消息  
- `@RabbitListener`标注在类上或方法上
  - 若标注在类上需要配合`@RabbitHandler`可以根据消息类型进行区分，自动选择合适类型的方法接受消息

```java
    /**
     * @param message 原生消息详细信息 消息头+消息体
     * @param entity 发送的类型消息，自动转化为实体
     * @param channel 传输数据的通道
     */
    @RabbitListener(queues = "hello-java-queue")
    public void receiveMessage(Message message, OrderReturnReasonEntity entity, Channel channel){
        byte[] body = message.getBody();
        MessageProperties messageProperties = message.getMessageProperties();
    }


@Component
@RabbitListener(queues = "consumer_queue")
public class Receiver {

    @RabbitHandler
    public void processMessage1(String message) {
        System.out.println(message);
    }

    @RabbitHandler
    public void processMessage2(byte[] message) {
        System.out.println(new String(message));
    }
    
}
```

## 消息可靠抵达
保证消息不丢失，可靠抵达，可以使用事务消息，但是性能会下降严重   
可靠抵达主要分为两端的处理  

- publisher发送端 `confirmCallback` 确认模式 
  - 只要消息被broker接收到就会执行`confirmCallback`，如果是集群cluster模式,那么需要所有的broker都接收到才会调用confirmCallback
  - 这里只能保证抵达消息服务器，而不保证被准确投递到执行的queue中
- publisher发送端 `returnCallback` 未投递到queue 退回模式
  - 投递失败触发
- consumer接受端 ack机制 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/rabbitmq可靠抵达.png)

```properties
# 开启发送端确认
spring.rabbitmq.publisher-confirms=true
#开启发送端消息抵达队列确认
spring.rabbitmq.publisher-returns=true
#只要抵达队列，以异步发送优先回调 ReturnsCallback
spring.rabbitmq.template.mandatory=true
```

```java
@Configuration
public class MyRabbitConfig {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }

    /**
     * 定制 RabbitTemplate
     * @PostConstruct MyRabbitConfig 对象创建完成后执行这个方法
     * correlationData 当前消息的唯一关联数据 消息唯一id
     * ack 消息是否成功收到
     * cause 失败原因
     */
    @PostConstruct
    public void initRabbitTemplate(){
        //设置确认回调
        /*
        * correlationData 当前消息的唯一关联数据 消息唯一id
         * ack 消息是否成功收到 true |false
         * cause 失败原因
        * */
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            System.out.println("correlationData = " + correlationData);
            System.out.println("ack = " + ack);
            System.out.println("cause = " + cause);
        });

        //设置消息抵达队列的确认回调
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {

            /**
             * 只要消息没有投递给指定队列 就触发
             * @param returned
             *     private final Message message; 投递失败的消息详细信息
             *     private final int replyCode;
             *     private final String replyText; 回复的文本内容
             *     private final String exchange; 交换机
             *     private final String routingKey; 路由键
             */
            @Override
            public void returnedMessage(ReturnedMessage returned) {
                System.out.println("returned = " + returned);

            }
        });
    }

}
```

**消费端确认**   
- 默认是自动确认，只要消息接收到，就会在队列中删除消息
- 若开启手动确认，只有消费者确认后才能从队列中删除  

```properties
# auto 自动 manual 手动ack none  
spring.rabbitmq.listener.simple.acknowledge-model=manual
```

```java
@Service
@RabbitListener(queues = "stock.release.stock.queue")
public class OrderCloseListener {

    @Autowired
    private OrderService orderService;


    @RabbitHandler
    public void handleStockLockedRelease(OrderEntity entity, Message message, Channel channel) throws IOException {
        try {
            orderService.closeOrder(entity);
            //确认ack
            channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
        }catch (Exception e){
            //异常拒绝ack
            channel.basicReject(message.getMessageProperties().getDeliveryTag(),true);
        }
    }
}
```



# 参考
> - [尚硅谷视频](https://www.bilibili.com/video/BV1DJ411m7NR)  