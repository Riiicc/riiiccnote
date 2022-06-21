# Spring 注解驱动开发 
> 当前spring版本 **4.1.2RELEASE** ，仅用于思想参考，其中的很多注解已经有变化了（Spring5 ），实际应用要注意

## 组件注册
### @Conifguration  
> 在IOC容器中组件注册 

注入IOC

```java
@Configuration
public class MyConfig {

    @Bean
    public Person person01(){
        return new Person("李四",12);
    }
    @Bean(name = "ppp")
    public Person person02(){
        return new Person("李四1",12);
    }
}
```
测试调用 
```java
public class TestCl {
    public static void main(String[] args) {
//        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
//        Object person = context.getBean("person");
//        System.out.println(person);

        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(MyConfig.class);
//        Object person01 = applicationContext.getBean("pppp");
        String[] beanNamesForType = applicationContext.getBeanNamesForType(Person.class);

        System.out.println(Arrays.toString(beanNamesForType));
    }
}
```


### @ComponentScan
> 自动扫描组件,定义扫描规则 `@ComponentScan` 可以通过 `value` 属性指定要扫描的包,就可以将 `@Component @Controller @Service @Repository` 注解的类放入IOC容器中

> `excludeFilters` 指定排除规则,`FilterType.ANNOTATION` 排除注解 
> `includeFilters` 只包含指定规则的组件,前提是需要配置 `useDefaultFilters = false` 
```java 
//排除Controller注解和Service 注解 
//包含Repository 注解
@ComponentScan(value = "com.ric",
        excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value= {Controller.class, Service.class})},
        includeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value = Repository.class)},
        useDefaultFilters = false)
```
> `@ComponentScan` 可以多个同时使用,或者使用`@ComponentScans` 包裹多个 `@ComponentScan`
> 
### FilterType规则

```java
public enum FilterType {
    ANNOTATION,//注解
    ASSIGNABLE_TYPE,//类型
    ASPECTJ,//aspectj表达式
    REGEX,//正则表达式
    CUSTOM;//自定义规则

    private FilterType() {
    }
}
```
#### 自定义FilterType规则
 
> 通过 `MyTypeFilter` 类实现判断逻辑,类中实现 `TypeFilter` 接口 
```java

@ComponentScan(value = "com.ric",
//        excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value= {Controller.class, Service.class})},
//        includeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value = Repository.class)},
        includeFilters = {@ComponentScan.Filter(type = FilterType.CUSTOM,value = MyTypeFilter.class)},
        useDefaultFilters = false)
```

```java
public class MyTypeFilter implements TypeFilter {

    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        //获取当前类的注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取当前正在扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //获取当前类资源(类路径)
        Resource resource = metadataReader.getResource();

        String className = classMetadata.getClassName();
        //只允许类名中有ll 的放入IOC容器
        if (className.contains("ll")){
            return true;
        }
        System.out.println(className);

        return false;
    }
}

```

### Scope 设置

- prototype 多实例 调用(获取Bean)时创建
- singleton 单实例 **默认值**  IOC容器启动即创建
- ~~request 同一次请求创建一个实例~~
- ~~session 同一个session 创建一个实例~~
- ~~global-session~~ 在一个全局HTTP Session中，每个Bean定义对应一个实例。该作用域仅在基于Web的Spring上下文中才有效


### 懒加载Bean @Lazy-bean 

> 单实例bean,默认在容器启动的时候创建对象,
> 懒加载: `@Lazy` 容器启动不创建对象,**第一次**使用(获取) Bean创建对象,并初始化;**针对单实例bean**

```java
@Configuration
public class MyConfig2 {

    @Lazy
    @Bean
    @Scope
    public Person person(){
        return new Person("11",11);
    }
}
```


### @Conditional 
> 按照一定的条件进行判断,满足条件给容器中注册bean,可以放在方法上也可以放在类上(针对类中所有bean)  

```java
/**
 * conditionContext 判断条件能使用的上下文环境
 * annotatedTypeMetadata 注释信息
 */
public class MyConditional implements Condition {

    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        //获取ioc使用的Beanfactory
        ConfigurableListableBeanFactory beanFactory = conditionContext.getBeanFactory();
        //获取当前环境信息
        Environment environment = conditionContext.getEnvironment();
        //获取类加载器
        ClassLoader classLoader = conditionContext.getClassLoader();
        //获取bean定义的注册类
        BeanDefinitionRegistry registry = conditionContext.getRegistry();
        //判断容器中是否包含person Bean
        boolean person = registry.containsBeanDefinition("person");


        String property = environment.getProperty("os.name");
        //判断只有windows系统放入IOC容器
        if (property.contains("Windows")){
            return true;
        }

        return false;
    }
}


```

```java
    @Bean
    @Conditional(value = {MyConditional.class})
    public Person person01(){
        return new Person("person01",11);
    }
```

### @Import
容器中注册组件
- 自己写的类,包扫描+组件标注注解,`@Controller,@Service,@Repository,@Component`
- 导入第三方组件 `@Bean` 
- `@Import(Color.class)` 导入组件,id默认时组件的全类名 `com.ric.Color`
- `ImportSelector` 返回需要导入的组件的全类名 调用方式 `@Import(MySelector.class)` `MySelector`类实现`ImportSelector`接口,实现方法返回全类名数组

```java
public class MySelector implements ImportSelector {
    public String[] selectImports(AnnotationMetadata annotationMetadata) {

        return new String[]{"com.ric.bean.Yellow","com.ric.bean.Blue"};

    }
}

//调用方法
@Import(MySelector.class)
```

- `ImportBeanDefinitionRegistrar` 

```java

public class MyDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    /**
     * @param annotationMetadata 当前类的注解信息
     * @param beanDefinitionRegistry 注册类,通过其注册Bean
     */
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {

        //如果容器中存在red bean 就将Yellow注册
        boolean red = beanDefinitionRegistry.containsBeanDefinition("red");
        if(red){
            beanDefinitionRegistry.registerBeanDefinition("yellow",new RootBeanDefinition(Yellow.class));
        }
    }

}

//调用方法
@Import(MyDefinitionRegistrar.class)
```

### FactoryBean

> 使用Spring 提供的FactoryBean (工厂Bean),实现`FactoryBean`接口  
> 注:获取`FactoryBean` 本身使用 `applicationContext.getBean("&colorFactoryBean");` 
> &在`BeanFactory` 接口中有说明 `String FACTORY_BEAN_PREFIX = "&";`

```java
public class ColorFactoryBean implements FactoryBean<Color> {


    /**
     * @return Color对象 这个对象会添加到容器中
     * @throws Exception
     */
    public Color getObject() throws Exception {
        System.out.println("TTTT");
        return new Color();
    }

    public Class<?> getObjectType() {
        return Color.class;
    }

    /**
     * @return 是否单例
     */
    public boolean isSingleton() {
        return true;
    }
}

```

```java
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
```


## Bean生命周期
> Bean创建- 初始化-销毁的过程
> 容器管理bean的生命周期
> 我们可以自定义初始化和销毁方法,容器在进行到当前声明周期的时候来调用我们自定义的初始化和销毁方法
> 示例,数据源的操作

- 构造对象
  - 单实例:在容器启动时创建对象
  - 多实例:在每次获取的时候创建对象

- 初始化: 对象创建完成并切赋值好,调用初始化方法

- 销毁:
  - 单实例: 容器关闭时
  - 多实例: 容器不会管理这个bean(只会创建),**容器不会调用销毁方法**(需要手动调用销毁方法)

- 指定初始化和销毁方法
  - 1. 通过`@Bean` 指定init-method 和 destroy-method
  - 2. 通过让Bean 实现 `InitializingBean, DisposableBean` 接口 
  - 3. 使用`@PostConstruct` bean 创建完成属性赋值后,执行其标注的方法,`@PreDestroy` bean 销毁前进行调用,通知进行清理工作
  - 4. `BeanPostProcessor` bean 的后置处理器 ,在bean的初始化前后进行工作
    - 接口方法 `postProcessBeforeInitialization` 在bean 初始化(initMethod) 之前进行工作
    - 接口方法 `postProcessAfterInitialization` 在bean 初始化(initMethod) 之后工作

使用注解
```java
@Configuration
public class MyInitConfig {

    @Scope("prototype")
    @Bean(initMethod = "initMethod",destroyMethod = "destroyMethod")
    public Car car(){
        System.out.println("bean");
        return new Car();
    }
}
//实体
public class Car {
    public Car() {
        System.out.println("constr");
    }
    public void initMethod(){
        System.out.println("initMethod");
    }
    public void destroyMethod(){
        System.out.println("destroyMethod");
    }
}

// 测试 
    @Test
    public void test03(){
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MyInitConfig.class);
        Object colorFactor = applicationContext.getBean("car");
        applicationContext.close();

    }
// 结果
// bean
// constr
// initMethod
// destroyMethod


```

实现`InitializingBean, DisposableBean`类方法
```java
public class Cat implements InitializingBean, DisposableBean {
    public void destroy() throws Exception {
        //bean 销毁时调用
        System.out.println("cat destroyMethod");
    }

    public void afterPropertiesSet() throws Exception {
        //bean 创建完成,属性值赋值完成,
        System.out.println("cat initMethod");

    }

    public Cat() {
        System.out.println("cat init");
    }
}
```

`PostConstruct` 注解
```java
public class Dog {
    public Dog() {

    }
    @PostConstruct
    public void initMethod(){
        System.out.println("dog--initMethod");
    }
    @PreDestroy
    public void destroyMethod(){
        System.out.println("dog--destroyMethod");
    }
}
```
`BeanPostProcessor` 后置处理器,会为容器中的所有组件添加处理器
```java

public class MyBeanPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        System.out.println("postProcessBeforeInitialization"+o+">>>>"+s);
        return o;
    }

    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("postProcessAfterInitialization"+o+">>>>"+s);
        return o;
    }
}
```

- Spring底层对 `BeanPostProcessor` 使用
  - bean 赋值 
  - 注入其他组件
  - `@Autowried`
  - 生命周期注解功能
  - `@Async`
  - xxxBeanPostProcessor 

## 属性赋值

### @Value /@PropertySource

- @Value使用的几种方式
  - 基本数值(以字符串形式) `@Value("张三")`
  - SpEl `#{}` 如:`@Value("#{20-3}")`
  - `${xxx.xx}` 取出配置文件中的值(运行环境变量中的值)
    - 通过`@PropertySource(value = {"classpath:/application.properties"})` 配置文件路径

## 自动装配

> Spring 利用依赖注入 完成对容器中各个组件的依赖关系赋值

### @Autowired
  - 默认有限按照类型寻找对应组件
  - 如果有有多个相同类型组件,默认按照属性名称作为id去容器中加载, `private BookDao bookDao;`默认先寻找**id为bookDao的bean**
  - 使用`@Qualifier("bookDao2")` 指定寻找id为bookDao2 的bean
  - 使用`@Autowried(required = false)` 作为非必须的标识, 如果容器不存在需要的bean就会返回空,而不会报错
  - 通过`@Primary` 注解设置 Bean的默认装配优先级,被其注解的bean默认优先加载,可以加在类上

###  @Resource和@Inject
- spring 还支持使用` @Resource` 和 `@Inject`
  - `@Resource` 是 `javax.annotation` 包中,默认按照组件名称进行装配,不支持优先级设置等其他
  - `@Inject` 需要导入 `javax.inject` 依赖,功能和autowired基本相同,支持`Primary` 不支持`required=false` 

### 方法,构造器位置的自动装配
  - 标注在方法上的 `@Autowried` Spring容器创建当前对象就会调用方法完成赋值,方法参数从ioc容器中获取 
  - 标注在构造器上或者构造器的入参(只有一个构造器可省略@Autowried)上  同上 
  - @bean 创建对象时同样可以 入参获取

### Aware接口
> 自定义组件想要使用Spring容器底层的一些组件,如`ApplicationContext` `BeanFactory` 等
> 自定义组件实现 `xxxAware`  在创建对象的时候,会调用接口规定的方法注入相关组件
> 如 `ApplicationContextAware` `EnvironmentAware` `ResourceLoaderAware`,这类Aware 都是使用后置处理器来实现相关功能

### @Profile  
> Spring 提供的可以根据当前环境,动态激活和切换一系列组件的功能 
> 示例`@Profile("test")` `@Profile("prod")`  默认值：`@Profile("default")` ，在`@Bean`方法或者类组件上添加,
> 没有标注的bean 在任何环境下都加载

**使用方法**
- 使用命令行参数指定 环境信息 `-Dspring.profiles.active=test`  多个使用 `-Dspring.profiles.active="test,dev"`
- 使用代码方式激活环境
```java
    public void test03(){
        //创建applicationContext 无参构造
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        //设置激活环境 可以指定多个
        applicationContext.getEnvironment().setActiveProfiles("test","dev");
        //注册配置类
        applicationContext.register(MyConfig.class);
        //刷新环境
        applicationContext.refresh();
        
    }
```


## AOP
> 在程序运行期间动态的将某段代码切入到指定位置进行运行的编程方式,底层为动态代理 


### 使用
- 导入AOP模块 `spring-aspects`
- 通知方法
  - 前置通知`@Before`:目标方法运行之前运行
  - 后置通知`@After`:目标方法运行之后运行,包括正常结束和异常结束
  - 返回通知`@AfterReturning`:目标方法正常返回之后运行
  - 异常通知`@AfterThrowing`:目标方法运行出现异常后运行
  - 环绕通知`@Around`:动态代理,手动推进目标方法运行 `joinPoint.procced()``
- 给切面类标注通知注解,切面表达式, `execution` ,切入点抽取`@Pointcut`
- 将切面类和业务逻辑类都加入到容器中
- 标记切面类,`@Aspect` 
- 配置类中添加 `@EnableAspectJAutoProxy`,开启基于注解的aop模式
- 定制方法信息
  - 切面方法入参定制 `JoinPoint`方法信息 `returning`返回值信息 `throwing`异常信息 

**拢共分三步** 
- 1. 将业务逻辑组件和切面类都加入到容器中,使用`@Aspect` 标记切面类
- 2. 在切面类上的每个通知方法 标注通知注解(注意标注的几种方法,表达式写法,提取切入点)
- 3. 开启基于注解的AOP模式 `@EnableAspectJAutoProxy`


```java
package com.ric.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;

/**
 * 切面类
 */
@Aspect
public class LogAspects {

    //抽取切入点
    @Pointcut(value = "execution(public int com.ric.aop.MathCalculator.*(..))")
    public void pointCut(){}

    @Before("pointCut()")
    public void start(JoinPoint joinPoint){
        //入参
        Object[] args = joinPoint.getArgs();
        //方法名
        String name = joinPoint.getSignature().getName();
        System.out.println("运行开始");
    }

    @After("pointCut()")
    public void end(){
        System.out.println("运行结束");
    }

    @AfterReturning(value = "pointCut()",returning = "result")
    public void logreturn(Object result){
        //result 封装了返回值

        System.out.println("运行返回");
    }
    @AfterThrowing(value = "pointCut()",throwing = "exp")
    public void logExp(JoinPoint joinPoint,Exception exp){
        //exp 异常信息
        System.out.println(joinPoint.getSignature().getName()+"运行异常");
    }
}


//配置类
@Configuration
@EnableAspectJAutoProxy
public class MyConfigAOP {

    @Bean
    public MathCalculator mathCalculator(){
        return new MathCalculator();
    }

    @Bean
    public LogAspects logAspects(){
        return new LogAspects();
    }
}

```

### 原理  
略 p28-p35

总结： 
- `@EnableAspectJAutoProxy` 开启AOP功能
- `@EnableAspectJAutoProxy` 会给容器中注册一个组件->`AnnotationAwareAspectJAutoProxyCreator` 
- `AnnotationAwareAspectJAutoProxyCreator` 是一个后置处理器
  - `registerBeanPostProcessors` 注册后置处理器,创建`AnnotationAwareAspectJAutoProxyCreator` 对象
  - `finishBeanFactoryInitialization` 初始化剩下的单实例bean
    - 创建业务逻辑组件和切面组件
    - `AnnotationAwareAspectJAutoProxyCreator` 拦截组件创建过程
    - 判断组件是否组要增强, 是,切面的通知方法包装成增强器 `Advisor` ;给业务逻辑组件创建一个代理对象`cglib`
  - 执行目标方法:
    - 代理对象执行目标方法


## 声明式事务 
> 导入依赖,数据源,数据库驱动,spring jdbc 
> 配置数据源 jdbcTemplate
> `@EnableTransactionManagement` 开启事务 `@Transactional` 标记需要回滚方法 
> 配置事务管理器控制事务 `PlatformTransactionManager`-> `DataSourceTransactionManager`

略


## 扩展原理

### BeanFactoryPostProcessor

> beanFactory 的后置处理器， 在BeanFacotry标准初始化之后调用,所有的Bean定义已经保存加载到beanFactory,但是bean的实例还未创建

### BeanDefinitionRegistryPostProcessor 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/springzz1.png)
> All regular bean definitions will have been loaded,but no beans will have been instantiated yet

> 其优先于父类 `BeanFactoryPostProcessor` 执行,可以在容器中再额外添加一些组件

### ApplicationListener
> 监听容器中发布的事件,事件驱动模型开发

- 监听器监听事件 ,`ApplicationEvent` 及其子类
- 把监听器加入到容器
- 只要容器中有相关事件发布,我们就能监听到这个事件
  - `ContextRefreshedEvent` 容器刷新完成,所有bean 完全创建完成会发布这个事件
  - `ContextClosedEvent` 容器关闭 
- `applicationContext.publishEvent()` 发布一个事件

- `@EventListener` 来监听事件

### Spring容器创建过程
略

## Servlet3









## 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV1gW411W7wy)
> - []()
