# Spring Cloud

# 版本限定
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/springcloud版本.png)

# 组件升级说明

## 服务注册中心
- `Nacos`🉑
- ~~`Eureka`~~❌
- `Zookeeper` 
- `Consul`

## 服务调用
- `OpenFeign`🉑
- `Ribbon`❌
- `LoadBalancer`
- `Feign`❌

## 服务降级熔断
- `Sentinel`🉑
- ~~`Hystrix`~~❌
- `Resilience4j`

## 服务网关
- `Gateway`🉑
- ~~`Zuul`~~❌
- `Zuul2`⚠
- `kong`

## 服务配置
- `Nacos`🉑
- `Config`

## 服务总线
- `Nacos`🉑
- `Bus`

# RestTemplate
RestTemplate提供了多种便捷访问远程Http服务的方法，是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集   

- 使用restTemplate访问restful接口非常的简单粗暴无脑。
- (url, requestMap, ResponseBean.class)这三个参数分别代表。
- REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。

使用RestTemplate来实现基本服务间调用  

# Eureka服务治理 
Spring Cloud封装了Netflix 公司开发的Eureka模块来实现服务治理  

在传统的RPC远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册    

Eureka包含两个组件:`Eureka Server`和`Eureka Client`   
`Eureka Server`提供服务注册服务  
`Eureka Client`通过注册中心进行访问   
它是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒)   


## Eureka 集群  
让Eureka之间互相进行注册,如果是两个eureka 那么eureka1 的defaultZone 写 eureka2地址,2写1的地址   

```yml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #集群指向其它eureka
      defaultZone: http://eureka7001.com:7001/eureka/
      #单机就是7002自己
      #defaultZone: http://eureka7002.com:7001/eureka/
``` 

若是三个及以上,配置除本身外其他的EurekaServer即可   

```yml
eureka:
  client:
    # 是否将自己注册到注册中心 默认为true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有注册信息，默认为true。如果是集群部署 必须设置为true，否则ribbon无法负载
    fetch-registry: true
    service-url:
      # 注册中心的地址列表 多个用逗号隔开
      defaultZone: http://eureka2.xxx.cn:7002/eureka,http://eureka1.xxx.cn:7001/eureka
  instance:
    # 实例的ip
    ip-address: 127.0.0.1
    # 实例编号
    instance-id: payment8001
    # 调用方是否显示ip
    prefer-ip-address: true
    # 每隔多少秒心跳（续约）一次
    lease-renewal-interval-in-seconds: 1
    # 表示Eureka Server等待心跳的间隔，和Eureka服务端配置一致即可
    lease-expiration-duration-in-seconds: 2
```

使用@LoadBalanced注解赋予RestTemplate负载均衡的能力    

## 服务发现Discovery
对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息    
基于`org.springframework.cloud.client.discovery` 的 `DiscoveryClient`来进行基础的服务发现功能实现
同时主类要添加`@EnableDiscoveryClient`允许服务发现       
通过`http://localhost:8002/payment/discovery`访问获取返回的服务信息  

```java
    @Resource
    private DiscoveryClient discoveryClient;

    @GetMapping(value = "/payment/discovery")
    public Object discovery()
    {
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info("*****element: "+element);
        }

        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
          //CLOUD-PAYMENT-SERVICE	192.168.0.107	8002	http://192.168.0.107:8002
            log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
        }
        //{"services":["cloud-payment-service","cloud-order-service"],"order":0}
        return this.discoveryClient;
    }
```

## Eureka 的自我保护机制
如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式:   
`EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY’RE NOT. RENEWALS ARE LESSER THANTHRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUSTTO BE SAFE`   

此时Eureka Server 将会尝试保护其服务注册表中的信息,不在删除服务注册表中的数据,也就是不会注销任何微服务   
即:某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存   

### 禁止自我保护

使用`eureka.server.enable-self-preservation = false`可以禁用自我保护模式  

```yml
eureka:
  server:
    #关闭自我保护机制，保证不可用服务被及时踢除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

同时将客户端的信息改为如下,方便刷新

```yml
eureka:
  instance:
    instance-id: payment8001
    prefer-ip-address: true #添加此处
    # 每隔多少秒心跳（续约）一次
    lease-renewal-interval-in-seconds: 1
    # 表示Eureka Server等待心跳的间隔，和Eureka服务端配置一致即可
    lease-expiration-duration-in-seconds: 2
```

# Zookeeper 服务注册和发现  


# Consul 

# 三种注册中心区别
| 组件名    | 语言CAP | 服务健康检查 | 对外暴露接口 | Spring Cloud集成 |
| --------- | ------- | ------------ | ------------ | ---------------- |
| Eureka    | Java    | AP           | 可配支持     | HTTP             |
| Consul    | Go      | CP           | 支持         | HTTP/DNS         |
| Zookeeper | Java    | CP           | 支持客户端   | 已集成           |


CAP  
- `C Consistency` (强一致性)
- `A Availability` (可用性)
- `P Partition tolerance`（分区容错性) 分布式系统必须保证


# Ribbon
Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。   
Ribbon是Netflix发布的开源项目，主要功能是**提供客户端的软件负载均衡算法和服务调用**。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。     
就是在配置文件中列出Load Balancer(简称LB)后面所有的机器，Ribbon会自动的帮助你基于某种规则(如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。   

Ribbon目前也进入维护模式。未来可能被`Spring Cloud LoadBalacer`替代。   

> LB负载均衡(Load Balance)是什么:简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA (高可用)。

与Nginx 负载均衡的区别   
Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是由服务端实现的。   
Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术  
Nginx 是集中式负载均衡,即在服务消费方和提供方之间使用独立的LB措施      
Ribbon是进程内的负载均衡,将负载均衡逻辑集成在消费方,消费者从注册中心获知有那些地址可用,自己从中选择一个合适的服务器   

## Ribbon默认自带的负载规则 

- `RoundRobinRule` 轮询
- `RandomRule` 随机
- `RetryRule` 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试,获取可用服务
- `WeightedResponseTimeRule` 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- `BestAvailableRule` 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- `AvailabilityFilteringRule` 先过滤掉故障实例，再选择并发较小的实例
- `ZoneAvoidanceRule` 默认规则,复合判断server所在区域的性能和server的可用性选择服务器

## Ribbon负载实现 
配置依赖

```xml
<dependency>
    <groupld>org.springframework.cloud</groupld>
    <artifactld>spring-cloud-starter-netflix-ribbon</artifactid>
</dependency>

```

添加规则配置类   
**这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下,也就是说不要将Ribbon配置类与主启动类同包**  

```java
@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule(){
        return new RandomRule();//随机
        // return new RoundRobinRule(); 轮询
    }
}

@SpringBootApplication
@EnableEurekaClient
//添加到此处
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = MySelfRule.class)
public class OrderMain80
{
    public static void main( String[] args ){
        SpringApplication.run(OrderMain80.class, args);
    }
}
```

## Ribbon默认负载轮询算法原理  
默认负载轮训算法: `rest接口第几次请求数` % `服务器集群总数量` = `实际调用服务器位置下标`，每次服务重启动后rest接口计数从1开始。  

```java
List<Servicelnstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");

// 如:
List [0] instances = 127.0.0.1:8002
List [1] instances = 127.0.0.1:8001
// 8001+ 8002组合成为集群，它们共计2台机器，集群总数为2，按照轮询算法原理：

// 当总请求数为1时:1%2=1对应下标位置为1，则获得服务地址为127.0.0.1:8001
// 当总请求数位2时:2%2=О对应下标位置为0，则获得服务地址为127.0.0.1:8002
// 当总请求数位3时:3%2=1对应下标位置为1，则获得服务地址为127.0.0.1:8001
// 当总请求数位4时:4%2=О对应下标位置为0，则获得服务地址为127.0.0.1:8002
// 如此类推…
```
# Feign/OpenFeign 

声明式的REST Web 客户端,`Feign = Ribbon + RestTemplate`

> `Feign`是`Spring Cloud`组件中的一个轻量级RESTful的HTTP服务客户端Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是:使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务。   
> `OpenFeign`是`Spring Cloud`在`Feign`的基础上支持了SpringMVC的注解，如`@RequesMapping`等等。`OpenFeign`的`@Feignclient`可以解析SpringMVc的`@RequestMapping`注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。  


## OpenFeign 的超时控制  
`Feign`默认等待1秒钟，超过后报错   
Feign调用超时主要异常：`feign.RetryableException:Read timed out executing GET http://CLOUD-PAYMENT-SERVCE/payment/feign/timeout`  

通过配置`Ribbon`配置超时时间来进行超时控制(Feign内置了Ribbon)  

```yml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)(单位：毫秒)
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000

```

## OpenFeign 的日志打印功能  
通过配置日志级别,了解Feign中Http请求的细节   


- `NONE`：默认的，不显示任何日志;
- `BASIC`：仅记录请求方法、URL、响应状态码及执行时间;
- `HEADERS`：除了BASIC中定义的信息之外，还有请求和响应的头信息;
- `FULL`：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。

注意`Logger`使用`feign.Logger`包下的类  

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

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

```yml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.lun.springcloud.service.PaymentFeignService: debug
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/feign日志.png)


# Hystrix 断路器 (服务降级)
 
`Hystrix`是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等.     
**`Hystrix`能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。**

- 服务降级
  - 服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示，fallback
  - 程序运行导常
  - 超时
  - 服务熔断触发服务降级
  - 线程池/信号量打满也会导致服务降级

- 服务熔断
  - 类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示。
  - 服务的降级 -> 进而熔断 -> 恢复调用链路

- 服务限流
  - 秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行。


## 降级容错解决的维度要求
超时导致服务器变慢 变成 超时不再等待   
出错(宕机或程序运行出错)  变成 出错要有兜底

- 对方服务(8001)超时了，调用者(80)不能一直卡死等待，必须有服务降级。
- 对方服务(8001)down机了，调用者(80)不能一直卡死等待，必须有服务降级。
- 对方服务(8001)OK，调用者(80)自己出故障或有自我要求(自己的等待时间小于服务提供者)，自己处理降级。

## 单服务的降级配置
`@HystrixCommand` 设置自身调用超时时间的峰值，峰值内可以正常运行，超过了需要有兜底的方法处埋，作服务降级fallback    
主启动类添加 `@EnableCircuitBreaker` 

```java
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

/**
 * 类描述
 */
@Service
public class PaymentService {


    public String paymentInfo(Integer id) {
        return "线程池：" + Thread.currentThread().getName() + "info Id:" + id;
    }

//设置了超时时间  3s 超过3s 就会调用降级
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
    })
    public String paymentInfo_TimeOut(Integer id) {
        try {
            TimeUnit.MILLISECONDS.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池:  " + Thread.currentThread().getName() + " id:  " + id + "\t" + "O(∩_∩)O哈哈~" + "  耗时(秒): 3";
    }

    //用来善后的方法
    public String paymentInfo_TimeOutHandler(Integer id) {
        return "线程池:  " + Thread.currentThread().getName() + "  8001系统繁忙或者运行报错，请稍后再试,id:  " + id + "\t" + "o(╥﹏╥)o";
    }
}

```

一旦调用服务方式失败(或超时)并且抛出错误信息后,会自动调用`@HystrixCommand` 标注好的 `fallbackMethod`方法返回对应信息   

## Hystrix 全局服务降级DefaultProperties  
使用类注解 `@DefaultProperties` +方法注解 `@HystrixCommand`

```java
import com.netflix.hystrix.contrib.javanica.annotation.DefaultProperties;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@Slf4j
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
public class OrderHystirxController {
    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id)
    {
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
//    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
//            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
//    })
    @HystrixCommand//用全局的fallback方法
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        //int age = 10/0;
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }
    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id)
    {
        return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }

    // 下面是全局fallback方法
    public String payment_Global_FallbackMethod()
    {
        return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
    }
}

```

## Hystrix 通配服务降级FeignFallback 
通过`@FeignClient` 注解的 fallback 属性来配置通配服务降级   
创建@FeignService标注的接口的实现类,重写方法来进行方法通配    

```java
@Component
public class PaymentFallbackService implements PaymentService{

    @Override
    public String paymentInfo_OK(Integer id)
    {
        return "-----PaymentFallbackService fall back-paymentInfo_OK ,o(╥﹏╥)o";
    }

    @Override
    public String paymentInfo_TimeOut(Integer id)
    {
        return "-----PaymentFallbackService fall back-paymentInfo_TimeOut ,o(╥﹏╥)o";
    }
}


@Component
@FeignClient(value = "CLOUD-HYTRIX-SERVICE" ,fallback = PaymentFallbackService.class)
public interface PaymentService {
    @GetMapping("/payment/hystrix/ok/{id}")
    String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}

```

## Hystrix 服务熔断 

在Spring Cloud框架里，熔断机制通过`Hystrix`实现。`Hystrix`会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是`@HystrixCommand`

```java

    //====服务熔断
    @GetMapping("/payment/circuit/{id}")
    public String paymentCircuitBreaker(@PathVariable("id") Integer id)
    {
        String result = paymentService.paymentCircuitBreaker(id);
        log.info("****result: "+result);
        return result;
    }
```

```java
//=====服务熔断
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期 统计最近多久的请求 
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),// 失败率达到多少后跳闸
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
        if(id < 0) {
            throw new RuntimeException("******id 不能负数");
        }
        String serialNumber = IdUtil.simpleUUID();

        return Thread.currentThread().getName()+"\t"+"调用成功，流水号: " + serialNumber;
    }
    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
        return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " +id;
    }
```

当访问`/payment/circuit/-1` 时触发熔断机制,访问`/payment/circuit/1`正常返回   
多次访问`-1`而引发熔断占比到`errorThresholdPercentage`60% 时,访问正常请求也会返回错误信息, 停止访问后稍后自动恢复  

**关键参数:**   
- `快照时间窗`：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。
- `请求总数阀值`：在快照时间窗内，必须满足请求总数阀值才有资格熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用次数不足20次7,即使所有的请求都超时或其他原因失败，断路器都不会打开。
- `错误百分比阀值`：当请求总数在快照时间窗内超过了阀值，比如发生了30次调用，如果在这30次调用中，有15次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%阀值情况下，这时候就会将断路器打开。

断路器开启或者关闭的条件   
到达以下阀值，断路器将会开启： 
- 当满足一定的阀值的时候(默认10秒内超过20个请求次数)
- 当失败率达到一定的时候(默认10秒内超过50%的请求失败)
- 当开启的时候，所有请求都不会进行转发

一段时间之后(默认是5秒)，这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若失败，继续开启。  

**全部参数说明** 

```java
@HystrixCommand(fallbackMethod = "fallbackMethod", 
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
                    // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。
                    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
                    
                    // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过 circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50, 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。
                    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
                    // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，如果成功就设置为 "关闭" 状态。
                    @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
                    // 断路器强制打开
                    @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
                    // 断路器强制关闭
                    @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
                    // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
                    @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
                    
                    // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。
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
                    // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，否则将使用 LinkedBlockingQueue 实现的队列。
                    @HystrixProperty(name = "maxQueueSize", value = "-1"),
                    // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。
                    // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
                    @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
                }
               )
public String doSomething() {
	...
}

```

## Hystrix 图形化Dashboard搭建

添加图形化依赖 

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

```

详情看参考博客   


# Gateway 网关  
`Spring Cloud Gateway`的目标提供统一的路由方式且基于 Filter链的方式提供了网关基本的功能，例如:安全，监控/指标，和限流。  

## 主要作用
- 方向代理
- 鉴权
- 流量控制
- 熔断
- 日志监控

## 主要概念 
- `Route(路由)` - 路由是构建网关的基本模块,它由ID,目标URI,一系列的断言和过滤器组成,如断言为true则匹配该路由；
- `Predicate(断言)` - 参考的是Java8的java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数),如果请求与断言相匹配则进行路由；
- `Filter(过滤)` - 指的是Spring框架中GatewayFilter的实例,使用过滤器,可以在请求被路由前或者之后对请求进行修改。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/spring_cloud_gateway_diagram.png)

核心逻辑：**路由转发 + 执行过滤器链。**


## 配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

配置文件进行配置

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
#############################新增网关配置###########################
  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          #uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          #uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
####################################################################

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka

```

通过注入`RuteLocator Bean`进行配置

```java

@SpringBootApplication
public class DemogatewayApplication {
	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		return builder.routes()
			.route("path_route", r -> r.path("/get")
				.uri("http://httpbin.org"))
			.route("host_route", r -> r.host("*.myhost.org")
				.uri("http://httpbin.org"))
			.route("rewrite_route", r -> r.host("*.rewrite.org")
				.filters(f -> f.rewritePath("/foo/(?<segment>.*)", "/${segment}"))
				.uri("http://httpbin.org"))
			.route("hystrix_route", r -> r.host("*.hystrix.org")
				.filters(f -> f.hystrix(c -> c.setName("slowcmd")))
				.uri("http://httpbin.org"))
			.route("hystrix_fallback_route", r -> r.host("*.hystrixfallback.org")
				.filters(f -> f.hystrix(c -> c.setName("slowcmd").setFallbackUri("forward:/hystrixfallback")))
				.uri("http://httpbin.org"))
			.route("limit_route", r -> r
				.host("*.limited.org").and().path("/anything/**")
				.filters(f -> f.requestRateLimiter(c -> c.setRateLimiter(redisRateLimiter())))
				.uri("http://httpbin.org"))
			.build();
	}
}
```

```java
@Configuration
public class GateWayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder)
    {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();

        routes.route("path_route_atguigu",
                r -> r.path("/guonei")
                        .uri("http://news.baidu.com/guonei")).build();

        return routes.build();
    }
}

```

## 动态路由 
默认情况下`Gateway`会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能，而非写死一个地址  
需要注意的是uri的协议为`lb`，表示启用`Gateway`的负载均衡功能   

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
#############################新增网关配置###########################
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
####################################################################

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka

```

## GateWay常用的 Predicate 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/gatewaypredicates.png)

> 详情参考官网,有全套案例    
> 时间范围,Cookie(键值对),Header(`X-Request-Id`整数),Host,Method(POST/GET),Path(地址路径),Query(参数),RemoteAddr(请求地址)  

**Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理**


## GateWay的Filter/Global Filters
全局日志记录,统一网关鉴权等   

> 详见官网   

```java
@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("***********come in MyLogGateWayFilter:  "+new Date());

        //http://localhost:9527/payment/get/1?uname=123
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");

        if(uname == null)
        {
            log.info("*******用户名为null，非法用户，o(╥﹏╥)o");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }

        return chain.filter(exchange);
    }

    /**
     * 最先加载配置 顶层加载作用
     * @return
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```


# Config 分布式配置中心介绍

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

`SpringCloud`提供了`ConfigServer`来解决这个问题，我们每一个微服务自己带着一个`application.yml`，上百个配置文件的管理.

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。    
SpringCloud Config默认使用Git来存储配置文件(也有其它方式,比如支持SVN和本地文件)  
通过切换分支来切换不同的环境配置  

## 配置

```yml
server:
  port: 3344

spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: git@github.com:zzyybs/springcloud-config.git #GitHub上面的git仓库名字
        ####搜索目录
          search-paths:
            - springcloud-config
          # 使用ssh 协议需要配置密钥，使用https 协议需要填写用户名和密码
          #username: *****
          #password: ******
      ####读取分支
      label: master

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka


```

## 配置读取规则 
- `/{label}/{application}-{profile}.yml`（推荐）
  - `http://localhost:3344/master/config-dev.yml`(master分支)
  - `http://localhost:3344/dev/config-dev.yml`(dev分支)
- `/{application}-{profile}.yml`
  - `http://localhost:3344/config-dev.yml`
- `/{application}/{profile}/{label}`
  - `http://localhost:3344/config/dev/master`
- `/{application}-{profile}.properties`
- `/{label}/{application}-{profile}.properties`  

## Config客户端配置
bootstrap.yml

```yml
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

```java
@RestController
@RefreshScope
public class ConfigClientController
{
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo()
    {
        return configInfo;
    }
}

```

访问 http://localhost:3355/configInfo 即可看到配置项

## 问题局限
分布式配置的动态刷新问题  
- Linux运维修改GitHub上的配置文件内容做调整
- 刷新3344，发现ConfigServer配置中心立刻响应
- 刷新3355，发现ConfigClient客户端**没有任何响应**
- 3355没有变化除非自己重启或者重新加载

# Bus 消息总线  

`Spring Cloud Bus`是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了Java的事件处理机制和消息中间件的功能。`Spring Clud Bus`目前支持`RabbitMQ`和`Kafka`。  
`Spring Cloud Bus`能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道  

**基本原理:**    
`ConfigClient`实例都监听MQ中同一个topic(默认是Spring Cloud Bus)。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其它监听同一Topic的服务就能得到通知，然后去更新自身的配置   



## Bus动态刷新定点通知

`http://localhost:3344/actuator/bus-refresh/{destination}` 刷新运行在3355端口上的config-client


# Stream 消息驱动
`Spring Cloud Stream`是一个构建消息驱动微服务的框架     
不再关注具体MQ的细节，我们只需要用一种适配绑定的方式，自动的给我们在各种MQ内切换   
应用程序通过`inputs或者 outputs` 来与`Spring Cloud Stream`中`Binder对象`交互。  

通过我们配置来`binding(绑定)`，而`Spring Cloud Stream` 的`binder对象`负责与消息中间件交互。所以，我们**只需要搞清楚如何与Spring Cloud Stream交互**就可以方便使用消息驱动的方式。   

通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。   
`Spring Cloud Stream`为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了发布-订阅、消费组、分区的三个核心概念   

**屏蔽底层消息中间件的差异,降低切换成本,统一消息的编程模型**


## 编码API和常用注解

组成 | 说明 |
---------|----------|
 Middleware | 中间件，目前只支持RabbitMQ和Kafka |
 Binder | Binder是应用与消息中间件之间的封装，目前实行了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型(对应于Kafka的topic,RabbitMQ的exchange)，这些都可以通过配置文件来实现 | 
 @Input | 注解标识输入通道，通过该输乎通道接收到的消息进入应用程序 | 
 @Output | 注解标识输出通道，发布的消息将通过该通道离开应用程序 | 
 @StreamListener | 监听队列，用于消费者的队列的消息接收 | 
 @EnableBinding | 指信道channel和exchange绑定在一起 | 
 
## Stream生产者示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2022</artifactId>
        <groupId>com.ric.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ric.springcloud</groupId>
    <artifactId>cloud-stream-rabbitmq-provider8801</artifactId>
    <version>1.0-SNAPSHOT</version>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <!--基础配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>

```

```yml
server:
  port: 8801

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


```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8801 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8801.class,args);
    }
}

```

```java
import com.lun.springcloud.service.IMessageProvider;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.MessageChannel;

import javax.annotation.Resource;
import java.util.UUID;


@EnableBinding(Source.class) //定义消息的推送管道
public class MessageProviderImpl implements IMessageProvider
{
    @Resource
    private MessageChannel output; // 消息发送管道

    @Override
    public String send()
    {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println("*****serial: "+serial);
        return null;
    }
}

```

```java
import com.lun.springcloud.service.IMessageProvider;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class SendMessageController
{
    @Resource
    private IMessageProvider messageProvider;

    @GetMapping(value = "/sendMessage")
    public String sendMessage() {
        return messageProvider.send();
    }

}

```


## Stream消费者示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2022</artifactId>
        <groupId>com.ric.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ric.springcloud</groupId>
    <artifactId>cloud-stream-rabbitmq-consumer8802</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--基础配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

```

```yml
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
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址


```

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StreamMQMain8802 {
    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8802.class,args);
    }
}

```

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Component;


@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListenerController
{
    @Value("${server.port}")
    private String serverPort;


    @StreamListener(Sink.INPUT)
    public void input(Message<String> message)
    {
        System.out.println("消费者1号,----->接受到的消息: "+message.getPayload()+"\t  port: "+serverPort);
    }
}

```


## Stream之消息重复消费

多个消费端会重复消费同一条消息,使用Stream中的消息分组来解决消息的重复消费问题    
通过分组 `group: B_Group` 来区分不同服务,保证每个服务同一个消息只消费一次
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
            group: A_Group #<----------------------------------------关键

```


## Stream之消息持久化
`group` 属性同样可以避免消息丢失,先发送消息再启动消费者仍然可以接收消息   



# Sleuth 链路监控 


# SpringCloud Alibaba 

- [官网](https://spring.io/projects/spring-cloud-alibaba#overview)  
- [中文说明](https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md)
- [文档](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)


- `Sentinel`流量控制、熔断降级、系统负载保护
- `Nacos`动态服务发现、配置管理和服务管理
- `RocketMQ`一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- `Dubbo`Apache Dubbo™ 是一款高性能 Java RPC 框架。
- `Seata`阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
- `Alibaba Cloud OSS` 阿里云对象存储服务（Object Storage Service，简称 OSS）
- `Alibaba Cloud SchedulerX`分布式任务调度产

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.5.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```

## Nacos
Nacos就是注册中心＋配置中心的组合 -> `Nacos = Eureka+Config+Bus`

### Nacos 服务消费者注册和负载

配置详见 

[提供者](https://gitee.com/riiicc/cloud2022/tree/master/cloudalibaba-provider-payment9001)
[消费者](https://gitee.com/riiicc/cloud2022/tree/master/cloudalibaba-consumer-nacos-order83)


### Nacos 服务配置中心
Nacos 配置管理 dataId字段说明:   
- 格式默认为 `${prefix}-${spring.profiles.active}.${file-extension}`
- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置
- `spring.profiles.active` 即为当前环境对应的 `profile`， 当 spring.profiles.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型 

一般示例 `nacos-config-client-dev.yml`  

```yaml
# nacos配置
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nacos配置dataid命名规则.png)

**自带动态刷新**   
修改下Nacos中的yaml配置文件，再次调用查看配置的接口，就会发现配置已经刷新  


### Nacos 命名空间(Namespace)分组(Group)和DataID三者关系
类似Java里面的包名和类名  
`namespace` 是可以用于区分部署环境的,`Group` 和`DataID` 逻辑上区分两个目标对象    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Nacos之命名空间分组和DataID三者关系.png)

默认 `Namespace = public` `Group = DEFAULT_GROUP` 


例: 当前有三个开发环境 可以创建三个Namespace 


#### DataID 方案
默认空间+默认分组+新建dev和test两个DataID  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nacosdataid配置.png)

```yaml
# 配置active 来选择对应的动态配置
spring:
  profiles:
#    active: dev # 表示开发环境
    active: test # 表示测试环境
    #active: info

```

#### Group 方案 
通过Group实现环境区分 - 新建Group

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/group方案.png)

```yaml
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 
      config:
        server-addr: localhost:8848 
        file-extension: yaml 
        group: TEST_GROUP # 指定分组名称 DEV_GROUP
```

#### Namespace(命名空间)方案 

创建命名空间 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/命名空间namespace.png)

此时配置列表和服务列表多出菜单  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/配置列表命名空间.png)

```yaml
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 
      config:
        server-addr: localhost:8848 
        file-extension: yaml 
        group: TEST_GROUP  # 生效的的分组
        namespace: 2a2e6619-8b96-4acd-a9e1-09d10c888a3d #命名空间id
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/命名空间配置dev.png)


### Nacos 集群  


[集群部署文档](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)

- 集群部署需要Linux环境
- 需要切换到Mysql数据库(默认为`derby`数据库)
  - Apache Derby 是100% Java 编写的内存数据库
- 最终通过Nginx 的负载均衡实现集群的访问 


## Sentinel 
流量路由、流量控制、流量整形、熔断降级、系统自适应过载保护、热点流量防护等多个维度来帮助开发者保障微服务的稳定性  

[官网](https://sentinelguard.io/zh-cn/docs/introduction.html)

Sentinel 分为两个部分：  
- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 `Dubbo / Spring Cloud` 等框架也有较好的支持
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器 


### 安装

[Releases List](https://github.com/alibaba/Sentinel/releases)   
下载到本地sentinel-dashboard-1.7.0.jar 通过 `java -jar`运行控制台,登录账号密码均为`sentinel`    

需要被流控的项目配置 

```yml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sentinel控制台.png)

### 流量控制

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/流量控制.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/流量控制规则.png)

- 资源名：唯一名称，默认请求路径。
- 针对来源：Sentinel可以针对调用者进行限流，填写微服务名，默认default（不区分来源）。
- 阈值类型/单机阈值：
  - QPS(每秒钟的请求数量)︰当调用该API的QPS达到阈值的时候，进行限流。
  - 线程数：当调用该API的线程数达到阈值的时候，进行限流。
- 是否集群：不需要集群。
- 流控模式：
  - 直接：API达到限流条件时，直接限流。
  - 关联：当关联的资源达到阈值时，就限流自己。
  - 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流)【API级别的针对来源】。
- 流控效果：
  - 快速失败：直接失败，抛异常。
  - Warm up：根据Code Factor（冷加载因子，默认3）的值，从阈值，经过预热时长，才达到设置的QPS阈值。
  - 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效。



### 服务降级(熔断)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/降级规则.png)

- `RT`（平均响应时间，秒级）
  - 平均响应时间超出阈值(ms)且在时间窗口内通过的请求>=5，两个条件同时满足后触发降级。
  - 窗口期过后关闭断路器
  - RT最大4900（更大的需要通过-Dcsp.sentinel.statistic.max.rt=XXXX才能生效）

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/rt降级策略.png)

> 如果超过`200ms`还没有处理完,在未来`1s`内,触发服务降级,服务不可用


- 异常比列（秒级）
  - QPS >= 5且异常比例（秒级统计）超过阈值时，触发降级;时间窗口结束后，关闭降级 
  - 异常比率的阈值范围是 `[0.0, 1.0]`，代表 `0% - 100%`
`
> 如果请求方法异常比例超过设定阈值,在未来`xx(隔离时间) s`内,触发服务降级,服务不可用

- 异常数(分钟级)
  - 异常数(分钟统计）超过阈值时，触发降级;时间窗口结束后，关闭降级

> **时间窗口**:`Sentinel 1.8.0`之后添加了半开状态,经过熔断时长后熔断器会进入探测恢复状态（`HALF-OPEN` 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断


### 热点Key 

热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 `Top Key` 数据，并对其访问进行限制     
`@SentinelResource`



```java
@RestController
@Slf4j
public class FlowLimitController
{

    ...

    @GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey",blockHandler/*兜底方法*/ = "deal_testHotKey")
    public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                             @RequestParam(value = "p2",required = false) String p2) {
        //int age = 10/0;
        return "------testHotKey";
    }
    
    /*兜底方法*/
    public String deal_testHotKey (String p1, String p2, BlockException exception) {
        return "------deal_testHotKey,o(╥﹏╥)o";  //sentinel系统默认的提示：Blocked by Sentinel (flow limiting)
    }

}

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/热点key.png)

经过如上设置可以实现:
- 针对`hotkey方法`的第一个参数的访问`QPS超过1` 时会触发马上降级处理
- 异常信息用了我们自己定义的返回方法`deal_testHotKey` 而不会返回默认的`Blocked by Sentinel (flow limiting)`,类似`HystrixCommand`
- **若使用热点限流,必须自定义返回方法,即blockHandler,否则热点限流会报错**

我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样,特例:**假如当p1的值等于5时，它的阈值可以达到200**,我们可以如下设置

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/hotkey高级配置.png)


- 当p1等于5/10的时候，阈值变为100/200
- 当p1不等于5的时候，阈值就是平常的1

**热点参数的注意点，参数必须是基本类型或者String**

### Sentinel系统规则

[文档](https://github.com/alibaba/Sentinel/wiki/系统自适应限流)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sentinel系统规则.png)

`Sentinel` 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性

### 流控 @SentinelResource配置 


#### 两种资源配置方式

- 按资源名称限流 + 后续处理

```java
import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.atguigu.springcloud.alibaba.myhandler.CustomerBlockHandler;
import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RateLimitController {
    
    @GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException")
    public CommonResult byResource() {
        return new CommonResult(200,"按资源名称限流测试OK",new Payment(2020L,"serial001"));
    }
    
    public CommonResult handleException(BlockException exception) {
        return new CommonResult(444,exception.getClass().getCanonicalName()+"\t 服务不可用");
    }
}

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sentinel资源流控.png)

资源名配置为`byResource` 即可对当前接口进行流控,流控后返回自定义结果   
**注意资源最左侧没有/**   


- 按照Url地址限流 + 后续处理  

```java
@RestController
public class RateLimitController
{
	...

    @GetMapping("/rateLimit/byUrl")
    @SentinelResource(value = "byUrl")
    public CommonResult byUrl()
    {
        return new CommonResult(200,"按url限流测试OK",new Payment(2020L,"serial002"));
    }
}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sentinel-url流控.png)

url流控,资源名填入url(带`/`),限流后会返回Sentinel自带的限流处理结果`Blocked by Sentinel (flow limiting)`


#### 自定义统一限流处理类

创建 `CustomerBlockHandler` 类用于自定义限流处理逻辑,即达到限流条件后的处理逻辑方法   

```java
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;

public class CustomerBlockHandler {
    public static CommonResult handlerException(BlockException exception) {
        return new CommonResult(4444,"按客戶自定义,global handlerException----1");
    }
    
    public static CommonResult handlerException2(BlockException exception) {
        return new CommonResult(4444,"按客戶自定义,global handlerException----2");
    }
}

```

```java
@RestController
public class RateLimitController {
	...

    @GetMapping("/rateLimit/customerBlockHandler")
    @SentinelResource(value = "customerBlockHandler",
            blockHandlerClass = CustomerBlockHandler.class,//<-------- 指定限流处理类
            blockHandler = "handlerException2")//<-----------指定限流处理类中的具体方法
    public CommonResult customerBlockHandler()
    {
        return new CommonResult(200,"按客戶自定义",new Payment(2020L,"serial003"));
    }
}

```

#### 注解属性说明
[文档地址](https://github.com/alibaba/Sentinel/wiki/注解支持)


### 熔断 Sentinel服务熔断
熔断主要针对的是 `@SentinelResource`中`fallback`处理方法,详情可以参看上面的[注解支持文档](https://github.com/alibaba/Sentinel/wiki/注解支持),其中有注解属性的详细说明

`@SentinelResource(value = "fallback", fallback = "handlerFallback")`  
通过 fallback 配置处理异常的方法

```java
@RestController
@Slf4j
public class CircleBreakerController {
    
    public static final String SERVICE_URL = "http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;
 
    @RequestMapping("/consumer/fallback/{id}")
    //@SentinelResource(value = "fallback")//没有配置
    @SentinelResource(value = "fallback", fallback = "handlerFallback") //fallback只负责业务异常
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id,CommonResult.class,id);

        if (id == 4) {
            throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
        }else if (result.getData() == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
        }

        return result;
    }
    
    //本例是fallback
    public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
        Payment payment = new Payment(id,"null");
        return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
    }
    
}

```

访问`http://localhost:84/consumer/fallback/4` 页面返回`兜底异常nandlerFal1back, exception内容illegalkrgumentEBxceptiorn,非法参数异`


### Sentinel 支持Feign 

```yml
# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true

```

带`@Feignclient`注解的业务接口，`fallback = PaymentFallbackService.class`

```java
import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "nacos-payment-provider",fallback = PaymentFallbackService.class)
public interface PaymentService
{
    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
}

```

```java
import com.atguigu.springcloud.entities.CommonResult;
import com.atguigu.springcloud.entities.Payment;
import org.springframework.stereotype.Component;

@Component
public class PaymentFallbackService implements PaymentService {
    @Override
    public CommonResult<Payment> paymentSQL(Long id)
    {
        return new CommonResult<>(44444,"服务降级返回,---PaymentFallbackService",new Payment(id,"errorSerial"));
    }
}

```


### Sentinel 规则持久化  
一旦我们重启应用，sentinel规则将消失，生产环境需要将配置规则进行持久化  
将限流配置规则持久化进Nacos保存，只要刷新8401某个Rest地址，sentinel控制台的流控规则就能看到，只要Nacos里面的配置不删除，针对8401上sentinel上的流控规则持续有效

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>

```

```yml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719
      datasource: #<---------------------------关注点，添加Nacos数据源配置
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sentinel持久化nacos.png)

```json
[{
    "resource": "/rateLimit/byUrl",
    "limitApp": "default",
    "grade": 1,
    "count": 1, 
    "strategy": 0,
    "controlBehavior": 0,
    "clusterMode": false
}]
```

- `resource` ：资源名称；
- `limitApp` ：来源应用；
- `grade` ：阈值类型，0表示线程数, 1表示QPS；
- `count` ：单机阈值；
- `strategy` ：流控模式，0表示直接，1表示关联，2表示链路；
- `controlBehavior`：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待；
- `clusterMode`：是否集群。


## Seata 分布式事务 
`Seata`是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务  


### 产生原因
微服务环境下，一个业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证，**但是全局的数据一致性问题无法保证,由此产生分布式事务问题**  


### 主要过程说明  

分布式事务处理过程的一ID+三组件模型：
- `Transaction ID` XID 全局唯一的事务ID
- 三组件概念
  - TC (Transaction Coordinator) - 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或回滚。
  - TM (Transaction Manager) - 事务管理器：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
  - RM (Resource Manager) - 资源管理器：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/seata事务说明.png)

1. TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；
2. XID在微服务调用链路的上下文中传播；
3. RM向TC注册分支事务，将其纳入XID对应全局事务的管辖；
4. TM向TC发起针对XID的全局提交或回滚决议；
5. TC调度XID下管辖的全部分支事务完成提交或回滚请求。


### 使用
- server配置
  - 下载seata-server
  - `seata\conf`中 依据`application.example.yml`修改 `application.yml` 
  - `seata\script\server\db` 下的脚本创建数据库(选择db模式)

- client配置
  - 对应服务数据库内 `undo_log`建表

```sql
-- undo_log 表
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

[官网配置文档](http://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)

实现类业务方法添加 ` @GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)`



### 理解

TC、TM、RM关系

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/seata三种.png)



























# 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV18E411x7eT)
> - [csdn博客](https://blog.csdn.net/u011863024/article/details/114298270)