#  什么是spring
一个框架 容器 一套生态

# IOC 控制反转？
Ioc是一种设计思想，将对象的创建和调用交由spring进行管理(声明周期)
DI依赖注入是Ioc的一种实现方式，通过依赖注入装配对象
应用程序无需再代码中new相关对象，由IOC进行组装，核心就是Beanfactory

使用目的，降低耦合，简化管理对象

实现机制：工厂模式+ 反射  

IOC 负责对象的实例化、对象的初始化、对象和对象之间依赖关系配置、对象的销毁、对外提供对象的查找,对象的整个生命周期都是由容器来控制的。这个 IOC 是一个具有依赖注入功能的 容器  

# IOC的理解 
控制反转 ： ioc容器控制对象   我们主动创建依赖对象反转成 ioc创建依赖对象




# AOP 面向切面编程？
对业务逻辑的各个部分进行隔离，降低业务耦合，为解耦而生   
不修改源代码，完善、增强功能，模块化功能    

实际应用于 日志处理  权限处理  等

# AOP是如何做的  

原有逻辑抽象出变成切面，注入到目标对象中去，通过动态代理，将需要注入的切面进行代理，调用时直接将公共逻辑添加进去，不需要修改原有业务代码

# AOP名词  

连接点，类中可以被增强的方法叫做连接点    
切入点，真正被增强的方法  
通知（增强），实际增强的逻辑部分称为通知    
切面，动词，把通知应用到切入点的过程


# 通知类型  

前置  
后置  
环绕   
返回  
异常  


# Spring 常用模块
spring core  spring beans   spring context  spring jdbc  spring aop


# spring 中的设计模式 
工厂 beanfactory applicationcontext
单例  bean 
代理 aop 动态代理
模板方法  resttemplate, refresh()
观察者 EventListener applicationEvent


# 什么是springcontext 

基本模块 提供基础功能  ApplicationContext 接口表示Spring IoC容器，负责实例化、配置和组装bean。     
容器通过读取配置元数据获取关于要实例化、配置和组装哪些对象的指令    
Spring提供了ApplicationContext接口的几个实现    
ClassPathXmlApplicationContext或FileSystemXmlApplicationContext，用于加载bean



# Beanfactory 和 applicationContext
ApplicationContext 对 BeanFactory 功能进行了扩展,通过继承不同的接口,可以实现功能的进一步扩展   
ApplicationContext提供了更多功能  国际化，获取环境变量 配置信息
ApplicationContext 间接继承了 beanfactory   

ApplicationContext 启动就加载bean  beanfactory getbean 才进行加载实例化


# applicationContext的通常实现  

ClassPathXmlApplicationContext:
从类路径下的一个或多个Xml配置文件中加载上下文定义

FileSystemXmlApplicationContext:
从文件系统下的一个或多个Xml配置文件加载

AnnotationConfigApplicationContext:
从一个或多个java 配置类中加载Spring应用上下文


# 依赖注入  
类之间的依赖关系由容器来负责。简单来讲a依赖b，但a不创建（或销毁）b，仅使用b，b的创建（或销毁）交给容器

# 依赖注入方式  
构造器注入 (非部分注入，有循环依赖问题，用构造器参数实现强制依赖)
setter方法注入  (部分注入，没有循环依赖，可选依赖 灵活) 

属性注入 垃圾


# spring bean是什么
Spring IoC 容器管理的对象,Spring创建的对象   

bean中包括 配置元数据，构建方法，生命周期详情，依赖情况

# 配置bean
基于xml配置
基于注解配置(xml基础上+注解)
基于java 配置 (@Configuration @Bean)等注解

# spring 配置文件包含那些信息
xml文件包含 类信息，bean注入信息 bean之间的相互调用


# xml注入bena方式 
有参构造注入  `<constructor-arg name="oname" value="Hello"></constructor-arg>`
set注入  ` <property name="bname" value="Hello"></property>`
P名称空间 ` <bean id="book" class="com.atguigu.spring5.Book" p:bname="very" p:bauthor="good">`

# 类的作用域
bean  通过 scope属性 prototype 多例 每次调用返回新的bean   singleton 单例 

# bean 的其他作用域 
singleton 默认    
prototype  
request  1个http请求 1个bean
session  1个httpsession 1个bean
global-session  

# bean 单例是否线程安全  
不安全，安全取决有bean对象有无状态(内部有无存储数据) 如果有数据可能修改 就是线程不安全的   

没有内部存储数据，仅仅调用方法是线程安全的   

尽量不要在Bean中声明有状态的变量  

##  bean 线程安全解决方法
使用ThreadLocal，把变量变成线程私有的 (首选)  
synchronized、lock、CAS，保证互斥访问临界区    
把bean从单例（singleton）设置成原型（protopyte）    


# bean生命周期 


# 重要的生命周期方法   

初始化 
@postconstruct   
init-method   
InitializingBean接口的 afterPropertiesSet 

@PreDestroy   
DispoableBean 接口实现 destroy()  
destroyMethod   


# 内部bean
当一个bean仅被用作另一个bean的属性时，才能被声明为一个内部bean。通常是prototype

# spring注入Java集合
xml注入   
<list>类型用于注入一列值，允许有相同的值。   
<set> 类型用于注入一组值，不允许有相同的值。  
<map> 类型用于注入一组键值对，键和值都可以为任意类型。  
<props>类型用于注入一组键值对  

# bean 装配和自动装配
spring容器知道了bean的依赖关系后，把bean组装到一起，

自动装配 根据指定装配规则，（属性名称或者属性类型），spring 自动将匹配的属性值进行注入

# 自动装配方式 

`<bean id="emp" class="com.auto.Emp" autowire="byName">`
byType  constructor 。。。


# autowired自动装配过程 
@Autowired 注解对应的后处理器是 AutowiredAnnotationBeanPostProcessor   
扫描到 autowired resource inject等，自动查找bean 装配属性

1. 类型查找 
2. 类型有一个直接装配 有多个根据名称查找
3. 查找不到抛异常 可以通过 @Autowried(required = false) 设置,如果容器不存在需要的bean就会返回空

# 为什么不推荐 autowired 及 属性注入 
基于属性注入违反设计模式 单一职责 类间耦合过多   
基于属性注入，容易导致bean的初始化失败，java初始化顺序为 静态变量代码块-实例变量初始化语句-构造函数-autowired装配，调用构造方法是 还未进行注入   
autowired 默认byType 容易冲突，还需额外配置

# 能否注入空字符串和null,特殊符号
可以 ，null标签和 cdata

```xml
 <property name="address">
        <null/><!--属性里边添加一个null标签-->
    </property>

<!-- <<南京>> -->
    <property name="address">
            <value><![CDATA[<<南京>>]]></value>
        </property>
```

# component controller repository server区别 
component 通用标记为bean   
controller mvc控制器bean  
service 服务层类使用 指定意图方便区分  
repository dao 

# requestMapping 
可以放在类上 或者方法上 
类 映射url  
方法 映射url 及http方法 


# spring 事务  
ACID    
原子性 要么都成功要么都失败   
一致性 转账有人增加 有人减少  
隔离性 事务之间是相互隔离的  
持久性事务 一旦提交就是永久的修改   

#  编程式事务和声明式事务
声明式事务，1.基于注解方式，    2. 基于xml配置文件方式 底层使用AOP原理


# 事务传播行为 

```java
    @Transactional(propagation = Propagation.REQUIRED,rollbackFor = Exception.class)
    public void addUser(){
        userDao.addUser();
        testService.addTest();
    }

    @Transactional(propagation = Propagation.REQUIRED,rollbackFor = Exception.class)
    public void addTest(){
        testDao.addTest();
        int i = 1/0;
    }
```

- PROPAGATION_`REQUIRED`：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。
  - user 和test 一个出错全部回滚
- PROPAGATION_`SUPPORTS`：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
  - addUser 异常，会回滚。addTest不会滚。不管addUser是否有事务，addTest都以非事务方式运行
- PROPAGATION_`MANDATORY`：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
  - 如果addUser有事务，那么addTest跟addUser在同一个事务。如果addUser没有事务，那么就会报异常
- PROPAGATION_`REQUIRES_NEW`：创建新事务，无论当前存不存在事务，都创建新事务。
  - 回滚互不影响
- PROPAGATION_NOT_`SUPPORTED`：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
  - 如果addUser异常，会回滚。addTest不会滚。不管addUser是否有事务，addTest都以非事务方式运行
- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。
  - addUser的事务内嵌addTest的事务，如果addUser异常回滚，那么addTest也会回滚。如果addTest回滚，addUser不会回滚


# 隔离级别

- 默认 (以数据库为准) 二者冲突以spring为主
- 读未提交
- 读已提交
- 不可重复读
- 序列化

- 脏写
- 脏读
- 不可重复读
- 幻读 

# spring事务的实现原理方式   

添加了 @transanctional后，spring会基于这个类生成一个代理对象，会将这个代理对象作为Bean 当使用这个代理对象的方法时，有事务处理就会先把事务自动提交关闭，然后执行业务逻辑，如果逻辑无异常就直接提交，有异常就进行回滚


# spring事务失效情况   
bean 对象没有被spring管理 没加注解    
方法的访问修饰符不是public    
数据库不支持事务   
异常被捕获，无法识别到异常进行回滚   
传播行为、隔离级别 、超时时间、回滚配置等配置项错误   


# Spring AOP 和 Aspectj aop的区别 
代理模式一般是两种 静态代理和动态代理   
Aspect 是静态代理的代表  
Spring AOP是动态代理的代表   

Aspectj在编译器生成代理类，编译时增强，编译阶段将切面织入java字节码

srping AOP使用动态代理 ，不会修改字节码，临时生成代理对象，对目标方法做增强


# jdk代理和cglib代理  
jdk代理核心就是 为接口提供代理  核心类是 InvocationHandler 和 Proxy类    
InvocationHandler 通过 invoke 方法反射调用目标类代码，proxy 生成目标类代理对象


cglib 代理在内存中构建一个子类对象从而实现对目标对象功能扩展 通过实现MethodInterceptor 接口 重写intercept方法实现



# spring 4 和 spring 5 的区别  
4 后置通知在 返回后通知之前执行  
 异常的话后置通知也在 异常通知前   
    
5 后置通知在 返回后通知之后执行   
异常的话后置通知也在 异常通知后 

总之就是5 之前的 后置通知是先于返回后通知和异常通知的    


# 什么是循环依赖？请你谈谈？  
a依赖b b依赖c  c依赖a   

构造器注入会导致spring无法解决循环依赖，setter注入可以通过spring 三级缓存解决   

多实例 prototype 非单例模式，无法使用三级缓存， 也不支持循环依赖 

# 解释下spring中的三级缓存？
DefaultSingletonBeanRegistry类中的三个map    
第一级缓存：`Map<String, Object> singletonObjects`，成品单例池，常说的 Spring 容器就是指它，我们获取单例 bean 就是在这里面获取的，存放已经经历了完整生命周期的Bean对象   
第二级缓存：`Map<String, Object> earlySingletonObjects`，存放早期暴露出来的Bean对象，Bean的生命周期未结束（属性还未填充完整，可以认为是半成品的 bean）

第三级缓存：`Map<String, ObiectFactory<?>> singletonFactories`，存放可以生成Bean的工厂，用于生产（创建）对象



# 三级缓存分别是什么？三个Map有什么异同？  
第一级缓存：存放的是**已经初始化好了的Bean**，bean名称与bean实例相对应，即所谓的单例池。表示已经经历了完整生命周期的Bean对象

第一级缓存：**存放的是实例化了，但是未初始化的Bean**，bean名称与bean实例相对应。表示Bean的生命周期还没走完（Bean的属性还未填充）就把这个Bean存入该缓存中。也就是实例化但未初始化的bean放入该缓存里

第三级缓存：**表示存放生成bean的工厂，存放的是FactoryBean**，bean名称与bean工厂对应。假如A类实现了FactoryBean，那么依赖注入的时候不是A类，而是A类产生的Bean


# 如何检测是否存在循环依赖？实际开发中见过循环依赖的异常吗？
假如A依赖B,B依赖A,先创建A`doCreateBean()`只实例话但未属性填充的bean实例，并将bean的objectFactory放到三级缓存singletonFactories中提前曝光，     
接着执行`populateBean()`进行属性填充,填充时需要B，在缓存中未找到，所以如A的流程一样创建B的实例，对B进行属性填充时需要A,会从一二三级缓存中依次找，  
如果在三级缓存中找到了A的 ObjectFactory ，则通过执行他的getObject方法获取提前曝光的bean,并将bean放到二级缓存,删除三级缓存的A，然后注入，填充完后对B进行初始化，此时B是一个完整的对象会将他放到一级缓存中，   
B创建完后，A就可进行属性注入了，初始化完后也放到一级缓存中，从而避免了死循环。


# 多例的情况下，循环依赖问题为什么无法解决？
 Spring 框架不会缓存非单例的 bean 实例



































































