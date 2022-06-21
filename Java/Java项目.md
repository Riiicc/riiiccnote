# Java项目   

## 总结 
- Nacos 
  - standalone 启动
  - 动态配置 `@RefreshScope`
  - 利用命名空间做环境隔离 namespace group  
- gateway网关,路由过滤
  - 跨域配置
- 参数校验 `hibernate-validator` 
  - 常用注解 `@Email @NotNull @URL @Pattarn(regexp="")`
  - 分组校验 
  - 自定义实现校验注解 
- 统一异常处理 `@RestControllerAdivce` + ` @ExceptionHandler(value = {xxException.class})`
- 统一返回值设计,枚举类实现
- es
  - 倒排索引
  - DSL基本操作
  - 分词器
- gateway 针对域名进行断言的配置
- 压测 JMeter
- Redis
  - 缓存失效:穿透(key不存在直接查数据库),雪崩(key失效),击穿(热点key失效)
  - 分布式锁  
  - redisson  
  - 缓存一致性
  - Spring Cache 
- 线程池 
  - 七大参数,实际调用流程
- CompleteableFuture 异步
- spring 配置绑定 `@ConfigurationProperties`  
- 重定向传输数据 使用 `RedirectAttributes`
- MD5 Spring BCrypt
- Session共享 解决方案 
- 分布式Session 解决方案 `Spring Session` `@EnableRedisHttpSession`
- `ThreadLocal`
- RabbitMQ
  - Message
  - Publisher
  - Exchange
  - Queue
  - Binding
  - 可靠抵达,消息确认
  - 延时队列 TTL
  - 死信路由 DLX
  - 消息丢失 积压 重复
- 本地事务
  - 本地事务失效问题
- 分布式事务
  - CAP
  - BASE
  - Raft
  - 最大努力通知或者 可靠消息+最终一致性方案
- 定时任务 Quartz
- Sentinel 熔断,限流,降级
- K8s



## 安装Docker
https://docs.docker.com/engine/install/centos/

```bash
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine

sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo


sudo yum install docker-ce docker-ce-cli containerd.io

sudo systemctl start docker  

systemctl enable docker
```

## 后端项目构建

- 商品服务
- 仓储服务
- 订单服务
- 优惠券服务
- 用户服务

> `com.ric.gulimall.***`


结合SpringCloud Alibaba我们最终的技术搭配方案:  
SpringCloud Alibaba - Nacos: 注册中心(服务发现/注册)  
SpringCloud Alibaba . Nacos:配置中心(动态配置管理)  
SpringCloud- Ribbon:负载均衡  
SpringCloud- Feign:声明式HTTP客户端(调用远程服务)  
SpringCloud Alibaba - Sentinel:服务容错(限流、降级、熔断)  
SpringCloud . Gateway: API 网关(webflux 编程模式)  
SpringCloud- Sleuth:调用链监控   
SpringCloud Alibaba-Seata:原Fescar,即分布式事务解决方案   


### Nacos 
https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-discovery 

`启动 .\startup.cmd  -m standalone`
引入发现包 
```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2021.0.1.0</version>
        </dependency>
```


下载 nacos 服务器包     
https://nacos.io/zh-cn/docs/quick-start.html
启动 nacos 服务器   
启动命令   
`.\startup.cmd -m standalone`

配置 文件添加配置  
```yml
spring:
  application:
    name: gulimall-member
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```

启动类添加注解 `@EnableDiscoveryClient`



### feign 声明式远调用

http请求  

- 引入 open-feign
- 写接口

```java
@FeignClient("gulimall-coupon")
public interface CouponFeignService {

    @RequestMapping("/coupon/coupon/memberList")
    R memberCoupons();

}

```

- 注解开启 `@EnableFeignClients` 可以选择配置 `basepackage` 默认扫描同包下所有   


注： 由于SpringCloud Feign在Hoxton.M2 RELEASED版本之后不再使用 Ribbon 而是使用spring-cloud-loadbalancer   

需要排除Ribbon  引入spring-cloud-loadbalancer   详见common pom文件     


### nacos 作为配置中心   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli1.png)



https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config     

`@RefreshScope` 加在获取value 的controller 上  


最新版引入额外依赖，猜测可能在最新版 config starter 中移除了这个配置
```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-bootstrap -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
    <version>3.1.1</version>
</dependency>

```

利用命名空间做环境隔离   

```properties
spring.cloud.nacos.config.namespace=
spring.cloud.nacos.config.group=
```
也可以 同时加载多个配置文件  

配置中心存在的优先使用 配置中心内的属性  

```properties
在nacos浏览器中还可以配置：

命名空间：用作配置隔离。（一般每个微服务一个命名空间）

默认public。默认新增的配置都在public空间下

	开发、测试、开发可以用命名空间分割。properties每个空间有一份。

	在bootstrap.properties里配置

spring.cloud.nacos.config.namespace=b176a68a-6800-4648-833b-be10be8bab00  	# 可以选择对应的命名空间 ,即写上对应环境的命名空间ID
	
也可以为每个微服务配置一个命名空间，微服务互相隔离

配置集：一组相关或不相关配置项的集合。

配置集ID：类似于配置文件名，即Data ID

配置分组：默认所有的配置集都属于DEFAULT_GROUP。自己可以创建分组，比如双十一，618，双十二

spring.cloud.nacos.config.group=DEFAULT_GROUP  # 更改配置分组
最终方案：每个微服务创建自己的命名空间，然后使用配置分组区分环境（dev/test/prod）

加载多配置集
我们要把原来application.yml里的内容都分文件抽离出去。我们在nacos里创建好
后，在coupons里指定要导入的配置即可。

bootstrap.properties
spring.application.name=gulimall-coupon
spring.cloud.nacos.config.server-addr=192.168.11.1:8848


spring.cloud.nacos.config.namespace=ed042b3b-b7f3-4734-bdcb-0c516cb357d7  # # 可以选择对应的命名空间 ，即写上对应环境的命名空间ID
spring.cloud.nacos.config.group=dev  # 配置文件所在的组

spring.cloud.nacos.config.ext-config[0].data-id=datasource.yml
spring.cloud.nacos.config.ext-config[0].group=dev
spring.cloud.nacos.config.ext-config[0].refresh=true

spring.cloud.nacos.config.ext-config[1].data-id=mybatis.yml
spring.cloud.nacos.config.ext-config[1].group=dev
spring.cloud.nacos.config.ext-config[1].refresh=true

spring.cloud.nacos.config.ext-config[2].data-id=other.yml
spring.cloud.nacos.config.ext-config[2].group=dev
spring.cloud.nacos.config.ext-config[2].refresh=true


datasource.yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.1.103:3306/gulimall_sms?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: root

mybatis.yml
mybatis-plus:
  mapper-locations: classpath:/mapper/**/*.xml
  global-config:
    db-config:
      id-type: auto

other.yml
spring:
  application:
    name: gulimall-coupon
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.11.1:8848

server:
  port: 7000

```

### API网关  
路由,过滤,限流,监控  

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-query-route-predicate-factory  





## 前端项目构建


### ES6

- let const 
- 解构 字符串方法
- 箭头函数
- 对象优化
- map reduce
- promise 
- 



## 网关配置

网关 路径重写  说明
https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-rewritepath-gatewayfilter-factory

详见网关配置文件 

## 网关配置后的跨域问题 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli2.png)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli3.png)


网关 跨域配置
```java
@Configuration
public class CorsApiConfiguration {

    @Bean
    public CorsWebFilter corsWebFilter(){
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();

        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addAllowedOriginPattern("*");
        // corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.setAllowCredentials(true);


        source.registerCorsConfiguration("/**",corsConfiguration);
        return new CorsWebFilter(source);
    }
}
```

配置路由注意 模糊配置的覆盖问题, `/api/product/**` 会被 `/api/**` 覆盖,需要配置顺序 进行匹配,把范围高的放在后面匹配



## curd

plus 逻辑删除配置  
```
@TableLogic 
@TableLogic(value = "1",delval = "0")

mybatis-plus:
  global-config:
    db-config:
      id-type: auto
      #逻辑删除
      logic-delete-value: 1
      logic-not-delete-value: 0
```

## elementui
https://element.eleme.io/#/zh-CN/component/dialog 

相关配置,注意 配置项 的值 如果报错,就需要使用绑定  `:xxx="xxx"`  

tree 
 编辑 
 删除

 弹框 
 修改 弹框

 树形拖拽功能

## fast renren
快速开发vue页面   自动生成页面的使用




## oss 对象云存储 
相关配置

和使用 官方stater 进行简化配置  

新建第三方 类库 子项目 聚合所有第三方业务处理 `gulimall third party`   

https://help.aliyun.com/document_detail/31926.html 

文件上传配置 经过服务端签名后,前端直接上传    

![示意图](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli4.png)  



## 表单的校验
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli6.png)  

### 前端表单校验
https://element.eleme.io/#/zh-CN/component/form
默认校验和 自定义校验等



### 后端参数校验 
`JSR-303` 

给bean 添加校验注解  `javax.validation`

```xml
<dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.0.13.Final</version>
        </dependency>



```

`@Email` `@NotNull` `@URL` `@Pattarn(regexp="")` 正则 

需要在接口方法入参 添加 `@Valid` 注解 进行参数校验 

对应实体(内部)或者 参数前使用对应注解进行添加校验条件  

示例  
```java
    @RequestMapping("/save")
    public R save(@Valid @RequestBody CategoryEntity category,BindingResult result){
        if(result.hasErrors()){
            Map<String, String> map = new HashMap<>();
            Map<String, String> collect = result.getFieldErrors().stream().collect(Collectors.toMap(FieldError::getField, FieldError::getDefaultMessage));

            return R.error(400,"error").put("data",collect);
        }
		categoryService.save(category);

        return R.ok();
    }
```

### 配合校验功能的统一异常处理类

```java
@Slf4j
@RestControllerAdvice(basePackages = "com.ric.gulimall.product.controller")
public class GulimallExceptionControllerAdvice {

    @ExceptionHandler(value = {MethodArgumentNotValidException.class})
    public R handleValidException(MethodArgumentNotValidException e){
        log.error("数据校验出现问题{},异常类型{}",e.getMessage(),e.getClass());
        BindingResult result = e.getBindingResult();
        Map<String, String> collect = result.getFieldErrors().stream().collect(Collectors.toMap(FieldError::getField, FieldError::getDefaultMessage));
        return R.error().put("data",collect);
    }
}
```

错误返回示例

```json
{
    "msg": "未知异常，请联系管理员",
    "code": 500,
    "data": {
        "name": "品牌名称不能为空",
        "logo": "logo必须是一个合法的url地址",
        "sort": "不能为null",
        "firstLetter": "不能为空"
    }
}
```


### 统一返回code 参考设计

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli5.png)

可以将code值 作为枚举类型 方便统一管理
```java
public enum  CodeEnum {
    VALID_EXCEPTION(10001,"参数个hi校验失败"),
    ERROR_SYSTEM(10002,"系统错误");

    private int code;
    private String msg;

    CodeEnum(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

调用

`return R.error(CodeEnum.VALID_EXCEPTION.getCode(),CodeEnum.VALID_EXCEPTION.getMsg());`


### 分组校验
新增和修改校验条件不同 的处理     
使用分组校验  使用注解中的 groups字段

定义两个接口 `UpdateGroup.class`  

```java
  @NotNull(message = "修改必须指定id" ,groups = {UpdateGroup.class})
	@Null(message = "新增不能指定id",groups = {AddGroup.class})
	@TableId
	private Long brandId;
	/**
	 * 品牌名
	 */
	@NotBlank(message = "品牌名称不能为空",groups = {AddGroup.class,UpdateGroup.class})
	private String name;
```

在接口入参使用 `@Validated({AddGroup.class})` 进行配置     
如果字段没有进行 分组标识,那么校验 就不会生效   

```java
 @RequestMapping("/save")
    public R save(@Validated({AddGroup.class}) @RequestBody BrandEntity brand){
		brandService.save(brand);

        return R.ok();
    }
```

### 自定义校验注解   
- 编写自定义校验注解
- 编写自定义校验器
- 关联二者

部分代码

```java
/*校验注解*/
@Documented
@Constraint(
        validatedBy = {ListValueConstraintValidator.class}
)
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ListValue {

    String message() default "{com.ric.common.valid.ListValue.message}";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    int[] values() default {};
}

//com.ric.common.valid.ListValue.message 需要添加个 名为 ValidationMessages.properties  的配置文件 名字必须固定

/*校验器*/

/**
 * 自定义校验器 (校验逻辑)
 */
public class ListValueConstraintValidator implements ConstraintValidator<ListValue, Integer> {

    private Set<Integer> set = new HashSet<>();

    /**
     * @param integer 需要校验的值,入参
     * @param constraintValidatorContext
     * @return
     */
    @Override
    public boolean isValid(Integer integer, ConstraintValidatorContext constraintValidatorContext) {
        if (set.contains(integer)) {
            return true;
        }
        return false;
    }

    @Override
    public void initialize(ListValue constraintAnnotation) {
        int[] values = constraintAnnotation.values();
        for (int i : values) {
            set.add(i);
        }

    }
}

//调用示例 
	@ListValue(values = {0,1},groups = AddGroup.class)
	private Integer showStatus;
```
## SPU  SKU

SPU 商品
sku 具体型号的商品 期间的细微差距


## Vue 父子组件信息传递  

- 组件给调用组件的页面传递数据

使用事件机制.子组件给父页面发送事件  


## Cascader 级联选择器

`@JsonInclude(value = JsonInclude.Include.NON_EMPTY)` 可以设置 字段返回值为空时去除  

级联选择器 配置,可搜索

窗口关闭监听 ..

## 关于一些固定状态
固定状态采用枚举类型方便管理  

> 下面是两种形式 灵活应用

```java
public enum  CodeEnum {
    VALID_EXCEPTION(10001,"参数校验失败"),
    UNKNOWN_EXCEPTION(10002,"未知错误");

    private int code;
    private String msg;

    CodeEnum(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

```java
public class WareConstant {
    public enum PurchaseStatusEnum {
        CREATED(0, "新建"), ASSIGNED(1, "已分配"),
        RECEIVE(2, "已领取"), FINISH(3, "已完成"),
        ERROR(4, "有异常");

        PurchaseStatusEnum(int code, String msg) {
            this.code = code;
            this.msg = msg;
        }

        private int code;

        private String msg;

        public int getCode() {
            return code;
        }

        public String getMsg() {
            return msg;
        }
    }
}
```


# 基础篇 总结
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli6.png)

# 高级
## ES

- index 索引 （数据库）
- type 类型 （表）
- document 文档 （数据）-> 属性

### 安装
https://www.elastic.co/cn/downloads/elasticsearch

ES8 版本默认开启了 ssl 认证。

修改elasticsearch.yml配置文件

将xpack.security.enabled设置为false 后重启服务 
访问
`localhost:9200 `

- /_cat/nodes：查看所有节点 
- /_cat/health：查看ES健康状况 
-  /_cat/master：查看主节点信息
-  /_cat/indicies：查看所有索引
-  PUT http://192.168.163.131:9200/customer/external/1 索引一个文档(保存一条数据)

### 倒排索引


### 基本操作  
#### 索引一个文档  
`PUT http://192.168.163.131:9200/customer/external/1`  
`POST http://192.168.163.131:9200/customer/external/`

> PUT和POST都可以  
> ● POST新增，如果不指定id，会自动生成id。指定id就会修改这个数据，并新增版本号；  
> ● PUT可以新增也可以修改。PUT必须指定id；由于PUT需要指定id，我们一般用来做修改操作，不指定id会报错   


#### 查看文档 
`GET http://192.168.163.131:9200/customer/external/1` 

```json
{
    "_index": "customer",  # 在哪个索引(库)
    "_type": "external",   # 在哪个类型(表)
    "_id": "1",						 # 文档id(记录)
    "_version": 5,				 # 版本号
    "_seq_no": 4,					 # 并发控制字段，每次更新都会+1，用来做乐观锁
    "_primary_term": 1,		 # 同上，主分片重新分配，如重启，就会变化
    "found": true,
    "_source": {					 # 数据
        "name": "zhangsan"
    }
}

# 乐观锁更新时携带 ?_seq_no=0&_primary_term=1  当携带数据与实际值不匹配时更新失败
```

#### 更新文档   
`POST http://192.168.163.131:9200/customer/external/1/_update`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli7.png)  

几种更新文档的区别  
在上面索引文档即保存文档的时候介绍，还有两种更新文档的方式：  
- 当PUT请求带id，且有该id数据存在时，会更新文档；
- 当POST请求带id，与PUT相同，该id数据已经存在时，会更新文档； 


这两种请求类似，即带id，且数据存在，就会执行更新操作。
类比：
- 请求体的报文格式不同，`_update`方式要修改的数据要包裹在 `doc` 键下
- `_update`方式不会重复更新，数据已存在不会更新，版本号不会改变，另两种方式会重复更新（覆盖原来数据），版本号会改变
- 这几种方式在更新时都可以增加属性，PUT请求带id更新和POST请求带id更新，会直接覆盖原来的数据，不会在原来的属性里面新增属性  

#### 删除文档/索引

删除文档(数据) `DELETE http://192.168.163.131:9200/customer/external/1`   
删除 索引(数据库)`DELETE http://192.168.163.131:9200/customer`

#### 批量操作
通过传入`json`可以进行批量插入更新   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli8.png)    
   
对**所有索引**执行批量操作   
接口：`POST /_bulk`
通过 `Json` 属性来批量进行不同的操作
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/guli9.png)

> 这里的批量操作，当发生某一条执行发生失败时，其他的数据仍然能够接着执行，也就是说彼此之间是独立的

### 检索数据 
ES支持两种基本方式检索:  
- 通过REST request uri 发送搜索参数 （uri +检索参数）
- 通过REST request body 来发送它们（uri+请求体）
```
GET bank/_search?q=*&sort=account_number:asc
# q=* 查询所有
# sort=account_number:asc 按照account_number进行升序排列
```

后面这种也叫`QueryDSL`    

```json
{
  "query": {
    "match_all": { }
  },
  "sort": [
    {
      "account_number": "desc"
    }
  ]
}
```
 
搜索示例文档  
https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html#qs-search-data

### Query DSL
#### 基本语法格式

```json
QUERY_NAME:{
   ARGUMENT:VALUE,
   ARGUMENT:VALUE,...
}
```

```json
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 5,
  "sort": [
    {
      "account_number": {
        "order": "desc"
      },
      "balance": {
      	"order": "asc"
      }
    }
  ]
}


```

> match_all 查询类型【代表查询所有的所有】，es中可以在query中组合非常多的查询类型完成复杂查询；  
> from+size 限定，完成分页功能；从第几条数据开始，每页有多少数据  
> sort 排序，多字段排序，会在前序字段相等时后续字段内部排序，否则以前序为准；


#### 返回部分字段

> _source 指定返回结果中包含的字段名

```json
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 5,
  "sort": [
    {
      "account_number": {
        "order": "desc"
      }
    }
  ],
  "_source": ["balance","firstname"]
}

```

#### match-匹配查询

##### 精确查询-基本数据类型(非文本)

> 查找匹配 account_number 为 20 的数据 非文本推荐使用 `term`
```json
GET bank/_search
{
  "query": {
    "match": {
      "account_number": 20
    }
  }
}
```

##### 模糊查询-文本字符串

>  查找匹配 address 包含 mill 或 lane 的数据


```json
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill lane"
    }
  }
}
```

##### 精确匹配-文本字符串

> 查找 address 为 288 Mill Street 的数据。    
> 这里的查找是精确查找，只有完全匹配时才会查找出存在的记录，  
> 如果想模糊查询应该使用 `match_phrase` 短语匹配   

```json
GET bank/_search
{
  "query": {
    "match": {
      "address.keyword": "288 Mill Street"
    }
  }
}
``` 

- match_phrase-短语匹配

> 将需要匹配的值当成一整个单词（不分词）进行检索  
> 这里会检索 address 匹配包含短语 mill lane 的数据

```json
GET bank/_search
{
  "query": {
    "match_phrase": {
      "address": "mill lane"
    }
  }
}
```

##### multi_math-多字段匹配

> 检索 city 或 address 匹配包含 mill 的数据，会对查询条件分词


```json
GET bank/_search
{
  "query": {
    "multi_match": {
      "query": "mill",
      "fields": [
        "city",
        "address"
      ]
    }
  }
}
```

##### bool-复合查询  

> 复合语句可以合并，任何其他查询语句，包括符合语句。这也就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

> must：必须达到must所列举的所有条件   
> must_not，必须不匹配must_not所列举的所有条件。   
> should，应该满足should所列举的条件。   

> 查询 gender 为 M 且 address 包含 mill 的数据  


```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "gender": "M"
          }
        },
        {
          "match": {
            "address": "mill"
          }
        }
      ]
    }
  }
}
```  

##### filter-结果过滤  

> 并不是所有的查询都需要产生分数，特别是哪些仅用于filtering过滤的文档。  
> 为了不计算分数，elasticsearch会自动检查场景并且优化查询的执行。filter 对结果进行过滤，且**不计算相关性得分**。

> 这里先是查询所有匹配 address 包含 mill 的文档，   
> 然后再根据 10000<=balance<=20000 进行过滤查询结果  


```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "filter": {
        "range": {
          "balance": {
            "gte": "10000",
            "lte": "20000"
          }
        }
      }
    }
  }
}

```


>  在boolean查询中，must, should 和must_not 元素都被称为查询子句 。 
> 文档是否符合每个“must”或“should”子句中的标准，决定了文档的“相关性得分”。  
> 得分越高，文档越符合您的搜索条件。  
> 默认情况下，Elasticsearch 返回根据这些相关性得分排序的文档  

- term-精确检索

```json
GET bank/_search
{
  "query": {
    "term": {
      "age": "28"
    }
  }
}
# 查找 age 为 28 的数据
```

### 分词器   
```
POST customer/_analyze
{
   "analyzer": "ik_smart", 
   "text":"我是中国人"
}

//返回值
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国人",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}



```

自定义分词  

编辑 `elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml` 文件 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 -->
        <!-- <entry key="remote_ext_dict">words_location</entry> -->
        <entry key="remote_ext_dict">http://192.168.163.131/fenci.txt</entry>
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

### 基本配置

> 注意es 及其他插件的版本一定要统一


### 注意
数据若是对象 需要对_mapping 的属性设置为嵌入式 `nested` 


### 针对 自定义返回值 初始化设计要改为泛型类，方便数据转换和传输



### Feign 的调用流程
1. 构造请求数据，将对象转为json
2. 发送请求进行执行（执行成功会解码响应数据）
3. 执行请求会有重试机制



### Dev-tools thymeleaf


### nginx 反向代理 域名配置

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/gulinginx配置参考.png)



### gateway 针对域名进行断言的配置

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-host-route-predicate-factory

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: lb://gulimall-product
        predicates:
        - Host=**.gulimall.com,**.anotherhost.org
```


## 压力测试
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/性能指标说明.png)

### JMeter

报错解决  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jmeter.png)

## 性能监控 jconsole 与 jvisualvm


jvisualvm 插件地址修改

访问 https://visualvm.github.io/pluginscenters.html

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvisualvm.png)  

选择对应版本的连接 修改    
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/jvisualvm1.png)

示例 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/gcdemo.png)




## 优化参考

### Nginx 动静分离 
 直接用 nginx 代理静态文件，而不是通过项目进行进行静态资源的访问

### 代码优化
除了代码优化 还可以考虑一些插件自带的优化功能，比如thymeleaf 的缓存

### 调整内存
调整堆内存，减少 minor GC 和 **Full GC**次数  
尤其要避免 Full GC 
可以适当调大年轻代容量，让大对象可以在年轻代触发 young gc，调整大对象在年轻代的回收频次，尽可能保证大对象在年轻代回收


## 缓存 

> 适合放入缓存的:即时性,数据一致性 要求不高的
> 访问量大且更新频率不高的数据(读多 写少)


### redis 
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

使用spring boot 自动配置的 RedisTemplate 


#### redis 堆外内存溢出

使用默认的Lettuce 操作redis  容易出现堆外内存溢出  估计5.2 之后问题解决

推荐使用jedis 进行操作 

上述两种无论哪种都会自动配置 根据 `RedisConnectionFacotry` 自动配置

#### 高并发情况下的缓存失效

- 缓存穿透  

> 大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层 

解决方案 : 如果缓存和数据库都查不到某个 key 的数据就写一个空值(或者特定的值)到 Redis 中去并设置过期时间


- 缓存雪崩

> 缓存在同一时间大面积的失效(相同的过期时间,同时过期)，后面的请求都直接落到了数据库上，造成数据库短时间内承受大量请求

解决方案 : 设置不同失效时间,或者永不失效


- 缓存击穿

> 热点高频的key 突然失效  

解决方案: 加锁,由一个请求去请求数据库 然后保存到缓存中,其他请求再去缓存中查询  

本地锁单机可以解决上述问题   

分布式系统需要采用分布式锁  

#### 分布式锁  
> 所有的服务去操作同一个key,通过`set NX` 命令进行判断   
> 原子加锁  `Boolean boo =  redisTemplate.opsForvalue().setIfAbsent("lock","11")` 方法返回是否成功
> 原子解锁 使用lua 脚本实现    


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redislua.png)
```java
/**
* 从数据库查询并封装数据::分布式锁
*
* @return
*/
public Map<String, List<Catalogs2Vo>> getCatalogJsonFromDbWithRedisLock() {

    //1、占分布式锁。去redis占坑 设置过期时间必须和加锁是同步的，保证原子性（避免死锁）
    String uuid = UUID.randomUUID().toString();
    Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", uuid, 300, TimeUnit.SECONDS);
    if (lock) {
        System.out.println("获取分布式锁成功...");
        Map<String, List<Catalogs2Vo>> dataFromDb = null;
        try {
            //加锁成功...执行业务
            dataFromDb = getCatalogJsonFromDB();
        } finally {
            // lua 脚本解锁
            String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
            // 删除锁
            redisTemplate.execute(new DefaultRedisScript<>(script, Long.class), Collections.singletonList("lock"), uuid);
        }
        //先去redis查询下保证当前的锁是自己的
        //获取值对比，对比成功删除=原子性 lua脚本解锁
        // String lockValue = stringRedisTemplate.opsForValue().get("lock");
        // if (uuid.equals(lockValue)) {
        //     //删除我自己的锁
        //     stringRedisTemplate.delete("lock");
        // }
        return dataFromDb;
    } else {
        System.out.println("获取分布式锁失败...等待重试...");
        //加锁失败...重试机制
        //休眠一百毫秒
        try {
            TimeUnit.MILLISECONDS.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return getCatalogJsonFromDbWithRedisLock();     //自旋的方式
    }
}
```


https://redis.io/commands/set/  


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redisset.png)


#### 分布式锁工具 redisson (Redlock)
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redisson.png)


```xml
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.17.0</version>
</dependency>  

```

```java
public class IndexController {

    @Autowired
    private RedissonClient redissonClient;

    @GetMapping("index")
    public String indexPage() {

        RLock lock = redissonClient.getLock("lock");
        //如果拿不到锁就会进入等待 阻塞
        lock.lock();
        try {
            System.out.println("加锁成功.....");
            TimeUnit.SECONDS.sleep(30);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
            System.out.println("释放锁");
        }
        return "123";
    }
}
```

- 阻塞式等待锁
- 默认`lock.lock();`锁自动续期，业务处理时间长，锁会自动续期(10s定时续期)，不会导致锁过期删掉  
  - 若设定超时时间就不会自动续期,超时自动删除锁,自动解锁时间要大于业务执行时间  
- 加锁的业务只要运行完成，就不会给当前锁续期，即使不手动解锁，锁默认在30s 后删除`private long lockWatchdogTimeout = 30 * 1000;`  

- 读写锁,公平锁等均有实现
  - 读+读 无锁
  - 写 + 读 读等写
  - 写+写 阻塞
  - 读+写 写等读完

```java
//读写锁示例
@GetMapping("write")
    public String write() {
        RReadWriteLock writeLock = redissonClient.getReadWriteLock("readwritelock");
        RLock wLock = writeLock.writeLock();
        String s = "111111";
        try {
            wLock.lock();
            TimeUnit.SECONDS.sleep(30);

            redisTemplate.opsForValue().set("wrval", s);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            wLock.unlock();
        }
        return  s;
    }

    @GetMapping("read")
    public String read() {
        RReadWriteLock read = redissonClient.getReadWriteLock("readwritelock");
        RLock rLock = read.readLock();
        String s = "";
        try {
            rLock.lock();
            s = redisTemplate.opsForValue().get("wrval");
        }  finally {
            rLock.unlock();
        }
        return  s;
    }
```

- 信号量,信号量值为0 时acquire() 阻塞等待,不为0时立即恢复响应,
  - 可以用在限流情形
- `park.tryAcquire()` 不会阻塞 

```java
    @GetMapping("park")
    public String park() {
        RSemaphore park = redissonClient.getSemaphore("park");
        try {
            park.acquire();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "11";
    }
    @GetMapping("go")
    public String go() {
        RSemaphore park = redissonClient.getSemaphore("park");
        park.release();
        return "11";
    }
```



- 闭锁 CountDownLatch

```java
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.trySetCount(1);
latch.await();

// 在其他线程或其他JVM里
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.countDown();
```



#### 缓存一致性 

- 锁的名字就代表一个锁,名字相同就是同一把锁  

- 缓存和数据库的一致性如何保证
  - 双写模式: 修改数据库的同时更新缓存 
  - 失效模式: 修改数据库后删掉缓存数据,下次查询缓存自动更新 
  - 上述方式都有缺陷会导致缓存不一致问题,缓存数据+过期时间能够解决大部分缓存需求 

> canal - binlog解决缓存一致性

> 本项目的缓存一致性解决方案 : 
> 1. 缓存的所有数据都有过期时间,数据过期下一次查询触发主动更新 
> 2. 读写数据的时候加上分布式读写锁,若经常更新数据,建议直接操作数据库  


### Spring Cache  
https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache

> 使用 Spring Cache 来统一不同的缓存技术,使用注解简化开发

#### 配置  

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

自动配置:
● CacheAutoConfiguration 会导入 RedisCacheConfiguration;   
● 会自动装配缓存管理器 RedisCacheManager;  

```properties
spring.cache.type=redis

#spring.cache.cache-names=qq,毫秒为单位
spring.cache.redis.time-to-live=3600000

#如果指定了前缀就用我们指定的前缀，如果没有就默认使用缓存的名字作为前缀
#spring.cache.redis.key-prefix=CACHE_
spring.cache.redis.use-key-prefix=true

#是否缓存空值，防止缓存穿透
spring.cache.redis.cache-null-values=true
```

#### 常用注解
- `@Cacheable`  ：触发将数据保存到缓存的操作；
- `@CacheEvict`  : 触发将数据从缓存删除的操作；(失效模式)
- `@CachePut` ：不影响方法执行更新缓存；(双写模式)
- `@Cacheing`：组合以上多个操作；
- `@CacheConfig`：在类级别共享缓存的相同配置；


#### 使用
`@EnableCaching` 开启缓存 

> 注意 使用对应的 `@EnableCaching` 方法 必须时精油 bean 注入调用的才会生效 ，调用**本类注解标注的方法无效**

```java
    @Cacheable("skuinfo")
    @Override
    public List<CategoryEntity> listWithTree() {}
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/springcache.png)  

> key 默认自动生成,`缓存名字::SimpleKey[]`   
> 缓存的value 值 默认使用json序列化机制,将序列化后的数据存储到redis   
> `@Cacheable(value = {"category"}, key = "#root.method.name", sync = true)`
> 通过配置文件配置过期时间 `spring.cache.redis.time-to-live=3600000`

`spel`表达式 参考
https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache-spel-context


- 批量删除操作(Caching 组合操作) 

```java
    @Caching(evict = {
            @CacheEvict(value = "category",key = "'getlevelCateGorys'"),
            @CacheEvict(value = "category",key = "'getJson'")
    })
    //或者删除所有 'category' 分区 
    @CacheEvict(value = "category",allEntries = true)
```


## 线程池 

### 线程池七大参数说明

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) 
```

- `corePoolSize`: 核心线程数,池中一直**保持的线程的数量**，即使线程空闲。除非设置了 `allowCoreThreadTimeOut`
- `maximumPoolSize`: 池中允许的最大的线程数
- `keepAliveTime`: **存活时间**,当线程数大于核心线程数的时候，**超出核心线程数的线程**在最大多长时间没有接到新任务就会终止释放 ，最终线程池维持在 `corePoolSize` 大小
- `unit`: 时间单位
- `workQueue`: 阻塞队列，用来存储等待执行的任务，如果当前对线程的需求超过了 corePoolSize大小， 就 会放在这里 等待空闲线程执行.
- `threadFactory`：创建线程的工厂，比如指定线程名等
- `handler`：拒绝策略，如果线程满了，线程池就会使用拒绝策略。

### 工作流程

- 线程池创建,准备好 core 数量的核心线程,准备接受任务
  - core 线程占满,其他任务放入阻塞队列,空闲的 core 线程会自己去阻塞队列获取任务执行
  - 若阻塞队列满了 就直接开新线程执行,最大线程数 max
  - 若max 满了就 对后续线程执行拒绝策略拒绝任务
  - 多余的max 线程 在空闲指定时间 keepalivetime 后自动销毁,最终线程数为core 


> 面试：  
> 一个线程池 core 7； ； max 20  ，queue ：50 ，100 并发进来怎么分配的；   
> 先有 7 个能直接得到执行，接下来 50 个进入队列排队，在多开 13 个继续执行。现在 70 个被安排上了。剩下 30 个使用拒绝策略。

## CompleteableFuture 异步

### 创建异步对象 
> `runXxxx` 都是没有返回结果的， `supplyXxx` 都是可以获取返回结果的  
> 没有返回结果的 返回值为 `Void`  
> `executor` 为线程池,也可以不写使用默认线程池   

```java
  CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> { });
  CompletableFuture.runAsync(()->{},executor);

  CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 0);
  CompletableFuture.supplyAsync(()->{return 0;},executor);
```

### 执行完成时回调方法  

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T, ? super Throwable> action);

public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action);

public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action, Executor executor);

public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn)
```
- `whenComplete` 可以处理正常和异常的计算结果， `exceptionally` 处理异常情况。
- `whenComplete` 和 `whenCompleteAsync` 的区别：
  - `whenComplete` ：是执行当前任务的线程执行继续执行 `whenComplete` 的任务。
  - `whenCompleteAsync` ：是执行把 `whenCompleteAsync` 这个任务继续提交给线程池来进行执行。

> 方法不以 Async 结尾，意味着 Action 使用相同的线程执行，而 Async 可能会使用其他线程执行（如果是使用相同的线程池，也可能会被同一个线程选中执行）

```java
 CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("Thread.currentThread().getName() = " + Thread.currentThread().getName());
            // int i = 10 / 2;
            int i = 10 / 0;
            System.out.println("i = " + i);
            return i;
        }, executor);

        completableFuture.whenComplete((res,exp)->{
          //返回值
            System.out.println("res = " + res);
            //异常
            System.out.println("exp = " + exp);
        }).exceptionally(exp->{
          //若无异常这里不会执行
            System.out.println("exp = " + exp);
            return 10;
        });
```

### handle 方法

> handle 方法和 complete 方法类似,但是 可以改变返回值

```java
        CompletableFuture<Integer> handle = completableFuture.handleAsync((res, exp) -> {
            if (res != null) {
                return 2;
            }
            if (exp != null) {
                return 3;
            }
            return 0;
        },executor);

```

### 线程串行化方法
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/线程串行化.png)

### 两任务组合 都要完成
> 两个任务必须都完成，触发该任务。每个种类都有三个重载方法,分别是当前线程,异步(默认线程池),异步(指定线程池)    
> `thenCombine(Async)` ：组合两个 future，获取两个 future 的返回结果，并返回当前任务的返回值  
> `thenAcceptBoth(Async)` ：组合两个 future，获取两个 future 任务的返回结果，然后处理任务，没有返回值。  
> `runAfterBoth(Async)` ：组合两个 future，不需要获取 future 的结果，只需两个 future 处理完任务后，处理该任务。


### 两任务组合  任意一个完成

> 当两个任务中，任意一个 future 任务完成的时候，执行任务。每个种类都有三个重载方法,分别是当前线程,异步(默认线程池),异步(指定线程池)       
> `applyToEither(Async)` ：两个任务有一个执行完成，获取它的返回值，处理任务并**有新的返回值**。  
> `acceptEither(Async)` ：两个任务有一个执行完成，获取它的返回值，处理任务，没有新的返回值。   
> `runAfterEither(Async)` ：两个任务有一个执行完成，不需要获取 future 的结果，处理任务，也没有返回值。   


### 多任务组合 
> `allOf` ：等待所有任务完成  
> `anyOf` ：只要有一个任务完成,并返回成功任务的返回值     

```java
  CompletableFuture<Void> completableFuture = CompletableFuture.allOf(future1, future2);
  CompletableFuture<Object> future = CompletableFuture.anyOf(future1, future2);
  //下面两个方法都可以用做阻塞等待
  completableFuture.join();
  //这里获取的数据就是成功执行的任务返回值
  Object o = future.get();

```

## spring 配置线程池以及配置绑定
详细配置原理参考spring boot 教程 配置绑定   

```java
@Configuration
public class MyThreadConfig {

    @Bean
    public ThreadPoolExecutor threadPoolExecutor(ThreadPoolConfigProperties configProperties){
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(configProperties.getCoreSize(), configProperties.getMaxSize(), configProperties.getKeepAliveTime(),
                TimeUnit.SECONDS, new ArrayBlockingQueue<>(10), Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());

        return threadPoolExecutor;
    }
}
```

```java
@ConfigurationProperties(prefix = "mall.thread")
@Component
@Data
public class ThreadPoolConfigProperties {

    private Integer coreSize;
    private Integer maxSize;
    private Integer keepAliveTime;
}

```

```yaml
mall:
  thread:
    core-size: 20
    max-size: 50
    keep-alive-time: 10
```

## 身份认证  

### 短信校验 

### 重定向
重定向传输数据 使用 `RedirectAttributes` 类似`Model`


### 异常处理 -自定义异常

```java

public class UsernameExistException extends RuntimeException{
    public UsernameExistException() {
        super("用户名已经存在了铁汁");
    }
}
```

### MD5 加密 和spring BCrypt 加密


### 社交登录
略

### session  
session 共享问题 及session原理   

> session真正的值 会保存到服务器中，若是分布式环境多个服务器，或者域名(服务)不同， 将会存在session 同步问题，如何做到session共享    

> 注意理解 sessionManager
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/session.png)


传统解决session共享方案   
- `session复制`，Tomcat 原生支持session复制，同步传输session
- `客户端存储` 将session 信息保存到客户端cookie中（cookie长度有限制，泄露信息）  
- `ip_hash` nginx ip_hash 固定访问某台服务器  
- `统一存储` 使用redis 统一存储session 信息  


### 分布式session 解决方案Spring session  

https://spring.io/projects/spring-session  

登录后保存session 到redis中,并且浏览器保存的session domain 是`gulimall.com`   

- 引入依赖,需要使用session信息的都要引入
```xml
<!-- 整合 spring session 实现 session 共享-->
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

- 配置文件 application.yaml`,省略的redis 的配置信息

```yaml
spring:
  session:
    store-type: redis
```

- 配置注解 `@EnableRedisHttpSession`  

> 至此可以实现基本的spring session 使用
> 但是redis 中存储的 内容是字节码内容,需要配置序列化器进行自定义JSON序列化,使其可视化  
> 需要放大session 作用域 到根域名,不同服务之间可以使用同一个session 


- 配置文件

```java
@Configuration
public class GulimallSessionConfig {
    @Bean
    public CookieSerializer cookieSerializer() {
        DefaultCookieSerializer cookieSerializer = new DefaultCookieSerializer();
        //放大作用域
        cookieSerializer.setDomainName("gulimall.com");
        //自定义session key
        cookieSerializer.setCookieName("GULISESSION");
        return cookieSerializer;
    }

    @Bean
    public RedisSerializer<Object> springSessionDefaultRedisSerializer() {
        return new GenericJackson2JsonRedisSerializer();
    }
}
```


### Spring Session 原理 


## 单点登录  

> 客户端示例

```java
@GetMapping("/emp")
    public String emp(Model model, HttpSession session,String token){
        List<String> emps = new ArrayList<>();

        if (!StringUtils.isEmpty(token)) {
            RestTemplate restTemplate=new RestTemplate();
            //根据token去授权中心 请求用户信息
            ResponseEntity<String> forEntity = restTemplate.getForEntity("http://sso.mroldx.cn:8080/userinfo?token=" + token, String.class);
            String body = forEntity.getBody();

            session.setAttribute("loginUser", body);
        }

        Object loginUser = session.getAttribute("loginUser");
        if(loginUser==null){
            //未登录
            return "redirect:"+"http://ssoserver.com:7009/login.html?redirect_url=http://localhost:7008/emp";
        }else {
            emps.add("张三");
            emps.add("李四");
            model.addAttribute("emps", emps);
            return "employees";
        }
    }
```


> 授权中心示例   


```java
@Controller
public class LoginController {

    @Autowired
    StringRedisTemplate redisTemplate;

    @ResponseBody
    @GetMapping("/userinfo")
    public String userinfo(@RequestParam(value = "token") String token) {
        String s = redisTemplate.opsForValue().get(token);

        return s;

    }


    @GetMapping("/login.html")
    public String loginPage(@RequestParam("redirect_url") String url, Model model, @CookieValue(value = "sso_token", required = false) String sso_token) {
        if (!StringUtils.isEmpty(sso_token)) {
            //cookie不为空说明登陆过,则直接获取token 重定向为源地址

            return "redirect:" + url + "?token=" + sso_token;
        }
        model.addAttribute("url", url);

        return "login";
    }

    @PostMapping(value = "/doLogin")
    public String doLogin(@RequestParam("username") String username, @RequestParam("password") String password,
                          @RequestParam("redirect_url") String url, HttpServletResponse response) {

        //登录成功跳转，跳回到登录页 并将token放入cookie中 下次访问直接查询
        if (!StringUtils.isEmpty(username) && !StringUtils.isEmpty(password)) {

            String uuid = UUID.randomUUID().toString().replace("_", "");
            redisTemplate.opsForValue().set(uuid, username);
            Cookie sso_token = new Cookie("sso_token", uuid);

            response.addCookie(sso_token);
            return "redirect:" + url + "?token=" + uuid;
        }
        return "login";
    }

}

```


## ThreadLocal 购物车
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/thread-local.png)

`Map<Thread,Object> threadLocal`  

```java
public static ThreadLocal<UserInfoTo> toThreadLocal = new ThreadLocal<>();

// 赋值
toThreadLocal.set(userInfoTo);

// 取值
UserInfoTo userInfoTo = CartInterceptor.toThreadLocal.get();
```

## 重定向携带数据 RedirectAttributes

> `RedirectAttributes`
> `addFlashAttribute` 将数据放在 session 里面可以在页面取出,但是只能取出一次   
> `addAttribute`  数据放在连接后自动拼接 到重定向连接后 `****.com?skuId=**` 


## 消息队列 RabbitMQ

- 异步处理
- 引用解耦
- 流量控制/流量削峰

### 常见概念
- 消息代理 `message broker` 
- 目的地 `destination`   

> 消息发送者发送消息后,将由消息代理接管,消息代理保证消息传递到指定**目的地**
> 消息队列主要有两种形式的目的地

- 队列 `queue` 点对点消息通信
  - 消息发送->进入队列->消费
  - 消息只有唯一的发送者和接收者
- 主题 `topic` 发布/订阅 消息通信 
  - 消息发送->主题->多个订阅者监听主题-> 消息到达所有订阅者都可以收到消息 

- JMS `java message service`
- AMQP `Advanced Message Queuing Protocol`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/rabbitmq.png)  


### RabbitMQ 概念 

- Message 消息

> 消息由 消息头 + 消息体 而消息头则由一系列的可选属性组成， 这些属性包括 `routing-key` （路由键）、 `priority`（相对于其他消息的优先权）、`delivery-mode`（指出该消息可 能需要持久性存储）等  

- Publisher

> 消息的生产者，也是一个向交换器发布消息的客户端应用程序  

- Exchange
 
> 交换机，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。    
> Exchange有4种类型：direct(默认)，fanout,topic,和~~headers~~，不同类型的Exchange转发消息的策略有所区别  
> direct 绑定路由键，只有对应的路由键能收到  
> fanout 广播,不处理路由键，所有队列都会收到消息  
> topic 订阅发布模式，通配符 `*/#` 可以对应一个或多个队列   

- Queue

> 消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走

- Binding  
  
> 绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。   
> Exchange 和 Queue 的绑定可以是多对多的关系

### 整合springboot
- 引入 `spring-boot-stater-amqp`
- yaml 配置、 `@EnableRabbit`,监听消息需要这个注解,若是单纯发送消息不需要    
- AmqdAdmin 组件/RabbitTemplate 组件 
- 监听消息 : `@RabbitListener` 必须标注在容器中的方法,或类上, `@RabbitHandler` 只能标注在方法上 
  - 一般使用 `@RabbitListener` 标注在类上监听队列,使用 `@RabbitHandler` 处理队列中的不同入参类型消息

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

发送消息测试  

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
        //发送的消息是对象 就会序列化对象 需要配置序列化器进行自定义序列化 ,一般需要指定 CorrelationData 
        rabbitTemplate.convertAndSend("hello-java-exchange","hello.java",entity,new CorrelationData(UUID.randomUUID().toString()));
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

自定义序列化器，注意这里和redis 的序列化器的区别

```java
@Configuration
public class MyRabbitConfig {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

监听消息测试  

```java
 /**
     * @param msg  原生消息详细信息 消息头+ 消息体
     * @param entity 发送的消息类型 , spring 会自动转化消息体为当前类型
     * @param channel 当前传输数据的通道  
     */
    @RabbitListener(queues = {"hello-java-queue"})
    public void revMessage(Message msg, OrderReturnReasonEntity entity, Channel channel){
        //消息体
        byte[] body = msg.getBody();
        //消息头属性
        MessageProperties messageProperties = msg.getMessageProperties();

        System.out.println("object = " + msg);
        System.out.println("entity = " + entity);
    }
```

> 同一个消息只能有一个客户端收到   
> 只有一个消息完全处理完成(处理方法运行结束), 才能接收到下一个消息   

### 消息确认机制 可靠抵达

> 保证消息不丢失,收发都能可靠抵达,使用事务消息,对性能影响严重

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/rabbitmq可靠抵达.png)

- 生产端的确认机制
  - `publisher` `confirmCallback` 确认模式  
  - `publisher` `returnCallback` 未投递到 `queue` 退回模式
- 客户端的确认机制
  - `consumer` ack机制,手动ack  

配置文件   

> 在2.2.0及之后该属性 `publisher-confirm` 过期使用`spring.rabbitmq.publisher-confirm-type`属性配置代替，用来配置更多的确认类型

```yaml
spring:
  application:
    name: gulimall-order
  rabbitmq:
    host: localhost
    port: 5672
    virtual-host: /
    # 开启发送端确认
    publisher-confirm-type: correlated
    # 开启发送端消息抵达队列确认
    publisher-returns: true
    # 只要抵达队列,以异步发送优先回调 return confirm
    template:
      mandatory: true
    # 手动ack 
    listener:
      simple:
      # none 默认 auto 自动，manual 手动
        acknowledge-mode: manual
```


####  生产端的确认机制

>  `confirmCallback`  `returnCallback`

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
         * ack 消息是否成功收到
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

> 在创建 `connectionFactory` 的时候设置 `PublisherConfirms(true)` 选项，开启 `confirmcallback` 。   
> `CorrelationData` ：用来表示当前消息唯一性。
> 消息只要被 broker 接收到就会执行 confirmCallback，如果是 cluster 模式，需要所有broker 接收到才会调用 confirmCallback。  
> 被 broker 接收到只能表示 message 已经到达服务器，并**不能保证消息一定会被投递到目标 queue 里**。所以需要用到接下来的 `returnCallback` 。

#### 客户端的确认机制 ack

```yaml
    # 手动ack 
    listener:
      simple:
      # none 默认 auto 自动，manual 手动
        acknowledge-mode: manual
``` 
> 自动确认`AcknowledgeMode.NONE` 默认为自动确认，不管消费者是否成功处理了消息，消息都会从队列中被移除。  
> 根据情况确认`AcknowledgeMode.AUTO` 根据方法的执行情况来决定是否确认还是拒绝（是否重新入队列）   
> 如果消息成功被消费（成功的意思是在消费的过程中没有抛出异常），则自动确认   
> 当抛出AmqpRejectAndDontRequeueException 异常的时候，则消息会被拒绝，且消息不会重回队列    
> 当抛出 ImmediateAcknowledgeAmqpException 异常，则消费者会被确认     
> 其他的异常，则消息会被拒绝，并且该消息会重回队列，如果此时只有一个消费者监听该队列，则有发生死循环的风险，多消费端也会造成资源的极大浪费，这个在开发过程中一定要避免的。可以通过 setDefaultRequeueRejected（默认是true）去设置   
> `AcknowledgeMode.MANUAL`  手动确认   


> 客户端的默认自动确认,只要消息收到,客户端就会自动确认   
> 若只有一个消息处理成功，消费端突然宕机了，结果MQ中剩下的消息全部丢失了   
> 消费端如果无法确定此消息是否被处理完成，可以手动确认消息，即处理一个确认一个，未确认的消息不会被删除   

> 只要我们没有明确告诉MQ收到消息。没有 Ack，消息就一直是 Unacked 状态，即使 consumer 宕机，消息也不会丢失，会重新变为 Ready，等待下次有新的 Consumer 连接进来时，再发给新的 Consumer
> 消费者获取到消息，成功处理，可以回复 Ack 给 Broker    
> `ack()` 用于肯定确认；broker 将移除此消息   
> `nack()` 用于否定确认；可以指定 broker 是否丢弃此消息，可以批量   
> `reject()` 用于否定确认；同上，但不能批量   

```java
        long deliveryTag = msg.getMessageProperties().getDeliveryTag();

        try {
            //确认签收
            channel.basicAck(deliveryTag,false);
            //否定确认,后面的false 代表是否重新入队,若为true 那么拒绝后的消息会再次插入队列
            channel.basicNack(deliveryTag,false,false);
            //拒绝
            channel.basicReject(deliveryTag,false);
        } catch (IOException e) {
            e.printStackTrace();
        }
```

> 实际应用场景: ` channel.basicAck(deliveryTag,false);` 签收,业务完成成功
> `channel.basicNack(deliveryTag,false,true);` 拒签,业务失败,拒签,再次尝试


> 消息如果一直没有调用ack()的话，则会一直处于 Unacked 状态，这些 Unacked 状态的消息，都不会被丢弃，
> 如果客户端宕机，等服务端感知到消费端宕机了，它就会将这个消息改为 Ready 状态，Ready 状态的消息，全部都会被重新投递  

### 延时队列   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/延时队列场景.png)   

#### TTL （Time To Live）
> RabbitMQ 可以对队列和消息分别设置TTL,二者同时设置取最小值   
> 通过设置消息的 `expiration` 字段或者 `x-message-ttl` 属性来设置时间   
> 推荐设置队列的过期时间  

> 消息的TTL就是消息的存活时间   
> 就是指这个消息只要在我们指定的时间里边没有被人消费，那这个消息就相当于没用了，我们就把称为死信，然后这个消息相当于死了。
#### 死信路由（Dead Letter Exchanges）DLX  

死信条件:  
1.  被Consumer拒收了，并且reject方法的参数里requeue是false。  
2.  消息的TTL到了，消息过期了。   
3.  假设队列的长度限制满了，排在前面的消息就会被丢弃或者扔到死信路由上   

### 消息积压、丢失、重复

####  消息丢失  

- 消息发送出去，由于网络问题没有抵达服务器  
  - 做好容错方法（try-catch），发送消息可能会网络失败，失败后要有重试机制，可记录到数据库，采用定期扫描重发的方式 
  - 做好日志记录，每个消息状态是否都被服务器收到都应该记录，可以创建一张关于消息的数据表，存到数据库里  

```sql
CREATE TABLE `mq_message` (
`message_id` char(32) NOT NULL,
`content` text,
`to_exchane` varchar(255) DEFAULT NULL,
`routing_key` varchar(255) DEFAULT NULL,
`class_type` varchar(255) DEFAULT NULL,
`message_status` int(1) DEFAULT '0' COMMENT '0-新建 1-已发送 2-错误抵达 3-已抵达',
`create_time` datetime DEFAULT NULL,
`update_time` datetime DEFAULT NULL,
PRIMARY KEY (`message_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

- 消息抵达Broker之后，Broker要将消息写入磁盘（持久化）时宕机  
  - publisher必须加入确认回调机制，确认成功的消息，修改数据库消息状态
- 自动ACK的状态下。消费者收到消息，但没来得及消息然后宕机 
  - 开启手动ACK，消费成功再移除，失败或者没来得及处理就noAck并重新入队

> 1. 做好消息确认机制 手动ack确认   
> 2. 每个发送的消息做好数据库记录,定期将失败的消息重新发送    

####  消息重复

1. 消费者的业务消费接口应该设计为幂等性的。比如扣库存有工作单的状态   
2. 使用防重表（redis/mysql），发送消息每一个都有业务的唯一标识，处理过就不用处理   
3. `rabbitMQ` 的每一个消息都有 `redelivered` 字段，可以获取是否是被重新投递过来的，而不是第一次投递过来的   


#### 消息积压  

> 消费者宕机积压    
> 消费者消费能力不足积压   
> 发送者发送流量太大     

- 上线更多的消费者，进行正常消费
- 上线专门的队列消费服务，将消息先批量取出来，记录数据库，离线慢慢处理



## 订单服务  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/订单流程.png)  

### 订单完整消息队列设计   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/订单服务完整消息队列.png) 


### 库存自动解锁场景
- 下单成功,订单过期没有支付系统自动取消,用户手动取消,都需要解锁库存
- 下单成功后业务调用失败,导致订单回滚,需要解锁库存  





## Feign 远程调用丢失请求头问题

> 使用feign 远程调用请求拦截器 ,使用拦截器处理 添加请求头 session信息等   

```java
@Configuration
public class FeignConfig {

    @Bean
    public RequestInterceptor requestInterceptor(){
        RequestInterceptor requestInterceptor = new RequestInterceptor(){
            @Override
            public void apply(RequestTemplate requestTemplate) {
                //使用 RequestContextHolder
                ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                HttpServletRequest request = requestAttributes.getRequest();
                //同步请求头数据
                String cookie = request.getHeader("Cookie");

                requestTemplate.header("Cookie",cookie);
            }
        };


        return requestInterceptor;
    }
}

```

### Feign 远程调用添加请求头 在多线程情况下的问题

> 上述问题解决后,在多线程异步调用feign 接口时又出现了无法获取 threadlocal 用户信息的问题  
> 因为使用异步调用,所以无法共享 `threadlocal` 信息 

> 解决方案: 在每个异步线程中重新赋值 `requestAttributes` ,主线程中获取,异步线程中赋值    



## 幂等性

> 接口幂等性就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的  
> 类似于数学里的幂等，1的1次幂，和1的100次幂，结果都是1

示例场景:
1. 用户多次点击按钮
2. 用户页面回退再次提交
3. 微服务互相调用，由于网络问题，导致请求失败。feign 触发重试机制
4. 其他业务情况

幂等解决方案   
1. token  机制 
2. 悲观锁\乐观锁\分布式锁
3. 数据库唯一索引 `UNIQUE`\防重表

> token 机制一般放在 redis 中,但是要注意 token 的**获取、比较和删除必须是原子操作**   
> 可以使用lua 脚本来操作  
> `if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end`

### 原子验证令牌 Redis 示例

```java
public SubmitOrderResponseVo submitOrder(OrderSubmitVo vo) {
        SubmitOrderResponseVo responseVo = new SubmitOrderResponseVo();
        //0 代表失败 1代表成功
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end"

        MemberEntityVo memberEntityVo = LoginUserInterceptor.loginUser.get();
        //下单

        //通过 Lua 脚本 原子 验证令牌
        //需要验证的token
        String orderToken = vo.getOrderToken();
        //原token
        String key = OrderConstant.USER_ORDER_TOKEN_PREFIX + memberEntityVo.getId();
        //比较并删除的原子操作 (通过Lua 脚本实现 )
        Long result = redisTemplate.execute(new DefaultRedisScript<Long>(script, Long.class), Collections.singletonList(key), orderToken);

        if(result==0L){
            //验证失败
            return responseVo;
        }



        return null;
    }
```


## 本地事务 
ACID   
- 原子性
- 一致性
- 隔离性
- 持久性

隔离级别   


传播行为  


### 本地事务失效问题  
同一个对象内，事务方法互调默认失效   
如果 a、b、c 方法全都在同一个 service 下，那么 b、c 做的**传播行为配置，都不会起作用**，也就是说b、c都会跟 a 共用一个事务  

使用代理对象调用b、c方法，即可解决  
- 导入 spring-boot-starter-aop 依赖，这个依赖引入了 aspectj 
- 启动类开启 aspectj 动态代理功能，以后所有的动态代理都是 aspectj 创建的（即使没有接口也可以创建动态代理），对外暴露代理对象
- `@EnableAspectJAutoProxy(exposeProxy=true) `
- 用代理对象对本类互调 `AopContext.currentProxy()` 调用方法 

```java
	@Transactional(timeout = 30)
	public void a(){
        // 直接强转，然后b、c的传播行为设置就能起作用了
		OrderServiceImpl orderService = (OrderServiceImpl) AopContext.currentProxy();
        orderService.b();
        orderService.c();
	}

	// REQUIRED：单纯的需要一个事务，如果 a 已经有了，就会直接使用 a 的
	@Transactional(propagation = Propagation.REQUIRED)
	public void b(){

	}
    
	// REQUIRES_NEW：总是需要一个新的事务
	@Transactional(propagation = Propagation.REQUIRES_NEW)
	public void c(){
	
	}
```

## 分布式事务 

> 本地错误无法导致远程服务回滚，最终数据不一致问题  
> `@Transactional` 是本地事务,在分布式系统下只能控制本身回滚,无法控制其他服务回滚   



### CAP 理论  
> CAP 指的是一致性、可用性、分区容错性。  
> CAP 原则指的是这三个要素最多只能同时实现两点(**一致性和可以用性不能同时满足**),不可三者兼顾  
> 在以分布式存系统为限定条件的 CAP 世界里，P 是早已经确定的答案，**P 是必须的**。

> 在分布式系统内，P 是必然的发生的，不选 P，一旦发生分区错误，整个分布式系统就完全无法使用了，这是不符合实际需要的。  
> 所以，对于分布式系统，我们只能能考虑当发生分区错误时，**如何选择一致性和可用性**

[CAP理论中的P到底是个什么意思？ - 四猿外的回答 - 知乎](https://www.zhihu.com/question/54105974/answer/1643846752)

### Raft 算法实现分布式系统一致性

### BASE 理论
> BASE 其实就是对 CAP 的延伸   
> 保证最终一致，这就是我们说的 BASE

- 基本可用（Basically Available）  

> 将部分的消费者流量直接引导到一个错误页面，就是我们说的这个降级页面,我们要保证业务的基本能用。

- 软状态（ Soft State）  

> 软状态就指的是我们系统存在中间状态，不像我们的强一致，要么成要么败，我们也可以有一个中间状态。类似正在同步中这样。

- 最终一致（ Eventual Consistency）

> 最终一致指的就是我们这个系统里边的这些数据。经过一段时间后，最终达到业务状态是一致的


### 分布式事务常见的解决方案    

#### 2PC 模式  
> 2PC 就是我们说的二阶段提交，又叫做 `XA Transactions`。

> MySQL 从 5.5 版本开始支持，SQL Server 2005 开始支持，Oracle 7 开始支持

缺点:   
性能不理想，特别不适用于大型互联网场景高并发的情况下。    
这种情况下，我们这个 XA 根本就没法工作了，因为它要占用大量的锁定资源。   



#### 3PC 模式  


#### 柔性事务 - TCC 事务补偿方案

刚性事务就是我们说的遵循 ACID 这个原则的事务，强一致性的。  

柔性事务就是遵循我们之前说的 BASE 理论，实现最终一致性的。  

#### 柔性事务-最大努力通知方案


#### 柔性事务- 可靠消息+最终一致性方案（异步确保型）


### SEATA  

https://seata.io/zh-cn/docs/user/quickstart.html

1. 每个微服务创建 `undo_log`
2. 安装事务协调器 seata-server 
3. 导入依赖

```xml
<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
        </dependency>
```

4. 解压启动 seata-server  修改 `registry.conf` type="nacos" 
5. `@GlobalTransactional` 标记在 `TM (Transaction Manager) - 事务管理器` 即最初调用个服务得方法上  
6. 所有用到分布式事务得微服务,都需要使用 Seata代理数据源  

```java
import com.zaxxer.hikari.HikariDataSource;
import io.seata.rm.datasource.DataSourceProxy;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.util.StringUtils;

import javax.sql.DataSource;

/**
 * 类描述
 */
@Configuration
public class MySeataConfig {

    @Autowired
    DataSourceProperties dataSourceProperties;

    @Bean
    public DataSource dataSource(DataSourceProperties dataSourceProperties) {

        HikariDataSource dataSource = dataSourceProperties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
        if (StringUtils.hasText(dataSourceProperties.getName())) {
            dataSource.setPoolName(dataSourceProperties.getName());
        }
        return new DataSourceProxy(dataSource);
    }

}
```

7. 每个微服务都需要导入 registry.conf 和 file.conf  


### 最终总结 

> 真正的高并发分布式事务,建议使用柔性事务中的`最大努力通知`或者 `可靠消息+最终一致性方案 `
> 即使用消息队列(延时队列)来完成  

## 秒杀   

### 定时任务 Quartz  

Cron表达式：    
> 语法：秒 分 时 日 月 周 年（可忽略年，Spring 不支持年）  

- `,`：枚举
  - `(cron="7,9,23 * * * * ?")`：代表任意时刻的7，9，23秒启动这个任务； 

- `-`：范围
  - `(cron="7-20 * * * * ?")`：任意时刻的 7-20 秒之间，每秒启动一次 

- `*` ：任意
  - 指定位置的任意时刻都可以 

- `/` ：步长
  - `(cron="7/5 * * * * ?")`：第 7 秒启动，每 5 秒一次；
  - `(cron="*/5 * * * * ?")`：任意时间启动之后，每 5 秒一次； 

- `?` ：（出现在日和周几的位置）为了防止日和周冲突，如果1个精确了，另一个就得写?
  - `(cron="* * * 1 * ?")`：每月的 1 号，启动这个任务，如果两个都写精确值的话，可能会导致冲突，所以其中一个要使用? 

- `L` ：（出现在日和周的位置）”，last：最后一个
  - `(cron="* * * ? * 3L")`：每月的最后一个周二 

- `W` ：  Work Day：工作日
  - `(cron="* * * W * ?")`：每个月的工作日触发
  - `(cron="* * * LW * ?")`：每个月的最后一个工作日触发 

- `#` ： 第几个
- `(cron="* * * ? * 5#2")`：5 代表周 4，#2 代表第 2 个，合起来就是每个月的第 2 个周 4 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/corn时间表达式示例.png)


### Spring Boot 整合 Quartz  


```java
@Slf4j
@Component
@EnableAsync
@EnableScheduling
public class SeckillSchedule {

    @Async
    @Scheduled(cron = "0-5 * * ? * 4-5")
    public void hello(){
        log.info("hello{}",System.currentTimeMillis());
    }
}
```


> 定时任务默认阻塞   
> 采用 `completeableFuture` 异步运行  
> 采用自带 定时任务线程池进行配置 核心线程数   
> 定时任务直接进行异步执行 `@EnableAsync` `@Async`  

```yaml
spring:
  task:
  # 定时任务线程池配置
    scheduling:
      pool:
        size: 5
  # 异步任务线程池配置
    execution:
      pool:
        core-size: 10
```



### 业务
- 随机码  

> 为了防止有用户在得知秒杀请求时，发送大量请求对商品进行秒杀，我们采取了随机码的方式，即每个要参加秒杀的商品，都有一个随机码，只有通过正常提交请求的流程才可以获取，否则谁都无法得知随机码是多少，避免了恶意秒杀  

- 商品的分布式信号量 (redisson)

> 提前在 redis 里边设置一个信号量，这个信号量可以认为是一个自增量，假设这个信号量叫 count，它专门用来计数，它的初始值是 100，每进来一个请求，我们就让这个值减一，如果有用户想要秒杀这个商品，我们先去 redis 里边获取一个信号量,也就是给这一百的库存减一

- 流程  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/秒杀流程.png)


- 幂等性保证

> 通过分布式锁来保证幂等性  


### 秒杀系统设计

- 服务单一职责，独立部署秒杀服务
- 秒杀连接加密，秒杀接口须携带token
- 库存预热，快速扣减，使用定时任务将秒杀信息提前存入redis，使用信号量控制秒杀请求库存衰减  
- 动静分离   
- 恶意请求拦截，通过网关层级拦截   
- 流量错峰，开启验证码校验  
- 限流&熔断&降级，快速失败，熔断，限制请求次数，引导到降级页面   
- 队列削峰

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/秒杀操作流程图.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/秒杀mq消息.png)


## Sentinel 熔断,限流,降级

- 熔断 ,请求时间过长,直接不请求,返回降级数据  
- 限流,请求过多,限制流量
- 降级,停止服务来缓解压力,或者返回降级数据

https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D  

https://sentinelguard.io/zh-cn/docs/quick-start.html


- 引入starter

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

- 开启配置

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719
```

- 引入控制台,单独下载控制jar包运行即可  

- 引入监控 `spring-boot-starter-actuator`(相关详细配置看spring boot 监控)  

- 自定义Sentinel 返回信息 (默认返回:`Blocked by Sentinel (flow limiting)`)

```java
@Component
public class ExceptionHandlerPage implements BlockExceptionHandler {
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, BlockException e) throws Exception {
        R error = R.error(CodeEnum.TO_MANY_REQUEST.getCode(), CodeEnum.TO_MANY_REQUEST.getMsg());
        httpServletResponse.setCharacterEncoding("UTF-8");
        httpServletResponse.setContentType("application/json");
        httpServletResponse.getWriter().write(JSON.toJSONString(error));
    }
}
```

- 流控示例 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sentinel流控.png)

- 熔断降级,保护feign 接口

```yaml
feign:
  sentinel:
    enabled: true
```

```java
/**
 * 自定义feign调用熔断 返回示例
 */
@Component
public class SecKillFallback implements SeckillFeignService {
    @Override
    public R getSkuSeckilInfo(Long skuId) {
        return R.error(CodeEnum.TO_MANY_REQUEST.getCode(),CodeEnum.TO_MANY_REQUEST.getMsg());
    }
}
```

```java
@FeignClient(value = "gulimall-seckill",fallback = SecKillFallback.class)
public interface SeckillFeignService {

    /**
     * 根据skuId查询商品是否参加秒杀活动
     * @param skuId
     * @return
     */
    @GetMapping(value = "/sku/seckill/{skuId}")
    R getSkuSeckilInfo(@PathVariable("skuId") Long skuId);

}

```

- 网关流控  
https://sentinelguard.io/zh-cn/docs/api-gateway-flow-control.html

## 服务链路追踪  

Sleuth Zipkin  


## Kubernetes(K8S) KubeSphere DevOps

### 安装配置 vagrant VirtualBox 
使用 `vagrant` +  `VirtualBox` 创建 集群服务器   
配置服务器的网络环境   

1. 安装 `vagrant VirtualBox` 需要重启
2. 下载vagrant centos 镜像 
   1. https://vagrantcloud.com/centos/boxes/7/versions/2004.01/providers/virtualbox.box
3. 将镜像配置到 vagrant box 中去 `vagrant box add centos/7 D:\CentOS-7-x86_64-Vagrant-1905_01.VirtualBox.box`
4. 根据文件 `Vagrantfile` 安装,在所在目录下 执行 `vagrant up`进行安装 (不用经过2,3 会很慢等待)
5. `vagrant ssh k8s-node1` 访问 node1 服务器 修改 `/etc/ssh/sshd_config` 文件,设置`PasswordAuthentication yes` 开启账号密码访问权限  


``` 
Vagrant.configure("2") do |config|
   (1..3).each do |i|
        config.vm.define "k8s-node#{i}" do |node|
            # 设置虚拟机的Box
            node.vm.box = "centos/7"

            # 设置虚拟机的主机名
            node.vm.hostname="k8s-node#{i}"

            # 设置虚拟机的IP
            node.vm.network "private_network", ip: "192.168.56.#{99+i}", netmask: "255.255.255.0"

            # 设置主机与虚拟机的共享目录
            # node.vm.synced_folder "~/Documents/vagrant/share", "/home/vagrant/share"

            # VirtaulBox相关配置
            node.vm.provider "virtualbox" do |v|
                # 设置虚拟机的名称
                v.name = "k8s-node#{i}"
                # 设置虚拟机的内存大小
                v.memory = 2048
                # 设置虚拟机的CPU个数
                v.cpus = 4
            end
        end
   end
end
```

6. 关闭防火墙 `systemctl stop firewalld`  `systemctl disable firewalld`  
7. 关闭 selinux 校验  `sed -i 's/enforcing/disabled/' /etc/selinux/config`  
8. 关闭swap `sed -ri 's/.*swap.*/#&/' /etc/fstab`
9. 设置host 对应名称

```
10.0.2.15 k8s-node1
10.0.2.5 k8s-node2
10.0.2.4 k8s-node3
```

10. 将桥接的ipv4 流量传递到iptables链 

```
cat > /etc/sysctl.d/k8s.conf<< EOF
net. bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  

```

### 安装 Docker kubeadm kubelet kubectl

卸载docker相关   

```
sudo yum remove docker \ docker-client \ docker-client-latest \ docker-common \ docker-latest \ docker-latest-logrotate \ docker-logrotate \ docker-engine

sudo yum install -y yum-utils \ device-mapper-persistent-data \ lvm2

sudo yum-config-manager \ --add-repo \ https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install -y docker-ce docker-ce-di containerd.io

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'{registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]}EOF
sudo systemctl daemon-reload
sudo systemctl restart docker


cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name= Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled= 1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubemetes/yum/doc/rpm-package-key.gpg
EOF

yum list|grep kube
yum install -y kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3

systemctl enable kubelet
systemctl start kubelet


```

### 部署 k8s master

```
kubeadm init\
--apiserver-advertise-address=10.0.2.15\
--image-repository registry.cn-hangzhou.aliyuncs.com/google_containers\
--kubernetes-version v1.17.3\
--service-cidr= 10.96.0.0/16\
--pod-network-cidr=10.244.0.0/16

kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=1.17.3 --pod-network-cidr=10.244.0.0/16 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.0.2.15


kubeadm join 10.0.2.15:6443 --token ej1typ.vbbhuqx8dacvx3it \
    --discovery-token-ca-cert-hash sha256:efb0f439cd8d9c82c71ff6582363ef53f544dc9952f41fac1cf54a8cb03dacd5


kubectl apply -f kube-flannel.yml  

kubectl get pods

kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8
``` 

乱套了 后来 略


### kubesphere  

https://kubesphere.com.cn/docs/introduction/what-is-kubesphere/










## 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV15a411A7kP)
> - [语雀](https://www.yuque.com/zhangshuaiyin/guli-mall/krt1k5)