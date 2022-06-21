# 关于`《SpirngInAction4》`的笔记及理解   

# Bean 生命周期 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/bean生命周期.png)  

- Spring 对 bean 进行实例化
- Spring 将值和 bean的引用注入到bean对应的属性中
- 如果bean实现了 `BeanNameAware` 接口,Spring将bean 的ID传递给 `setBeanName()` 方法
- 如果bean实现了 `BeanFactoryAware` 接口,Spring 将调用` setBeanFactory()` 方法,将 BeanFactory 容器示例传入 
- 如果bean实现了 `ApplicationContextAware` 接口,Spring 将调用 `setApplicationContext()` 方法,将bean所在的应用上下文的引用传入
- 如果bean实现了 `BeanPostProcessor` 接口,Spring 将调用他们的 `postProcessBeforeInitialization()` 方法
- 如果bean实现了 `InitializingBean` 接口,Spring 将调用它们的 `afterPropertiesSet()` 方法,如果使用 `init-method` 声明了初始化方法,该方法也会被调用
- 如果bean实现了 `BeanPostProcessor` 接口,Spring 将调用它们的 `postProcessAfterInitialization()` 方法 
- 此时,bean已经准备就绪了,可以被应用程序使用
- 如果bean实现了 `DisposableBean` 接口,Spring 将调用它的 `destroy()` 接口方法,同样,如果使用 `destroy-method` 声明了销毁方法,该方法也会被调用  



## 对生命周期的理解   

1. 如果bean实现了`InstantiationAwareBeanPostProcessor` 接口,会在实例化前执行 `postProcessBeforeInstantiation` 方法
2. Spring 对 bean 进行实例化
3. 如果bean实现了`InstantiationAwareBeanPostProcessor` 接口,会在实例化后执行 `postProcessAfterInstantiation` 方法
4. 如果bean实现了`InstantiationAwareBeanPostProcessor` 接口,会在依赖注入前阶段执行 `postProcessProperties` 方法
5. bean中有`@Autowired @Value @Resource` 执行依赖注入,**包括赋值的set方法(xml 属性property注入方式)**
6. 如果bean实现了 `BeanNameAware` 接口,Spring将bean 的ID传递给 `setBeanName()` 方法
7. 如果bean实现了 `BeanFactoryAware` 接口,Spring 将调用` setBeanFactory()` 方法,将 BeanFactory 容器示例传入 
8. 如果bean实现了 `ApplicationContextAware` 接口,Spring 将调用 `setApplicationContext()` 方法,将bean所在的应用上下文的引用传入
9. 如果bean实现了 `BeanPostProcessor` 接口或其子接口,Spring 将调用他们的 `postProcessBeforeInitialization()` 方法
10. 如果bean实现了 `InitializingBean` 接口,Spring 将调用它们的 `afterPropertiesSet()` 方法,如果使用 `init-method` 声明了初始化方法,该方法也会被调用
11. 如果bean实现了 `BeanPostProcessor` 接口,Spring 将调用它们的 `postProcessAfterInitialization()` 方法 
12. 此时,bean已经准备就绪了,可以被应用程序使用
13. 如果bean 实现了接口 `DestructionAwareBeanPostProcessor` ,会在销毁前执行`postProcessBeforeDestruction`方法 
14. 如果bean实现了 `DisposableBean` 接口,Spring 将调用它的 `destroy()` 接口方法,同样,如果使用 `destroy-method`或者注解`@PreDestroy` 声明了销毁方法,该方法也会被调用  

 
> 所有的 `BeanPostProcessor` 理论上都是针对所有bean的整个生命周期的    
> 

初始化方法有三种    
- `@PostConstruct` 
- `InitializingBean`接口实现
- `@Bean(initMethod="xxx")`


销毁方法有三种  
- `@PreDestroy`
- `DispoableBean` 接口实现
- `@Bean(destroyMethod="xxx")`  


> - 代码参考   

```java

/**
 * Bean 实例
 */
@Component
public class LifeCycleBean implements BeanNameAware, ApplicationContextAware, BeanFactoryAware, DisposableBean {

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("BeanFactoryAware");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("ApplicationContextAware");
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("BeanNameAware");
    }

    public LifeCycleBean() {
        System.out.println("构造");
    }

    @Autowired
    public void autowire(@Value("${JAVA_HOME}") String name) {
        System.out.println("依赖注入：{}" + name);
    }

    @PostConstruct
    public void init() {
        System.out.println("初始化");
    }

//    @PreDestroy
//    public void destroy() {
//        System.out.println("销毁");
//    }


    @Override
    public void destroy() throws Exception {
        System.out.println("销毁");
    }
}


/**
 * 后置处理器全部
 */
@Component
public class MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {


    @Override
    // 实例化前（即调用构造方法前）执行的方法
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean"))
            System.out.println("实例化前执行,基于InstantiationAwareBeanPostProcessor");
        // 返回null保持原有对象不变，返回不为null，会替换掉原有对象
        return null;
    }

    @Override
    // 实例化后执行的方法
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            System.out.println("实例化后执行，这里如果返回 false 会跳过依赖注入阶段，InstantiationAwareBeanPostProcessor");
            // return false;
        }

        return true;
    }

    @Override
    // 依赖注入阶段执行的方法
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean"))
            System.out.println("依赖注入阶段执行，如@Autowired、@Value、@Resource");
        return pvs;
    }

    @Override
    // 初始化之前执行的方法
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean"))
            System.out.println("初始化之前执行，这里返回的对象会替换掉原本的bean，如 @PostConstruct、@ConfigurationProperties");
        return bean;
    }

    @Override
    // 初始化之后执行的方法
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean"))
            System.out.println("初始化之后执行，这里返回的对象会替换掉原本的bean，如 代理增强");
        return bean;
    }
    @Override
    // 销毁前执行的方法
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean"))
            System.out.println("销毁之前执行");
    }
}
```






