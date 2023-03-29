
# springboot 优点  
开箱即用 内嵌服务器 运行监控   

# 核心注解   

Springboootconfiguation

enableAutoconfiguation  
componentscan  


# 默认日志   
logback   
log4j2 可以进行配置  


# 读取配置的注解   
PropertySource  
Value   
ConfigurationProperties

# 其他注解  
@ImportResource 导入xml


# 自动配置原理 
@EnableAutoConfiguration -> @AutoConfigurationPackage  ->
利用 Registrar 给容器中导入多个组件(Application 所在包下的所有组件)

通过 META-INF/spring.factories 位置来加载一个文件,默认扫描当前系统所有META-INF/spring.factories 位置的文件

文件内部配置了自动加载的类   

同时搭配 Conditional 条件装配 完成相关组件的自动加载，如AOP 只有存在Advice.class类存在才会自动配置



# 配置文件加载顺序 
- properties
- yaml 
- 系统环境变量
- 命令行参数 


# 核心配置文件   
bootstrap.yml 或properties 由Bootstrap Context 加载比application.yml优先 

bootstrap.yml是系统级别配置   
application 是应用级别的配置   

bootstrap 先加载 所以可以被application  覆盖   


# spring profiles 条件装配  
根据条件 注册不同的bean，加载不同的配置文件  

dev test prod   @Profile("test")

通过 spring.active.profile=prod 指定 或者运行时参数--spring.profiles.active=prod



# 全局异常处理 
controllerAdvice

# 定时任务
scheduled

























