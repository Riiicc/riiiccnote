# Spring Boot笔记  

## Doc
https://docs.spring.io/spring-boot/docs/2.6.4-SNAPSHOT/reference/html/

## 自动配置说明

> 使用父项目做依赖管理  


```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

```

- 无需关注版本号,版本自动仲裁
  - 引入依赖默认都不写版本
  - 引入非版本仲裁的jar要写版本号

- 可以修改默认版本号
  - 查看`spring-boot-dependencies` 里面规定当前依赖的版本用的key
  - 再当前项目pom文件重写配置

```xml
    <properties>
        <mysql.version>5.1.43</mysql.version>
    </properties>
```
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot1.png)


- 自动配置实例  

> 默认扫描包路径为主类同包及其下属包  


```java
@SpringBootApplication(scanBasePackages="com.ric")
等同于
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.ric")
```


## 容器  
### 组件添加  

#### @Configuration 

> `@Configuration` 标记的类为配置类 

- 配置类里面使用`@Bean` 标注在方法上作为组件注册,默认为单实例
- 配置类本身也是组件
- 代理Bean方法
    - Full模式 `@Configuration(proxyBeanMethods = false)` 每个@Bean 方法每次调用都是新创建的
    - Lite模式 `@Configuration(proxyBeanMethods = true)` 每个@Bean 调用返回都是同一个实例(单实例,默认)

```java
#############################Configuration使用示例######################################################
/**
 * 1、配置类里面使用@Bean标注在方法上给容器注册组件，默认也是单实例的
 * 2、配置类本身也是组件
 * 3、proxyBeanMethods：代理bean的方法
 *      Full(proxyBeanMethods = true)、【保证每个@Bean方法被调用多少次返回的组件都是单实例的】
 *      Lite(proxyBeanMethods = false)【每个@Bean方法被调用多少次返回的组件都是新创建的】
 *      组件依赖必须使用Full模式默认。其他默认是否Lite模式
 *
 *
 *
 */
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {

    /**
     * Full:外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     * @return
     */
    @Bean //给容器中添加组件。以方法名作为组件的id。返回类型就是组件类型。返回的值，就是组件在容器中的实例
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        //user组件依赖了Pet组件
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }

    @Bean("tom")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}

```
#### @Bean @Component @Controller @Service @Repository  
略
#### @ComponentScan @Import  
> `@Import` 可以在任意类中进行导入
> `@Import({TUser.class, PatternLayoutEncoder.class})`  
> 给容器中自动创建出这两个类的组件


#### @Conditional 条件装配
条件装配 : 满足Conditional指定的条件,则进行组件注入  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot2.png) 

- `ConditionalOnBean` Bean存在时注入
- `ConditionalOnMissingBean` Bean不存在时注入
- `ConditionalOnClass` 类存在时注入  
- `ConditionalOnJava` 指定java版本注入
- `ConditionalOnWebApplication` 是web项目时注入
- ...

```java
@Configuration
//@ConditionalOnBean(name = "tom") 容器中有tom 下面的所有bean才能注册
public class MyConfig {
    @Bean("tom")
    public Pet pet1(){
        Pet pet = new Pet();
        pet.setName("tom");
        return pet;
    }


    @ConditionalOnBean(name = "tom") //注意tom必须在前面,在后面也不会生效
    @Bean
    public TUser user1(){
        return new TUser();
    }


}
```
### 原生配置文件引入@ImportResource
`@ImportResource("classpath:beans.xml")`  

注:springbootweb 项目默认 `classpath` 为 `resources` 包下


### 配置绑定 @ConfigurationProperties 

```properties
mycar.brand = BYD
mycar.price = 100000
```

```java
/**
 * 只有在容器中的组件，才会拥有SpringBoot提供的强大功能
 */
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {

    private String brand;
    private Integer price;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public Integer getPrice() {
        return price;
    }

    public void setPrice(Integer price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", price=" + price +
                '}';
    }
}
```
访问car接口可以直接返回`Car{brand='BYD', price='100000'}`
```java
@ResponseBody
@Controller
public class HelloController {

    @Autowired
    private Car car;


    @RequestMapping("car")
    public String getCar(){

        return car.toString();
    }
}
```

> 或者可以直接在配置类中使用注解 `@EnableConfigurationProperties(Car.class)` 可以省略Car中的 `@Component` 

上面的两种组合分别是
- 配置类 `@EnableConfigurationProperties` + 目标类 `@ConfigurationProperties`
- 目标类 `@Component` + 目标类 `@ConfigurationProperties`


## 自动配置原理

```java

@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{}


    
```
`@Configuration` 代表当前类是一个配置类    

`@ComponentScan`执行扫描包
### @EnableAutoConfiguration  
```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```
#### @AutoConfigurationPackage 
利用 `Registrar` 给容器中导入多个组件(`Application` 所在包下的所有组件)   

```java
//给容器导入一个组件
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};
}
```

#### @Import({AutoConfigurationImportSelector.class}) 

> 利用 `getAutoConfigurationEntry(AnnotationMetadata annotationMetadata)` 给容器中批量导入组件

> ` List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes)` 调用这个方法获取到所有需要导入到容器中的组件  

> 利用工厂加载 `Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)`得到所有的组件

> 通过 `META-INF/spring.factories` 位置来加载一个文件,默认扫描当前系统所有META-INF/spring.factories 位置的文件
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot3.png)

`spring-boot-autoconfigure-2.6.3.jar` 中的META-INF/spring.factories  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot4.png)

### 按需加载

> 虽然我们127个场景的所有自动配置启动的时候默认全部加载。xxxxAutoConfiguration按照条件装配规则（@Conditional），最终会按需配置。 


如: aop 只有存在`Advice.class`类存在才会自动配置
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot5.png)


### 修改默认配置

```java
        //2.3版本
        @Bean
		@ConditionalOnBean(MultipartResolver.class)  //容器中有这个类型组件
		@ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME) //容器中没有这个名字 multipartResolver 的组件
		public MultipartResolver multipartResolver(MultipartResolver resolver) {
            //给@Bean标注的方法传入了对象参数，这个参数的值就会从容器中找。
            //SpringMVC multipartResolver。防止有些用户配置的文件上传解析器不符合规范
			// Detect if the user has created a MultipartResolver but named it incorrectly
			return resolver;
		}
        // 给容器中加入了文件上传解析器；

        //2.6版本 简化了
        @Bean(name = {"multipartResolver"})
        @ConditionalOnMissingBean({MultipartResolver.class})
        public StandardServletMultipartResolver multipartResolver() {
            StandardServletMultipartResolver multipartResolver = new StandardServletMultipartResolver();
            multipartResolver.setResolveLazily(this.multipartProperties.isResolveLazily());
            return multipartResolver;
        }



```
**SpringBoot默认会在底层配好所有的组件。但是如果用户自己配置了以用户的优先**

### 总结
- Spring Boot 先加载所有的自动配置类 xxxxAutoConfiguration
- 每个配置类按照其内部条件生效,默认都会绑定配置文件中指定的值(xxxxxAutoConfiguration --->  xxxxProperties(@ConfigurationProperties)  ----> application.properties)
- 生效的配置类会给容器中装配很多组件
- 定制化配置
  - 用户直接自定义替换组件
  - 用户修改组件对应的配置(配置文件中的配置项)
自定义替换组件示例:
```java
        //写入配置类中
        @Bean(name = {"multipartResolver"})
        public StandardServletMultipartResolver multipartResolver() {
            StandardServletMultipartResolver multipartResolver = new StandardServletMultipartResolver();
            return multipartResolver;
        }
```

### 场景依赖
-  引入场景依赖,各种stater
   -    https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter
- 查看自动配置项
  - 通过配置文件`debug = true` 开启自动配置报告,分为两类 Negative 不生效 和Positive 生效  
- 修改配置项 
  - 参考文档修改配置项
  - https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties 
  - 自定义加入或者重写替换组件 
  - 自定义器xxxCustomizer(未知)


## 开发适用
### Lombok  
- `@Setter`
- `@Getter`
- `@NoArgsConstructor`
- `@AllArgsConstructor`
- `@ToString(exclude = "")` 
- `@EqualsAndHashCode`
- `@RequiredArgsConstructor`final 参数构造器

- `@Data` 等同于下面的组合
  - `@Getter/@Setter`
  - `@ToString`
  - `@EqualsAndHashCode`
  - `@RequiredArgsConstructor`  

- `@Value` 等同于下面的组合   
  - `@Getter (注意没有setter)`
  - `@ToString`
  - `@EqualsAndHashCode`
  - `@RequiredArgsConstructor`
>  @Value 注解，是适合加在值不希望被改变的类上，像是某个类的值当创建后就不希望被更改，只希望我们读它而已，就适合加上 @Value 注解

- `@Builder/@SuperBuilder` SuperBuilder 支持构造父类成员属性 
- `@Accessors(chain = true)` 生成setter方法返回this（也就是返回的是对象），代替了默认的返回void `User user=new User().setAge(31).setName("zhang");`
- `@Accessors(fluent = true)` 与chain=true类似，区别在于getter和setter不带set和get前缀。`Pet pet = new Pet().name("11"); String name = pet.name();`


- `@Slf4j` 日志打印 

### dev-tools
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```

## 配置文件  

### YAML 基本语法 
> YAML 是 "YAML Ain't Markup Language"（YAML 不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）

- key: value 注意空格
- 大小写敏感
- 使用缩进来表示层级关系
- 缩进允许使用tab,只允许空格
- 缩进的空格数不重要,只要相同层级的元素对其即可
- `#` 表示注释
- 字符串无需加引号,如果要加，''与""表示字符串内容 会被 转义(单引号)/不转义(双引号) 
- `'zhangsan \n 李四'` 不换行 , `"zhangsan \n 李四"` 换行 ,不想被特殊符号影响使用单引号

```yaml
k: v

行内写法：  k: {k1:v1,k2:v2,k3:v3}
#或
k: 
  k1: v1
  k2: v2
  k3: v3

行内写法：  k: [v1,v2,v3]
#或者
k:
 - v1
 - v2
 - v3
```

### 示例
```java
@Component
@ConfigurationProperties(prefix = "person")
@Data
public class Person {
	
	private String userName;
	private Boolean boss;
	private Date birth;
	private Integer age;
	private Pet pet;
	private String[] interests;
	private List<String> animal;
	private Map<String, Object> score;
	private Set<Double> salarys;
	private Map<String, List<Pet>> allPets;
}

@Data
public class Pet {
	private String name;
	private Double weight;
}
```

```yaml
# yaml表示以上对象
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
#   interests:
#     - 篮球
#     - 足球
  animal: 
    - jerry
    - mario 
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]
```
### 配置提示
> 自定义的类和配置文件绑定一般没有提示。添加下面的依赖重启后可以显示提示


```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

<!-- 开发环境无用,打包时将该jar包排除,提高性能 -->
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-configuration-processor</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

```


## WEB开发

### 静态资源访问

[Spring文档对应](https://docs.spring.io/spring-boot/docs/2.3.12.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content)

静态资源放在下面的路径中默认可以通过`当前项目根路径/静态资源名`访问：
- `/staitc`
- `/public`
- `/resources`
- `/META-INF/resources` 

> 原理: 静态资源映射默认为`/**` 
> 请求进入后先由`Controller` 看是否能处理,不能处理则交由静态资源处理器,静态资源也找不到就会返回404,
> 也就是说如果`Controller`中定义了和静态资源同名的映射,那么就会以`Controller`为准

```java
@ResponseBody
@Controller
public class HelloController {
    //此时访问http://localhost:8080/111.txt 会返回123
    @RequestMapping("/111.txt")
    public String hello(){

        return "123";
    }
}
```
改变默认的静态资源路径(**访问路径**和**实际路径**)
- 访问路径前添加`res`前缀
- 将静态资源路径放在`newpath,newpath1`文件夹下,多个目录逗号隔开   
- 注意第三个2.6版本改变配置属性
```yml
# 访问静态资源路径前多个 res 
spring:
  mvc:
    static-path-pattern: /res/**
# 实际路径 2.3版本
  resources:
    static-locations: classpath:/newpath,classpath:/newpath1
# 新版本改成如下
  web:
    resources:
      static-locations: classpath:/newpath,classpath:/newpath1
```

### WEBJars

通过引入对应jar包,以静态文件形式引入js等相关文件
```xml
<!-- 引入jquery -->
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.5.1</version>
        </dependency>
```
> 通过 http://localhost:8080/webjars/jquery/3.5.1/jquery.js 访问js
> 后面地址要按照依赖里面的包路径

### 欢迎页

- 静态资源下的index.html
- controller请求中的 `/index` 

```yml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致welcome page功能失效

  resources:
    static-locations: [classpath:/haha/]
```

### 自定义 Favicon
> favicon.ico 放在静态资源目录下即可。 

```yml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致 Favicon 功能失效
```

### 静态资源配置原理 

- SpringBoot启动默认加载xxxAutoConfiguration类 
- SpringMVC功能的自动配置类`WebMvcAutoConfiguration`  

```java

@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
```

#### 资源默认处理规则参考 

```java
        @Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			//webjars的规则
            if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
            
            //
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}

```
禁用静态资源
```yml
spring:
#  mvc:
#    static-path-pattern: /res/**

  resources:
    add-mappings: false   禁用所有静态资源规则
```
默认静态资源位置配置
```java

@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```
默认欢迎页配置

```java
	HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。	

	@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}

	WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
			ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
		if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
            //要用欢迎页功能，必须是/**
			logger.info("Adding welcome page: " + welcomePage.get());
			setRootViewName("forward:index.html");
		}
		else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
            // 调用Controller  /index
			logger.info("Adding welcome page template: index");
			setRootViewName("index");
		}
	}

```
### 请求参数处理  

#### Rest使用及原理  
- @xxxMapping
- REST风格支持(使用HTTP请求方式动词来表示对资源的操作)

对比项| 获取用户 | 删除用户| 修改用户| 保存用户
-----|----------|---------|---------|---------
 **以前** | `/getUser` | `deleteUser`| `/editUser`| `/saveUser` |
 **现在** | `/user` `GET` | `/user` `DELETE`| `/user` `PUT`| `/user` `POST`|  

#### form表单提交REST请求
- 核心Filter `HiddenHttpMethodFilter`
  - 用法: 表单method = post,隐藏域 _method=put,通过隐藏域支持其他REST请求
  - SpringBoot中手动开启
  - 通过自定义Filter 修改 _method 属性(隐藏域属性)
  - 请求被`HiddenHttpMethodFilter` 拦截

```yml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true # 开启页面表单的REST请求功能 可以使用表单发送除get/post之外的请求
```

```java
    @RequestMapping(value = "/user",method = RequestMethod.GET)
    public String getUser(){
        return "GET-张三";
    }

    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public String saveUser(){
        return "POST-张三";
    }


    @RequestMapping(value = "/user",method = RequestMethod.PUT)
    public String putUser(){
        return "PUT-张三";
    }

    @RequestMapping(value = "/user",method = RequestMethod.DELETE)
    public String deleteUser(){
        return "DELETE-张三";
    }
```
```java
/* 下面是源码 */
	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
	@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
		return new OrderedHiddenHttpMethodFilter();
	}


//自定义filter 将_method 改为_m 
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }
```
**form表单REST请求参考**
```html
<form action="/user" method="get">
    <input value="get提交" type="submit">
</form>
<form action="/user" method="post">
    <input value="post 提交" type="submit">
</form>
<form action="/user" method="post">
    <input name="_method" value="put" type="hidden">
    <input value="put 提交" type="submit">
</form>
<form action="/user" method="post">
    <input name="_method" value="delete" type="hidden">
    <input value="delete 提交" type="submit">
</form>
```

> 下图中可以看到默认为`_method` 并进行判断 和重写请求
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot6.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot7.png)

### 请求映射原理

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot8.png)

`HttpServletBean`(doGet)-> `FrameworkServlet` (prcessRequest>>doService) -> `DispatcherServlet`(doDispatch)
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// 找到当前请求使用哪个Handler（Controller的方法）处理
				mappedHandler = getHandler(processedRequest);
                
                //HandlerMapping：处理器映射。/xxx->>xxxx
```
RequestMappingHandlerMapping：保存了所有@RequestMapping 和handler的映射规则。
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot9.png)  

> 所有的请求映射都在HandlerMapping中。
> SpringBoot自动配置欢迎页的WelcomePageHandlerMapping,默认访问到index.html
> 请求进来,遍历尝试所有的HandlerMapping看是否存在请求信息
> 如果有请求信息就找到对应的Handler
> 如果没有寻找下一个HandlerMapping

```java
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```

### 普通参数与基本注解   

- `@PathVariable`
- `@RequestHeader`
- `@RequestParam`
- `@CookieValue`
- `@ModelAttribute`
- `@MatrixVariable`
- `@RequestBody`

> `@PathVariable` 获取路径中的值 如:`/car/{id}` 
> 同时支持多个变量值 如:`/car/{id}/owner/{username}` 可以使用`@PathVariable("id") Integer id,@PathVariable("username") String name`
> 也可以使用`@PathVariable Map<String,String> pv,` 获取所有的值的集合,必须为`Map<String,String>` 类型

> `@RequestHeader` 获取请求头信息 如:`@RequestHeader("User-Agent") String userAgent,`
> 同样,支持`多变量值`和`Map集合`获取 

> `@RequestParam` 获取请求参数如:`/car?id=18&name=zhangsn`
> 同样,支持`多变量值`和`Map集合`获取 

> `@CookieValue` 获取Cookie中的值 如:`@CookieValue("_ga") String _ga`

#### 矩阵变量@MatrixVariable
`@MatrixVariable` 
- SpringBoot默认禁用矩阵变量  
- 语法请求路径： `/cars/sell;low=34;brand=byd,audi,yd` 或者`/cars/sell;low=34;brand=byd,brand=audi,brand=yd` 注意两种`kv`形式
- 分号隔开的

> 手动开启,对于路径的处理都是使用`UrlPathHelper`

> 应用场景 cookie禁用,如何使用session中的内容(原来的sessionid存在cookie中)
> 可以使用矩阵变量携带对应的sessionid
> `/abc;jsessionid=xxxx`  

- 开启(允许)矩阵变量   


```java
//方式一
@Configuration(proxyBeanMethods = false)
public class MyConfig implements WebMvcConfigurer {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        //配置自定义的UrlPathHelper 不移除分号后面的内容， 矩阵变量功能就可以生效
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);

    }
}

//方式二
@Configuration(proxyBeanMethods = false)
public class NewConfig {
    
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
                
            }
        };
    }
}

```


**多层矩阵变量拼接**
```java
    // /boss/1;age=20/2;age=10 
    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge
                    @PathVariable("bossId" Integer bossId)){
        Map<String,Object> map = new HashMap<>();

        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        //同样可以获取 bossid
        map.put("bossId",bossId);
        return map;

    }
```

### 参数处理原理
没看
### 数据相应和内容协商
没看

## 视图解析与模板引擎 

### 视图解析原理
> 视图解析：SpringBoot默认不支持 JSP，需要引入第三方模板引擎技术实现页面渲染。 
没懂这块 

#### Thymeleaf 配置
> Thymeleaf 也有自动配置类 `ThymeleafAutoConfiguration` 来自动配置相关默认项  

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration {}
```
- 所有配置都在 `ThymeleafProperties`
- 配置了 `SpringTemplateEngine` 
- 配置了 `ThymeleafViewResolver` 视图解析器

``` java
//配置项
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";

```
- 模板抽取 `th:insert/replace/include`

## 拦截器

### HandlerInterceptor 接口 

```java
/**
 * 登录检查
 * 1、配置好拦截器要拦截哪些请求
 * 2、把这些配置放在容器中
 */
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {

    /**
     * 目标方法执行之前
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("preHandle拦截的请求路径是{}",requestURI);

        //登录检查逻辑
        HttpSession session = request.getSession();

        Object loginUser = session.getAttribute("loginUser");

        if(loginUser != null){
            //放行
            return true;
        }

        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录");
//        re.sendRedirect("/");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }

    /**
     * 目标方法执行完成以后
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle执行{}",modelAndView);
    }

    /**
     * 页面渲染以后
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常{}",ex);
    }
}
```
### 配置拦截器

```java
/**
 * 1、编写一个拦截器实现HandlerInterceptor接口
 * 2、拦截器注册到容器中（实现WebMvcConfigurer的addInterceptors）
 * 3、指定拦截规则【如果是拦截所有，静态资源也会被拦截】
 */
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  //所有请求都被拦截包括静态资源
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**"); //放行的请求
    }
}
```

### 拦截器原理
**下面的是执行顺序**

- 根据当前请求,找到 `HandlerExecutionChain` 可以处理请求的handler 和 handler 的所有拦截器
- 顺序执行所有拦截器的 `preHandler` 方法  (所有拦截器的for循环)
  - 如果当前拦截器 `preHandler` 返回为true ,就继续执行下一个拦截器的 `preHandler` 
  - 如果当前拦截器 `preHandler` 返回 false ,直接倒序执行所有已经执行了 `preHandler` 方法的拦截器的 `afterCompletion` 方法(未执行 `pre` 的拦截器不会执行) 
- 如果任何一个拦截器执行失败,就会直接跳出不执行目标方法(实际调用的方法) 
- 所有拦截器都返回true(运行正常),就会执行目标方法
- 倒叙执行所有拦截器的 `postHandler`方法 (for循环)
- 前面任何异常都会倒序触发 `afterCompletion` 
- 如果方法全部执行完毕 也会倒序触发`afterCompletion`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot10.png) 

## 文件上传 

### 文件上传代码
```html
<form method="post" action="/upload" enctype="multipart/form-data">
    <input type="text" name="email">
    <input type="text" name="username">

    <input type="file" name="headerImg">
    <!-- multiple 支持多文件上传 -->
    <input type="file" name="photos" multiple>
    <input type="submit" value="提交">
</form>
```


```java
    /**
     * MultipartFile 自动封装上传过来的文件
     * @param email
     * @param username
     * @param headerImg
     * @param photos
     * @return
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {

        log.info("上传的信息：email={}，username={}，headerImg={}，photos={}",
                email,username,headerImg.getSize(),photos.length);

        if(!headerImg.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = headerImg.getOriginalFilename();
            headerImg.transferTo(new File("H:\\cache\\"+originalFilename));
        }

        if(photos.length > 0){
            for (MultipartFile photo : photos) {
                if(!photo.isEmpty()){
                    String originalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("H:\\cache\\"+originalFilename));
                }
            }
        }


        return "main";
    }

```
**配置上传**
```properties
# 单个文件大小
spring.servlet.multipart.max-file-size=10MB 
# 总大小
spring.servlet.multipart.max-request-size=100MB

```
### 文件上传原理 

> 文件上传自动配置类 `MultipartAutoConfiguration-MultipartProperties` 
> 自动配置了 `StandardServletMultipartResolver`


```java
    @Bean(
        name = {"multipartResolver"}
    )
    @ConditionalOnMissingBean({MultipartResolver.class})
    public StandardServletMultipartResolver multipartResolver() {
        StandardServletMultipartResolver multipartResolver = new StandardServletMultipartResolver();
        multipartResolver.setResolveLazily(this.multipartProperties.isResolveLazily());
        return multipartResolver;
    }
```
**上传原理步骤**
- 请求进来使用文件上传解析器(`StandardServletMultipartResolver.isMultipart`)判断并封装`resolveMultipart`返回(`MultipartHttpServletRequest`)文件上传请求
- 参数解析器来解析请求中的文件内容封装成 `MultipartFile` 
- 将request中文件信息封装为一个`Map；MultiValueMap<String, MultipartFile>FileCopyUtils`。实现文件流的拷贝


## 异常处理  

### 默认规则
- 默认情况下，Spring Boot提供`/error`处理所有错误的映射
- 对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。对于浏览器客户端，响应一个`whitelabel`错误视图，以HTML格式呈现相同的数据
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot11.png)
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot12.png)

- 要对其进行自定义，添加View解析为error
- 要完全替换默认行为，可以实现 ErrorController 并注册该类型的Bean定义，或添加`ErrorAttributes`类型的组件以使用现有机制但替换其内容。
- error/下的4xx，5xx页面会被自动解析,模糊匹配会匹配4开头的和5开头的

### 自动配置异常原理
- `ErrorMvcAutoConfiguration` 自动配置异常处理规则
  - 容器中的组件：类型 `DefaultErrorAttributes errorAttributes`
  - `public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver`
- 容器中的组件：类型：`BasicErrorController --> id：basicErrorController`（json+白页 适配响应）
  - 处理默认 /error 路径的请求；页面响应 new ModelAndView("error", model)；
  - 容器中放组件 BeanNameViewResolver（视图解析器）；按照返回的视图名作为组件的id去容器中找View对象 


- DefaultErrorAttributes：定义错误页面中可以包含哪些数据
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot13.png)


- 容器中的组件：类型：DefaultErrorViewResolver -> id：conventionErrorViewResolver
  - 如果发生错误，会以HTTP的状态码 作为视图页地址（viewName），找到真正的页面


####  定制错误处理逻辑
-  自定义错误页
   - ` error/404.html`,`error/500.html` 有精确的错误码页面就能精确匹配,没有找到对应页就会返回白页 

- `@ControllerAdvice + @ ExceptionHandler` 处理全局异常;底层是`ExceptionHandlerExceptionResolver` 支持
```java
@ControllerAdvice
public class ExceptionHandlerTest {

    @ExceptionHandler({NullPointerException.class,ArrayIndexOutOfBoundsException.class})
    public String handlerExceptionTest(Exception e){

        //支持直接返回视图
        return "login";
    }
}
```

- `@ResponseStatus + 自定义异常` 底层是 ResponseStatusExceptionResolver 

```java
@ResponseStatus(value = HttpStatus.FORBIDDEN,reason = "用户数量太多")
public class UserTooManyException extends RuntimeException {
    public UserTooManyException(String message) {
        super(message);
    }
}
```

- `Spring 底层的异常 如参数类型转化异常` DefalutHandlerExceptionResolver 处理底层框架的异常
- 自定义实现 `HandlerExceptionResolver` 处理异常,可以作为默认的全局异常处理规则 

```java
@Order(value = Ordered.HIGHEST_PRECEDENCE)
@Component
public class CustomHandlerExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler,
                                         Exception ex) {
        ModelAndView modelAndView = new ModelAndView();
        try {
            response.sendError(511,"12313");
        } catch (IOException e) {
            e.printStackTrace();
        }
        return modelAndView;
    }
}
```


## 数据访问

### 导入JDBC场景 Mysql驱动

 ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>

        <!-- 或者通过下面的方式指定版本 -->

    <properties>
        <java.version>1.8</java.version>
        <mysql.version>5.1.49</mysql.version>
    </properties>
 ```
### 分析自动配置 
**自动配置类**
- `DataSourceAutoConfiguration `  数据源的自动配置
  - 修改数据源的相关配置项 `spring.datasource`
  - 如果容器中没有 `DataSource` 就会进行自动配置
  - 自动配置的连接池是 `HikariDataSource`

```java
	@Configuration(proxyBeanMethods = false)
	@Conditional(PooledDataSourceCondition.class)
	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
	@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
			DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.OracleUcp.class,
			DataSourceConfiguration.Generic.class, DataSourceJmxConfiguration.class })
	protected static class PooledDataSourceConfiguration
```
- `DataSourceTransactionManagerAutoConfiguration` 事务管理器的自动配置
- `JdbcTemplateAutoConfiguration` JdbcTemplate 的自动配置, 可以来对数据库进行操作
  - 可以通过修改配置项  `@ConfigurationProperties(prefix = "spring.jdbc")` 来修改JdbcTemplate

```yml
# 设置请求超时时间
spring:
  jdbc:
    template:
      query-timeout: 4
```

### Durid 配置

**Config文件配置**
```java
@Configuration
public class DataSourceConifg {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource dataSource() throws SQLException {
        DruidDataSource druidDataSource = new DruidDataSource();
        //加入监控功能
        druidDataSource.setFilters("stat,wall");
        return druidDataSource;
    }

    /**
     * 配置druid 的监控功能
     * @return
     */
    @Bean
    public ServletRegistrationBean statViewServlet(){
        StatViewServlet statViewServlet = new StatViewServlet();

        ServletRegistrationBean<StatViewServlet> registrationBean =
                new ServletRegistrationBean<>(statViewServlet,"/druid/*");
        //druid 监控页配置账号密码
        registrationBean.addInitParameter("loginUsername","admin");
        registrationBean.addInitParameter("loginPassword","123");

        return registrationBean;

    }

    /**
     * webstatFilter 用于采集web-jdbc关联监控的数据
     * @return
     */
    @Bean
    public FilterRegistrationBean webStatFilter(){
        WebStatFilter webStatFilter = new WebStatFilter();
        FilterRegistrationBean<WebStatFilter> filterFilterRegistrationBean = new FilterRegistrationBean<>(webStatFilter);
        //添加拦截规则
        filterFilterRegistrationBean.setUrlPatterns(Arrays.asList("/*"));
        //排除不需要的请求
        filterFilterRegistrationBean.addInitParameter("exclusions","*.js,*.html,*.css");
        return filterFilterRegistrationBean;
    }
}

```
**yml配置**

```yml
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
SpringBoot配置示例 
https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter

引入stater 依赖

```xml
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
```
### Mybatis
**Stater**
```xml
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
```

- 导入官方stater
- 编写mapper接口 使用 `@Mapper` 注解标注
- 编写sql映射xml文件并绑定Mapper接口
- 在application.yaml中指定Mapper配置文件的位置，以及指定全局配置文件的信息 （建议；配置在mybatis.configuration）


最佳实践
- 引入mybatis-starter
- 配置yml,指定mapper-location位置即可
- `@MapperScan("com.atguigu.admin.mapper")` 简化，其他的接口就可以不用标注@Mapper注解

### NoSql Redis 
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

```

- `RedisAutoConfiguration` 自动配置类。 `RedisProperties` 属性类 --> `spring.redis.xxx`是对redis的配置
- 连接工厂是准备好的。`LettuceConnectionConfiguration`(新版)、`JedisConnectionConfiguration`
- 自动注入了`RedisTemplate<Object, Object>` ： xxxTemplate；
- 自动注入了`StringRedisTemplate`；k：v都是String 上面的string版

**使用自动配置的Template操作redis,默认使用`Lettuce`**
```java
    @Autowired
    StringRedisTemplate stringRedisTemplate;
    @Test
    void redisTest() {
        List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from t_user");
        ValueOperations<String, String> ss = stringRedisTemplate.opsForValue();
        ss.set("1111","2222");
    }
```

测试功能 使用redis 做接口访问次数统计,通过拦截器实现

拦截器
```java
@Component
public class RedisCountInterceptor implements HandlerInterceptor {
    @Autowired
    StringRedisTemplate redisTemplate;
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        StringBuffer requestURL = request.getRequestURL();

        redisTemplate.opsForValue().increment(requestURL.toString());

        return true;
    }
}

```

拦截器注册配置
```java
@Configuration(proxyBeanMethods = false)
public class MyConfig implements WebMvcConfigurer {
    @Autowired
    RedisCountInterceptor interceptor;
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(interceptor);
    }
}
```
## 单元测试 

### JUnit5 
> Spring Boot 2.2.0 版本开始引入 JUnit 5 作为单元测试默认库  

> `JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage`

> **JUnit Platform**: Junit Platform是在JVM上启动测试框架的基础，不仅支持Junit自制的测试引擎，其他测试引擎也都可以接入。
> **JUnit Jupiter**: JUnit Jupiter提供了JUnit5的新的编程模型，是JUnit5新特性的核心。内部 包含了一个测试引擎，用于在Junit Platform上运行。
> **JUnit Vintage**: 由于JUint已经发展多年，为了照顾老的项目，**JUnit Vintage提供了兼容JUnit4.x,Junit3.x的测试引擎**。

> SpringBoot 2.4 以上版本移除了默认对 Vintage 的依赖。如果需要兼容junit4需要自行引入（不能使用junit4的功能 @Test）,
> JUnit 5’s Vintage Engine Removed from spring-boot-starter-test,如果需要继续兼容junit4需要自行引入vintage

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

### 使用JUnit5 
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

```java
@SpringBootTest
class Boot05WebAdminApplicationTests {


    @Test
    void contextLoads() {

    }
}


```

### JUnit5常用注解

JUnit5的注解与JUnit4的注解有所变化
https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations

- `@DisplayName` 为测试类或者测试方法设置展示名称
- `@BeforeEach` 表示在每个单元测试前执行
- `@AfterEach` 每个单元测试后执行
- `@BeforeAll` 所有单元测试前执行
- `@AfterAll` 所有单元测试后执行
- `@Tag` 表示单元测试类别
- `@Disable` 表示测试类或者测试方法不执行
- `@Timeout` 测试方法运行超时报错 默认秒,可以指定单位 
- `@ExtendWith` 为测试类或者测试方法提供扩展类引用
- `@RepeatedTest` 指定重复执行次数 ,重复执行测试方法


### 断言 
> 测试方法核心部分,用来测试需要满足的条件进行验证,这些断言方法都是`org.junit.jupiter.api.Assertions`的静态方法,



方法 | 说明 | 
---------|----------|
 `assertEquals` | 判断两个对象或两个原始类型是否相等 | 
 `assertNotEquals` | 判断两个对象或两个原始类型是否不相等 |
 `assertSame` | 判断两个对象引用是否指向同一个对象 |
 `assertNotSame` | 判断两个对象引用是否指向不同的对象 |
 `assertTrue` | 判断给定的布尔值是否为 true |
 `assertFalse` | 判断给定的布尔值是否为 false |
 `assertNull` | 判断给定的对象引用是否为 nul |
 `assertNotNull` | 判断给定的对象引用是否不为 null |
 `assertArrayEquals` | 判断两个对象或原始类型的数组是否相等 |

> `assertAll` 方法接受多个 org.junit.jupiter.api.Executable 函数式接口的实例作为要验证的断言，可以通过 lambda 表达式很容易的提供这些断言

```java
    //组合断言 两个断言必须同时成功,
    @Test
    @DisplayName("assert all")
    public void all() {
        Assertions.assertAll("Math",
                () -> Assertions.assertEquals(2, 1 + 1),
                () -> Assertions.assertTrue(1 > 0)
        );
    }
```

> `assertThrows` 异常断言,判断一定抛出异常, 配合函数式编程就可以进行使用

```java
@Test
@DisplayName("异常测试")
public void exceptionTest() {
    ArithmeticException exception = Assertions.assertThrows(
           //扔出断言异常
            ArithmeticException.class, () -> System.out.println(1 % 0));

}
```

> `Assertions.assertTimeout()`为测试方法设置了超时时间

```java

@Test
@DisplayName("超时测试")
public void timeoutTest() {
    //如果测试方法时间超过1s将会异常
    Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(500));
}
```

> **快速失败** 通过 fail 方法直接使得测试失败

###  前置条件（assumptions）
> JUnit5 中的前置条件 ,类似于断言, 不同之处在于不满足的断言会使方法失败,而不满足的前置条件只会使测试方法的执行终止,
> 前置条件可以看成是测试方法执行的前提,当前提不满足时,就没有继续执行的必要


```java
@DisplayName("前置条件")
public class AssumptionsTest {
 private final String environment = "DEV";
 
 @Test
 @DisplayName("simple")
 public void simpleAssume() {
    assumeTrue(Objects.equals(this.environment, "DEV"));
    assumeFalse(() -> Objects.equals(this.environment, "PROD"));
 }
 
 //满足条件然后执行其他
 @Test
 @DisplayName("assume then do")
 public void assumeThenDo() {
    assumingThat(
       Objects.equals(this.environment, "DEV"),
       () -> System.out.println("In DEV")
    );
 }
}
```

### 嵌套测试

> `JUnit 5` 可以通过 Java 中的内部类和`@Nested` 注解实现嵌套测试，从而可以更好的把相关的测试方法组织在一起。
> 在内部类中可以使用`@BeforeEach` 和`@AfterEach` 注解，而且嵌套的层次没有限制
> 嵌套测试中，外层的`@Test` 方法不能驱动内层的方法`Before/after/each/all` 之类的方法

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

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```




### 参数化测试
https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests

- `@ParameterizedTest` 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型
- `@NullSource`: 表示为参数化测试提供一个null的入参
- `@EnumSource`: 表示为参数化测试提供一个枚举入参
- `@CsvFileSource`：表示读取指定CSV文件内容作为参数化测试入参
- `@MethodSource`：表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流)


```java
@SpringBootTest
public class CanShu {
    /* 参数化测试的两种方法示例  */

    @ParameterizedTest
    @ValueSource(ints = {1,2,3})
    void intTest(int i){
        System.out.println(i); //1,2,3
    }

    @ParameterizedTest
    @MethodSource("stringProvider")
    void methodTest(String i){
        System.out.println(i);//123,333

    }

    static Stream<String> stringProvider(){
        return Stream.of("123","333");
    }
}

```

### JUnit4 JUnit5 迁移区别
- 注解在 `org.junit.jupiter.api` 包中，断言在 `org.junit.jupiter.api.Assertions` 类中，前置条件在 `org.junit.jupiter.api.Assumptions` 类中
- 把`@Before` 和`@After` 替换成`@BeforeEach` 和`@AfterEach`。
- 把`@BeforeClass` 和`@AfterClass` 替换成`@BeforeAll` 和`@AfterAll`
- 把`@Ignore` 替换成`@Disabled`
- 把`@Category` 替换成`@Tag`
- 把`@RunWith、@Rule` 和`@ClassRule` 替换成`@ExtendWith`


## 指标监控 SringBoot Actuator  
https://docs.spring.io/spring-boot/docs/2.3.12.RELEASE/reference/html/production-ready-features.html#production-ready

### 使用

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

- 引入依赖进行配置 
- 访问地址 `http://localhost:8080/actuator/**`

```yml
management:
  endpoints:
    enabled-by-default: true #暴露所有端点信息
    web:
      exposure:
        include: '*'  #以web方式暴露
```

### Actuator Endpoint

- 最常用的Endpoint
  - `Health` 监控状况
  - `Metrics` 运行时指标
  - `Loggers` 日志记录

#### Health Endpoint
http://localhost:8080/health  

> 健康检查端点，我们一般用于在云平台，平台会定时的检查应用的健康状况，我们就需要Health Endpoint可以为平台返回当前应用的一系列组件健康状况的集合。

- health endpoint返回的结果，应该是一系列健康检查后的一个汇总报告
- 很多的健康检查默认已经自动配置好了，比如：数据库、redis等
- 可以很容易的添加自定义的健康检查机制

#### Metrics Endpoint
http://localhost:8080/actuator/metrics  
提供详细的、层级的、空间指标信息，这些信息可以被pull（主动推送）或者push（被动获取）方式得到；
- 通过Metrics对接多种监控系统
- 简化核心Metrics开发

#### 管理Endpoints

- 默认所有的Endpoint除shutdown都是开启的
- 需要开启或者禁用某个Endpoint。配置模式为 ` management.endpoint.<endpointName>.enabled = true`

```yml
management:
  endpoint:
    beans:
      enabled: true
```

- 或者禁用所有的Endpoint然后手动开启指定的Endpoint

```yml
management:
  endpoints:
    enabled-by-default: false
  endpoint:
    beans:
      enabled: true
    health:
      enabled: true
```
#### 暴露Endpoints

支持的暴露方式:
- **HTTP**：默认只暴露health和info Endpoint
- JMX：默认暴露所有Endpoint (通过命令`jconsole` 调出java控制台访问)
- 除过 `health` 和 `info` ，剩下的Endpoint都应该进行保护访问。如果引入`SpringSecurity`，则会默认配置安全访问规则

#### 定制Endpoint  

1. 自定以健康状态检查信息`extends AbstractHealthIndicator`  


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

1. `implements HealthIndicator`  



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

配置

```yml
management:
    health:
      enabled: true
      show-details: always #总是显示详细信息。可显示每个模块的状态信息
```

#### Info信息
访问 http://localhost:8080/actuator/info 默认为空,可以对其进行配置  
**配置文件中的@@符号可以获取pom.xml文件中的属性**

```yml
info:
  appName: Boot-admin
  appVersion: 1.0
  mavenProjectName: @project.artifactId@
  mavenProjectVersion: @project.version@
```

```json
{
"appName": "Boot-admin",
"appVersion": 1,
"mavenProjectName": "springboot-ricweb",
"mavenProjectVersion": "0.0.1-SNAPSHOT"
}
```

或者使用`InfoContributor` 定制信息

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

### 监控可视化 

https://github.com/codecentric/spring-boot-admin 


## Profile 功能
- 默认配置文件 `application.yml` 任何时候都会加载
- 指定环境配置文件，`application-{env}.yml` 
- 激活指定环境
  - 配置文件激活
  - 命令行激活`java -jar xxx.jar --spring.profiles.active=prod --person.name=haha` 
    - 修改配置文件的值以命令行优先 优先级:`命令行>指定文件>默认文件` 命令行可以修改任意配置文件内容


### 条件装配

```java
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```
### 分组
```yml
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq

使用：--spring.profiles.active=production  激活
```

## 外部化配置
指定环境优先，外部优先，后面的可以覆盖前面的同名配置项

### 外部配置源
常用：Java属性文件、YAML文件、环境变量、命令行参数；

### 配置文件查找位置 
**以下的顺序是由上到下 下级覆盖上级**
- classpath 根路径
- classpath 根路径下config目录 
- Jar包当前目录 
- Jar包当前目录的config目录
- /config子目录的直接子目录

> 可以利用这点 临时修改或者增加配置项 直接在jar包运行目录 下添加配置文件或者config目录下添加配置文件

## 自定义stater
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/boot14.png)











## 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV19K4y1L7MT)
> - []()
