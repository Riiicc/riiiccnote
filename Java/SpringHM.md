# Srping 详解黑马笔记

## 参考资料
> - [黑马spring](https://www.bilibili.com/video/BV1P44y1N7QG)  

## BeanFactory 和 ApplicationContext  

### 关系 

> 二者关系如图 `ApplicationContext` 间接继承了 `BeanFactory`
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ConfigurableApplicationContext.png)   

> `ApplicationContext` 对 `BeanFactory` 功能进行了扩展,通过继承不同的接口,可以实现功能的进一步扩展     

扩展主要体现:
- `MessageSource` i18n国际化
  - `context.getMessage("hi",null,Locale.CHINA)` 获取hi对应的中文
- `ResourcePatternResolver` 通配符匹配资源
  - `context.getResources("calsspath*:META-INF/spring.factories)`
- `ApplicationEventPublisher` 发送事件
  - `context.puhblicEvent() ` 通过 `@EventListener` 进行监听  
- `EnvironmentCapable` 获取环境变量 配置信息  
  - `context.getEnivirment().getProperty("java_home")`

### BeanFactory 实现    

```java
public class TestBeanFactory {
    public static void main(String[] args) {
        //理解整个 beanFactory 的流程 作用
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        //bean 的定义 class scope 初始化  销毁
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Config.class).setScope("singleton").getBeanDefinition();
        //注册bean
        beanFactory.registerBeanDefinition("config",beanDefinition);
        //此时bean工厂就有一个 名为 config 的bean 类型为Config.class
        Arrays.stream(beanFactory.getBeanDefinitionNames()).forEach(System.out::println);//config

        //给 beanFactory 添加一些常用的 后置处理器
        AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);

        Arrays.stream(beanFactory.getBeanDefinitionNames()).forEach(System.out::println);
//        org.springframework.context.annotation.internalConfigurationAnnotationProcessor  识别 @configuration 但是并不能解析 类中的@bean
//        org.springframework.context.annotation.internalAutowiredAnnotationProcessor 识别 @autowired
//        org.springframework.context.annotation.internalCommonAnnotationProcessor 识别 JSR250注解 @resource
//        org.springframework.context.event.internalEventListenerProcessor
//        org.springframework.context.event.internalEventListenerFactory

        //BeanFactory 后处理器,补充bean 定义
        beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).values().forEach(beanFactoryPostProcessor -> {
            //执行 每个后置处理器 (识别对应注解标识的类或方法) 这里执行完就可以 解析 (@configuration)类中的@bean
            beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
        });
        //这里会调用bean1 构造方法实例化bean1 但是无法获取bean2 打印null 无法识别@autowired
//        Bean2 bean2 = beanFactory.getBean(Bean1.class).getBean2();
//        System.out.println(bean2);

        //bean 后处理器 针对bean 的生命周期各个阶段提供扩展 ,如 @autowired @resource
        beanFactory.getBeansOfType(BeanPostProcessor.class).values().forEach(beanFactory::addBeanPostProcessor);

        Arrays.stream(beanFactory.getBeanDefinitionNames()).forEach(System.out::println);

        //提前初始化单例bean
        beanFactory.preInstantiateSingletons();
        
        //打印会多出@bean 的bean 内容 bean1 bean2...
        System.out.println(">>>>>>");
        Bean2 bean2 = beanFactory.getBean(Bean1.class).getBean2();
        System.out.println(bean2);


    }

    @Configuration
    static class Config {
        @Bean
        public Bean1 bean1() {
            return new Bean1();
        }

        @Bean
        public Bean2 bean2() {
            return new Bean2();
        }

        @Bean
        public Bean3 bean3() {
            return new Bean3();
        }

        @Bean
        public Bean4 bean4() {
            return new Bean4();
        }
    }

    @Slf4j
    static class Bean1 {
        @Autowired
        private Bean2 bean2;

        public Bean2 getBean2() {
            return bean2;
        }

        @Autowired
        @Resource(name = "bean4")
        private Inter bean3;

        public Inter getInter() {
            return bean3;
        }

        public Bean1() {
            log.debug("构造 Bean1()");
        }
    }

    @Slf4j
    static class Bean2 {
        public Bean2() {
            log.debug("构造 Bean2()");
        }
    }

    interface Inter {

    }

    @Slf4j
    static class Bean3 implements Inter {
        public Bean3() {
            log.debug("构造 Bean3()");
        }
    }

    @Slf4j
    static class Bean4 implements Inter {
        public Bean4() {
            log.debug("构造 Bean4()");
        }
    }
}
```

> beanFactory 不会主动做的事,beanfactory 是个基础组件  
> - 不会主动调用BeanFactory的后处理器
> - 不会主动添加Bean的后处理器
> - 不会主动初始化单例
> - 不会解析BeanFactory，还不会解析 ${}, #{}

> 关于 **后置处理器加载顺序** 的问题思考     
> `@Autowired` 和 `@Resource` 同时使用会以 `@Autowired` 为准   
> 因为 在加载Bean后处理器时,`AnnotationConfigUtils.registerAnnotationConfigProcessors()`默认Autowired的后处理器 `internalAutowiredAnnotationProcessor` 先于 Resource的后处理器 `internalCommonAnnotationProcessor` 加载     


> 这个加载的顺序也是可以进行定义的,使用`beanFactory.getDependencyComparator()` 比较器进行排序  
> 排序的依据是 `BeanPostProcessor` 内部的 `order` 属性    
> 其中`internalAutowiredAnnotationProcessor`的`order`属性的值为`Ordered.LOWEST_PRECEDENCE - 2`   
> `internalCommonAnnotationProcessor`的order属性的值为`Ordered.LOWEST_PRECEDENCE - 3`= `2147483647 -3`       
> 排序默认小的在前面, 就会使得 `internalCommonAnnotationProcessor` 先于 `internalAutowiredAnnotationProcessor` 加载    


### ApplicationContext接口的实现类  
- `ClassPathXmlApplicationContext`: 
  - 从类路径下的一个或多个Xml配置文件中加载上下文定义
- `FileSystemXmlApplicationContext`:
  - 从文件系统下的一个或多个Xml配置文件加载 
- `AnnotationConfigApplicationContext`:
  - 从一个或多个java 配置类中加载Spring应用上下文  
- `AnnotationConfigServletWebServerApplication`


> `ClassPathXmlApplicationContext` 可以查找类路径下所有文件的xml文件(包括Jar文件内部的xml)   
> 两种基于xml文件读取配置文件来加载bean的方法,内部都是通过 `XmlBeanDefinitionReader.loadBeanDefinitions(new FileSystemResources("D:/xxx.xml")||new ClassPathResource("bean.xml"))` 来读取   


## Bean 生命周期 

```java
@Component
@Slf4j
public class LifeCycleBean {
  //1
    public LifeCycleBean() {
        log.debug("构造");
    }
//2
    @Autowired
    public void autowire(@Value("${JAVA_HOME}") String name) {
        log.debug("依赖注入：{}", name);
    }
//3
    //容器初始化调用,只调用一次
    @PostConstruct
    public void init() {
        log.debug("初始化");
    }
//4
    //容器销毁调用 只有单例时最后调用,其他模式不同
    @PreDestroy
    public void destroy() {
        log.debug("销毁");
    }
}

```

> 编写自定义 `Bean` 的后处理器，需要实现 `InstantiationAwareBeanPostProcessor` 和 `DestructionAwareBeanPostProcessor` 接口，并加上`@Component`注解，对 `lifeCycleBean` 的生命周期过程进行扩展

```java
@Slf4j
@Component
public class MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {
    @Override
    // 实例化前（即调用构造方法前）执行的方法
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean"))
            log.debug("<<<<<<<<<<< 实例化前执行");
        // 返回null保持原有对象不变，返回不为null，会替换掉原有对象
        return null;
    }

    @Override
    // 实例化后执行的方法
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            log.debug("<<<<<<<<<<< 实例化后执行，这里如果返回 false 会跳过依赖注入阶段");
            // return false;
        }

        return true;
    }

    @Override
    // 依赖注入阶段执行的方法
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean"))
            log.debug("<<<<<<<<<<< 依赖注入阶段执行，如@Autowired、@Value、@Resource");
        return pvs;
    }

    @Override
    // 销毁前执行的方法
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean"))
            log.debug("<<<<<<<<<<<销毁之前执行");
    }

    @Override
    // 初始化之前执行的方法
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean"))
            log.debug("<<<<<<<<<<< 初始化之前执行，这里返回的对象会替换掉原本的bean，如 @PostConstruct、@ConfigurationProperties");
        return bean;
    }

    @Override
    // 初始化之后执行的方法
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean"))
            log.debug("<<<<<<<<<<< 初始化之后执行，这里返回的对象会替换掉原本的bean，如 代理增强");
        return bean;
    }
}

```

## 模板方法  

```java
public class TestMethodTemplatePattern {
    public static void main(String[] args) {
        MyBeanFactory beanFactory = new MyBeanFactory();
        beanFactory.addBeanPostProcessor(bean -> System.out.println("解析 @Autowired"));
        beanFactory.addBeanPostProcessor(bean -> System.out.println("解析 @Resource"));
        beanFactory.getBean();
    }

    static class MyBeanFactory {
        public Object getBean() {
            Object bean = new Object();
            System.out.println("构造：" + bean);
            System.out.println("依赖注入：" + bean);
            for (BeanPostProcessor processor : processors) {
                processor.inject(bean);
            }
            System.out.println("初始化：" + bean);
            return bean;
        }

        private List<BeanPostProcessor> processors = new ArrayList<>();

        public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor) {
            processors.add(beanPostProcessor);
        }
    }

    interface BeanPostProcessor {
        void inject(Object bean);
    }
}

```

## Bean 后处理器  

> Bean后处理器作用,为Bean 生命周期各个阶段提供扩展  

```java
public class Test01 {


    public static void main(String[] args) {
        //干净的spring 容器,不会自动加载bean
        GenericApplicationContext context = new GenericApplicationContext();

        //注册bean
        context.registerBean("bean1",Bean1.class);
        context.registerBean("bean2",Bean2.class);
        context.registerBean("bean3",Bean3.class);

        //获取value 识别
        context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        //添加 autowired value 后处理器
        context.registerBean(AutowiredAnnotationBeanPostProcessor.class);

        // @Resource @PostConstruct @PreDestroy JSR-250 support
        context.registerBean(CommonAnnotationBeanPostProcessor.class);

        //@ConfigurationProperties
        ConfigurationPropertiesBindingPostProcessor.register(context.getDefaultListableBeanFactory());
        //初始化容器
        context.refresh();
        //销毁容器
        context.close();


    }
}
```

- `@Autowired` 注解对应的后处理器是 `AutowiredAnnotationBeanPostProcessor`
- `@Value`注解需要配合`@Autowired`注解一起使用，所以也用到了`AutowiredAnnotationBeanPostProcessor`后处理器，然后`@Value`注解还需要再用到`ContextAnnotationAutowireCandidateResolver`解析器，否则会报错；
- `@Resource`、`@PostConstruct`、`@PreDestroy`注解对应的后处理器是`CommonAnnotationBeanPostProcessor`
- `@ConfigurationProperties`注解对应的后处理器是 `ConfigurationPropertiesBindingPostProcessor`   

> 注: `CommonAnnotationBeanPostProcessor` 处理器处理 `JSR-250`规范的注解   
> JSR是Java Specification Requests的缩写，意思是Java 规范提案   
> JSR 250 规范包含用于将资源注入到端点实现类的注释和用于管理应用程序生命周期的注释    
> 包含 JSR 250 标准中的每一个注释的 Java™ 类的名称为 `javax.annotation.xxx`   
> spring对JSR-250所提供的这**三个注解**提供支持，可以使用 `@Resource`、`@PostConstruct`、`@PreDestroy`     


### @Autowired 后处理器详解

- `@Autowired` 注解解析用到的后处理器是 `AutowiredAnnotationBeanPostProcessor`  
- 这个后处理器就是通过调用 `postProcessProperties(PropertyValues pvs, Object bean, String beanName)`完成注解的解析和注入的功能 
- 这个方法中又调用了一个私有的方法 `findAutowiringMetadata(beanName, bean.getClass(), pvs)`，其返回值 `InjectionMetadata` 中封装了被 `@Autowired` 注解修饰的属性和方法  
- 然后会调用`InjectionMetadata.inject(bean1, "bean1", null)`进行依赖注入   

> 下面的代码分别展示了通过反射使用 `findAutowiringMetadata` 来进行注入的三种不同情形    

```java
public class Test02 {
    public static void main(String[] args) throws Throwable {

        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        beanFactory.registerSingleton("bean2",new Bean2());
        beanFactory.registerSingleton("bean3",new Bean3());
        beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        //${} 解析器 解析spel表达式
        beanFactory.addEmbeddedValueResolver(new StandardEnvironment()::resolvePlaceholders);

        AutowiredAnnotationBeanPostProcessor postProcessor = new AutowiredAnnotationBeanPostProcessor();

        postProcessor.setBeanFactory(beanFactory);
        Bean1 bean1 = new Bean1();

        System.out.println(bean1);
        //调用postProcessProperties 完成注解解析和注入
        postProcessor.postProcessProperties(null,bean1,"bean1");
        System.out.println(bean1);

        //下面通过单独调用方法 来模拟  postProcessor.postProcessProperties 的过程
        //调用 postProcessProperties 内部方法 findAutowiringMetadata 来识别 autowired 和 value
        Method findAutowiringMetadata = AutowiredAnnotationBeanPostProcessor.class.getDeclaredMethod("findAutowiringMetadata", String.class, Class.class, PropertyValues.class);
        findAutowiringMetadata.setAccessible(true);

        //获取bean1 上加了autowired value 的成员变量 方法参数信息
        InjectionMetadata metadata = (InjectionMetadata)findAutowiringMetadata.invoke(postProcessor, "bean1", Bean1.class, null);

        //调用 inject方法进行 依赖注入,注入时按类型查找值
        metadata.inject(bean1,"bean1",null);

        //按照类型查找值
        /*
            @Autowired
            private Bean3 bean3;
         */
        Field bean3 = Bean1.class.getDeclaredField("bean3");
        DependencyDescriptor dependencyDescriptor = new DependencyDescriptor(bean3, false);

        Object o = beanFactory.doResolveDependency(dependencyDescriptor, null, null, null);
        System.out.println(o);//com.example.a04.Bean3@3b084709


        //按方法 注入查找
        /*
         @Autowired
        public void setBean2(Bean2 bean2) {
            log.debug("@Autowired 生效：{}", bean2);
            this.bean2 = bean2;
        }
         */
        Method setBean2 = Bean1.class.getDeclaredMethod("setBean2",Bean2.class);
        DependencyDescriptor me = new DependencyDescriptor(new MethodParameter(setBean2, 0), false);
        Object o1 = beanFactory.doResolveDependency(me, null, null, null);
        System.out.println(o1);//com.example.a04.Bean2@22a67b4


        /*
         @Autowired
        public void setJava_home(@Value("${JAVA_HOME}") String java_home) {
            log.debug("@Value 生效：{}", java_home);
            this.java_home = java_home;
        }
         */
        Method str = Bean1.class.getDeclaredMethod("setJava_home",String.class);
        DependencyDescriptor stdd = new DependencyDescriptor(new MethodParameter(str, 0), false);
        Object o2 = beanFactory.doResolveDependency(stdd, null, null, null);
        System.out.println(o2); //C:\Program Files\Java\jdk1.8.0_91
    }
}
```

## BeanFactory 后处理器 

> `@ComponentScan`、`@Bean`,`@Import`,`@ImportResource`对应的Bean工厂后处理器是 `ConfigurationClassPostProcessor`   
> `@MapperScan` 对应的Bean工厂后处理器是 `MapperScannerConfigurer`   

```java
        //干净容器
        GenericApplicationContext context = new GenericApplicationContext();
        context.registerBean("config",Config.class);
        context.registerBean(ConfigurationClassPostProcessor.class);// 识别 @componentScan @Bean @Import @ImportResource

        context.registerBean(MapperScannerConfigurer.class,bd->{
            //@MapperScanner
            bd.getPropertyValues().add("basePackage","com.example.a05.mapper");
        });
```

### Component 解析过程  
- 解析包名`@ComponentScan("com.example.a05.component")` 为路径格式匹配 `classpath*:com/example/a05/component/**/*.class`

```java
        ComponentScan annotation = AnnotationUtils.findAnnotation(MyConfig.class, ComponentScan.class);
        for (String s : annotation.basePackages()) {
            System.out.println(s);//com.example.a05.component
            //获取路径
            String path = "classpath*:" + s.replace(".", "/") + "/**/*.class";
            //获取实际资源
            Resource[] resources = context.getResources(path);
            CachingMetadataReaderFactory cachingMetadataReaderFactory = new CachingMetadataReaderFactory();
            //根据注解生成bean名称
            AnnotationBeanNameGenerator generator = new AnnotationBeanNameGenerator();

            for (Resource resource : resources) {
                System.out.println(resource);//file [D:\SpringSt\learning-java\hmspring\target\classes\com\example\a05\component\Bean2.class]
                //读取对应的class
                MetadataReader metadataReader = cachingMetadataReaderFactory.getMetadataReader(resource);
                //类的源信息
                ClassMetadata classMetadata = metadataReader.getClassMetadata();
                System.out.println("类名:" + classMetadata.getClassName());//com.example.a05.component.Bean2
                //注解信息元数据
                AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
                //是否加了Component注解true
                System.out.println("当前Bean是否加了Component注解" + annotationMetadata.hasAnnotation(Component.class.getName()));
                //判断@controller等派生注解 不包含 Component
                System.out.println("当前Bean是否加了派生注解" + annotationMetadata.hasMetaAnnotation(Component.class.getName()));

                if (annotationMetadata.hasAnnotation(Component.class.getName()) ||
                        annotationMetadata.hasMetaAnnotation(Component.class.getName())) {
                    AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder
                            .genericBeanDefinition(classMetadata.getClassName()).getBeanDefinition();
                    DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
                    //bean 的名称
                    String beanName = generator.generateBeanName(beanDefinition, beanFactory);
                    //注册
                    beanFactory.registerBeanDefinition(beanName,beanDefinition);

                }


            }
        }

```

### Bean 解析过程  

```java
CachingMetadataReaderFactory metadataReaderFactory = new CachingMetadataReaderFactory();
        //getMetadataReader 不通过类加载读取class
        MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(new ClassPathResource("com/example/a05/MyConfig.class"));
        //所有被@Bean 标注的方法
        Set<MethodMetadata> annotatedMethods = metadataReader.getAnnotationMetadata().getAnnotatedMethods(Bean.class.getName());
        for (MethodMetadata method : annotatedMethods) {
            //打印所有被@bean 标注的方法全类名
            System.out.println(method);
            //@Bean注解属性解析过程  以initMethod 为例
            //有属性值就返回,没有就返回空字符串
            String initMethod = (String)method.getAnnotationAttributes(Bean.class.getName()).get("initMethod");

            BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
            //通过 config bean 调用 @Bean 方法
            builder.setFactoryMethodOnBean(method.getMethodName(),"config");
            //自动装配策略 构造器 ,
            builder.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR);
            if(initMethod.length()>0){
                //配置初始化方法,配置之后才会执行 @Bean 中的initMethod 配置的方法
                builder.setInitMethodName(initMethod);
            }
            AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
            context.getDefaultListableBeanFactory().registerBeanDefinition(method.getMethodName(),beanDefinition);

        }
```


### Mapper 解析 

- 使用 `MapperFactoryBean<***Mapper>` 添加单个 Mapper 


```java
    @Bean
    public MapperFactoryBean<Mapper1> mapperFactoryBean1(SqlSessionFactory sqlSessionFactory) {
        MapperFactoryBean<Mapper1> mapperFactoryBean = new MapperFactoryBean<>(Mapper1.class);
        mapperFactoryBean.setSqlSessionFactory(sqlSessionFactory);
        return mapperFactoryBean;
    }
```

- 模拟实现  

```java

/**
 * 模拟 @Mapper 实现 
 */
public class MapperPostProcessor implements BeanDefinitionRegistryPostProcessor {

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry beanFactory) throws BeansException {
        try {

            //扫描class资源
            PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
            Resource[] resources = resolver.getResources("classpath:com/example/a05/mapper/**/*.class");

            CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
            AnnotationBeanNameGenerator generator = new AnnotationBeanNameGenerator();
            for (Resource resource : resources) {
                MetadataReader metadataReader = factory.getMetadataReader(resource);
                //类的元信息
                ClassMetadata classMetadata = metadataReader.getClassMetadata();
                //资源是否为接口
                if (classMetadata.isInterface()) {
                    //定义bean
                    AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(MapperFactoryBean.class)
                            .addConstructorArgValue(classMetadata.getClassName())//给构造方法设置参数值
                            .setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE)//自动装配模式
                            .getBeanDefinition();
                    //使用另一个bean definition 来生成beanName
                    AbstractBeanDefinition beanDefinition1 = BeanDefinitionBuilder.genericBeanDefinition(classMetadata.getClassName()).getBeanDefinition();
                    String beanName = generator.generateBeanName(beanDefinition1, beanFactory);
                    //注册bean
                    beanFactory.registerBeanDefinition(beanName,beanDefinition);
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }

    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

    }
}
```


## Aware 和 InitializingBean 接口   

- `Aware`接口用于注入一些与容器相关的信息
  - `BeanNameAware` bean名称
  - `BeanFactoryAware` 注入 `BeanFactory` 容器
  - `ApplicationContextAware` 注入 `ApplicationContext` 容器
  - `EmbeddedValueResolverAware` ${}  

```java
public class MyBean implements BeanNameAware, ApplicationContextAware, InitializingBean {
    @Override
    public void setBeanName(String name) {

        log.debug("当前bean：" + this + "，实现 BeanNameAware 调用的方法，名字叫：" + name);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        log.debug("当前bean：" + this + "，实现 ApplicationContextAware 调用的方法，容器叫：" + applicationContext);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        log.debug("当前bean：" + this + "，实现 InitializingBean 调用的方法，初始化");
    }

}
```

> Aware 接口提供了一种 内置的注入手段,可以注入 `BeanFactory` , `ApplicationContext`    
> `InitializingBean` 接口提供了一种内置的初始化手段   
> 内置的注入和初始化不受扩展功能影响,总会被执行,因此Spring 框架内部的类经常使用    


> 注意: 和`@Autowired`的区别   
>  `@Autowired`等注解基于bean 后处理器实现,属于扩展功能,有可能失效   
> Aware 属于内置功能,不需要加添扩展,spring也能识别  

## 初始化和销毁  

**三种初始化方法,执行顺序从上到下**
- `@PostConstruct` 
- `InitializingBean`接口实现
- `@Bean(initMethod="xxx")`

```java
public class Bean1 implements InitializingBean {
    @PostConstruct
    public void init1() {
        log.debug("初始化1，@PostConstruct");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        log.debug("初始化2，InitializingBean接口");
    }

    public void init3() {
        log.debug("初始化3，@Bean的initMethod");
    }
}

```

**三种销毁方法,执行顺序从上到下**  

- `@PreDestroy`
- `DispoableBean` 接口实现
- `@Bean(destroyMethod="xxx")`


## Scope 

### Scope类型
- `singleton` 单例,创建销毁跟随spring 容器
- `prototype` 多例,使用时创建,可以自定义销毁,容器不会主动销毁
- `request` 每个web请求一个实例,跟随请求创建和销毁 
- `session` 每个web会话一个实例(理解为浏览器关闭失效),创建销毁跟随会话
- `application` java程序一个实例  

> spring boot 默认session 过期时间最低为1min 低于1min按照一分钟计算   

### 失效实例分析   

- 在单例对象F中注入多例E,多例E失效

> 单例F中获取多例对象,每次都是同一个E对象,多例失效   
> 对单例对象F来讲,依赖注入之发生了一次,也就是F中的E始终是第一个依赖注入的E  

> 解决:   
> 使用 `@Lazy` 注解,在获取时才进行真正的注入     
> 在多例对象中配置proxyMode `@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)`    
> 使用`ObjectFactory<PrototypeBean3>`工厂类(详见下面代码)  
> 直接注入 `ApplicationContext` 通过 `getBean(Class)` 方法获取bean  


```java

@Scope("prototype")
@Component
public class PrototypeBean3 {
}
//下面伪代码
//使用ObjectFactory<PrototypeBean3>工厂类  
    @Autowired
    private ObjectFactory<PrototypeBean3> factory;
    
    //通过 getObject()获取实例   
     public PrototypeBean3 getPrototypeBean3() {
        return factory.getObject();
    }


    @Autowired
    private ApplicationContext context;
    
    public PrototypeBean4 getPrototypeBean4() {
        return context.getBean(PrototypeBean4.class);
    }

```

## AOP  
















