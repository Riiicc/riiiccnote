


# Spring 全笔记
> 关于Spring框架的所有笔记汇总整理  

待办  
- https://blog.csdn.net/m0_73687324/article/details/128255689 事务传播参考完善


---

## 参考资料
> - [尚硅谷spring](https://www.bilibili.com/video/BV1Vf4y127N5)
> - [视频笔记](https://blog.csdn.net/weixin_45496190/article/details/107059038)
> - [JavaKeeper](https://javakeeper.starfish.ink/framework/)
> - [SpringInAction5](https://potoyang.gitbook.io/spring-in-action-v5  )
> - `《SpringInAction4》`




##  IOC容器

### 什么是IOC（ Inversion of Control 控制反转）  

> IOC是一种设计思想,将设计好的对象交由容器控制,也就是 把对象创建和对象之间的调用过程，交给Spring进行管理  
> IoC 也称为依赖注入DI(Dependency Injection),或者说 依赖注入是IOC的一种常见方式   
> IoC/DI 思想中，应用程序是被动的，被动的等待 IoC 容器来创建并注入它所需要的资源   
> 使用IOC目的：为了降低耦合度    

> IoC 容器就是具有依赖注入功能的容器。IoC 容器负责实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。   
> **应用程序无需直接在代码中 new 相关的对象**，应用程序由 IoC 容器进行组装。在 Spring 中 BeanFactory 是 IoC 容器的实际代表者  

```java
//IOC底层伪代码   
//配置文件伪代码
/* <bean id="dao" class="com.ric.UserDao"> */

class UserFactory{
    String classValue = class 属性值; //class="com.ric.UserDao"
    //反射获取对象
    Class calz = Class.forName(classValue);
    return (UserDao)claz.newInstance();
}
```

### IOC核心接口 
- `BeanFactory` ：Spring 实例化、配置和管理对象的最基本接口,IOC容器中基本实现，一般不用，
> 加载配置文件时不会创建对象，在获取创建对象才会创建

- `ApplicationContext` ： `BeanFactory` 的子接口，提供更加强大的功能，开发使用  
> 加载配置文件就会创建对象，一般使用这种方法，项目启动就会创建对应的对象    

> `BeanFactory` 是 Spring 框架的基础设施，面向 Spring 本身；    
> `ApplicationContext` 面向使用 Spring 框架的开发者，几乎所有的应用场合都直接使用 `ApplicationContext` 而非底层的 `BeanFactory`


- 常见的`ApplicationContext` 实现  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sp1.png)
  - `FileSystemXmlApplicationContext` 根据物理路径加载
  - `ClassPathXmlApplicationContext`  根据 `classpath` 加载

### IOC底层
- xml解析
- 工厂模式
- 反射  

### Bean
- Spring IoC 容器管理的对象,Spring创建的对象  

> 配置Bean的方式   
> - 基于xml配置
> - 基于注解配置(xml基础上+注解)
> - 基于java 配置 (@Configuration @Bean)等注解 

> 创建对象默认执行**无参构造方法**完成对象创建，如果没有无参构造器就会报错  


```xml
<bean id="user" class="com.sp.spring.TUser"></bean>
```
- 属性
  - id属性：唯一标识
  - class属性：类全路径
  - name属性 ：可以写特殊符号

```java
//测试代码
public class SpringTest {

    @Test
    public void testSp(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        TUser bean = context.getBean("user3", TUser.class);
        System.out.println(bean);
    }
}
```

### XML操作
- spring 基于xml注入属性 
#### set方式注入

```java

//（1）传统方式： 创建类，定义属性和对应的set方法
public class Book {
        //创建属性
        private String bname;
        private String bauthor;

        //创建属性对应的set方法
        public void setBname(String bname) {
            this.bname = bname;
        }
   }
```
```xml
<!--（2）spring方式： set方法注入属性-->
<!-- 必须存在对应的set方法才可以 -->
<bean id="book" class="com.atguigu.spring5.Book">
    <!--使用property完成属性注入
        name：类里面属性名称
        value：向属性注入的值
    -->
    <property name="bname" value="Hello"></property>
    <property name="bauthor" value="World"></property>
</bean>


```
#### 有参构造器注入
```java
//（1）传统方式：创建类，构建有参函数
public class Orders {
    //属性
    private String oname;
    private String address;
    //有参数构造
    public Orders(String oname,String address) {
        this.oname = oname;
        this.address = address;
    }
  }

```

```xml
<!--（2）spring方式：有参数构造注入属性-->
<!-- 必须存在对应的构造方法才可以 -->
<bean id="orders" class="com.atguigu.spring5.Orders">
    <constructor-arg name="oname" value="Hello"></constructor-arg>
    <constructor-arg name="address" value="China！"></constructor-arg>
</bean>


```

#### p名称空间注入
```xml
<!--1、添加p名称空间在配置文件头部-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"		<!--在这里添加一行p-->

<!--2、在bean标签进行属性注入（算是set方式注入的简化操作）-->
    <bean id="book" class="com.atguigu.spring5.Book" p:bname="very" p:bauthor="good">
    </bean>

```

#### 注入空值和特殊符号

```xml

<bean id="book" class="com.atguigu.spring5.Book">
    <!--（1）null值-->
    <property name="address">
        <null/><!--属性里边添加一个null标签-->
    </property>
    
    <!--（2）特殊符号赋值-->
     <!--属性值包含特殊符号
       a 把<>进行转义 &lt; &gt;
       b 把带特殊符号内容写到CDATA
       c 内容不能换行否则传值也会换行
      -->
        <property name="address">
            <value><![CDATA[<<南京>>]]></value>
        </property>
</bean>
```
#### 注入属性-外部bean
```java
public class UserService {//service类

    //创建UserDao类型属性，生成set方法
    private UserDao userDao;
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void add() {
        System.out.println("service add...............");
        userDao.update();//调用dao方法
    }
}

public class UserDaoImpl implements UserDao {//dao类

    @Override
    public void update() {
        System.out.println("dao update...........");
    }
}

```

```xml
<!--1 service和dao对象创建-->
<bean id="userService" class="com.atguigu.spring5.service.UserService">
    <!--注入userDao对象
        name属性：类里面属性名称
        ref属性：创建userDao对象bean标签id值
    -->
    <property name="userDao" ref="userDaoImpl"></property>
</bean>
<bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>

```
#### 基于XML方式注入内部bean和级联赋值

> 1）一对多关系：部门和员工一个部门有多个员工，一个员工属于一个部门（部门是一，员工是多）   
> 2）在实体类之间表示一对多关系，员工表示所属部门，使用对象类型属性进行表示

```java
//部门类
public class Dept {
    private String dname;
    public void setDname(String dname) {
        this.dname = dname;
    }
}

//员工类
public class Emp {
    private String ename;
    private String gender;
    //员工属于某一个部门，使用对象形式表示
    private Dept dept;
    
    public void setDept(Dept dept) {
        this.dept = dept;
    }
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
}

```

- 注入属性-内部bean


```xml
<!--内部bean-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="Andy"></property>
        <property name="gender" value="女"></property>
        <!--设置对象类型属性-->
        <property name="dept">
            <bean id="dept" class="com.atguigu.spring5.bean.Dept"><!--内部bean赋值-->
                <property name="dname" value="宣传部门"></property>
            </bean>
        </property>
    </bean>

```

-  b）注入属性-级联赋值


```xml
<!--方式一：级联赋值-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="Andy"></property>
        <property name="gender" value="女"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
    </bean>
    <bean id="dept" class="com.atguigu.spring5.bean.Dept">
        <property name="dname" value="公关部门"></property>
    </bean>

 <!--级联赋值-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="jams"></property>
        <property name="gender" value="男"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
        /* 这里需要Emp中生成dept的get方法 否则报错 */
        <property name="dept.dname" value="技术部门"></property>
    </bean>
    <bean id="dept" class="com.atguigu.spring5.bean.Dept">
    </bean>

```
#### IOC 操作 Bean 管理——xml 注入集合属性

> 1、注入数组类型属性   
> 2、注入 List 集合类型属性   
> 3、注入 Map 集合类型属性    

```java
//（1）创建类，定义数组、list、map、set 类型属性，生成对应 set 方法
public class Stu {
    //1 数组类型属性
    private String[] courses;
    //2 list集合类型属性
    private List<String> list;
    //3 map集合类型属性
    private Map<String,String> maps;
    //4 set集合类型属性
    private Set<String> sets;
    
    public void setSets(Set<String> sets) {
        this.sets = sets;
    }
    public void setCourses(String[] courses) {
        this.courses = courses;
    }
    public void setList(List<String> list) {
        this.list = list;
    }
    public void setMaps(Map<String, String> maps) {
        this.maps = maps;
    }


```

```xml
<!--（2）在 spring 配置文件进行配置-->
    <bean id="stu" class="com.atguigu.spring5.collectiontype.Stu">
        <!--数组类型属性注入-->
        <property name="courses">
            <array>
                <value>java课程</value>
                <value>数据库课程</value>
            </array>
        </property>
        <!--list类型属性注入-->
        <property name="list">
            <list>
                <value>张三</value>
                <value>小三</value>
            </list>
        </property>
        <!--map类型属性注入-->
        <property name="maps">
            <map>
                <entry key="JAVA" value="java"></entry>
                <entry key="PHP" value="php"></entry>
            </map>
        </property>
        <!--set类型属性注入-->
        <property name="sets">
            <set>
                <value>MySQL</value>
                <value>Redis</value>
            </set>
        </property>
</bean>

```

#### 在集合里面设置对象类型值
```java
  //学生所学多门课程
    private List<Course> courseList;//创建集合
    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }

```

```xml

    <!--创建多个course对象-->
    <bean id="course1" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="Spring5框架"></property>
    </bean>
    <bean id="course2" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="MyBatis框架"></property>
    </bean>
    
   	<!--注入list集合类型，值是对象-->
       <property name="courseList">
           <list>
               <ref bean="course1"></ref>
               <ref bean="course2"></ref>
           </list>
       </property>

```
#### 在 spring 配置文件中引入名称空间 util

```xml
<!--第一步：在 spring 配置文件中引入名称空间 util-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util" <!--添加util名称空间-->
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">  <!--添加util名称空间-->
    
<!--第二步：使用 util 标签完成 list 集合注入提取-->
<!--把集合注入部分提取出来-->
 <!--1 提取list集合类型属性注入-->
    <util:list id="bookList">
        <value>易筋经</value>
        <value>九阴真经</value>
        <value>九阳神功</value>
    </util:list>

 <!--2 提取list集合类型属性注入使用-->
    <bean id="book" class="com.atguigu.spring5.collectiontype.Book" scope="prototype">
        <property name="list" ref="bookList"></property>
    </bean>

```

### IOC 操作 Bean 管理（FactoryBean） 

> 1、Spring 有两种类型 bean，一种普通 bean，另外一种工厂 bean（FactoryBean）  
> 2、普通 bean：在配置文件中定义 bean 类型就是返回类型   
> 3、工厂 bean：在配置文件定义 bean 类型可以和返回类型不一样 第一步 创建类，让这个类作为工厂 bean，实现接口 FactoryBean 第二步 实现接口里面的方法，在实现的方法中定义返回的 bean 类型   

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="mybean" class="com.factory.MyBean"> </bean>
</beans>
```

```java
public class MyBean implements FactoryBean<Course> {

    @Override
    public Course getObject() throws Exception {
        Course course = new Course();
        return course;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }
}

//测试方法
    @Test
    public void testSpFac(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
        Course course = context.getBean("mybean", Course.class);
        System.out.println(course);
    }

```

### IOC 操作 Bean 管理（bean 作用域）
在 Spring 里面，默认情况下，bean 是单实例对象，下面进行作用域设置： 

> （1）在 spring 配置文件 bean 标签里面有属性（scope）用于设置单实例还是多实例    
> （2）scope 属性值 第一个值 默认值，singleton，表示是单实例对象 第二个值 prototype，表示是多实例对象   


```xml
<bean id="book" class="com.atguigu.spring5.collectiontype.Book" scope="prototype"><!--设置为多实例-->
    <property name="list" ref="bookList"></property>
</bean>
```

> 3）singleton 和 prototype 区别   
> > a）singleton 单实例，prototype 多实例    
> > b）设置 scope 值是 singleton 时候，**加载 spring 配置文件**时候就会创建单实例对象 ；设置 scope 值是 prototype 时候，不是在加载 spring 配置文件时候创建对象，**在调用 getBean 方法时候创建多实例对象**    

**Spring 中其他的几种作用域**    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/spring1.png)

---
###  IOC 操作 Bean 管理（bean 生命周期）
> 1、生命周期 ：从对象创建到对象销毁的过程    
> 2、bean 生命周期   
>（1）通过构造器创建 bean 实例（无参数构造）  
>（2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）  
>（3）调用 bean 的初始化的方法（需要进行配置初始化的方法）  
>（4）bean 可以使用了（对象获取到了）  
>（5）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）  
```java
        public class Orders {
         //无参数构造
         public Orders() {
         System.out.println("第一步 执行无参数构造创建 bean 实例");
         }
         private String oname;
         public void setOname(String oname) {
         this.oname = oname;
         System.out.println("第二步 调用 set 方法设置属性值");
         }
         //创建执行的初始化的方法
         public void initMethod() {
         System.out.println("第三步 执行初始化的方法");
         }
         //创建执行的销毁的方法
         public void destroyMethod() {
         System.out.println("第五步 执行销毁的方法");
         }
        }

```
```xml
    <bean id="orders" class="com.factory.Order" init-method="initMethod" destroy-method="destroyMethod">
        <property name="name" value="手机"></property>
    </bean>
<!-- 后置处理器 自动识别 为当前所有bean都会添加后置处理器 -->
    <bean id="beanPost" class="com.factory.MyBeanPost"></bean>
```
```java
 @Test
 public void testBean3() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml");
    Orders orders = context.getBean("orders", Orders.class);
    System.out.println("第四步 获取创建 bean 实例对象");
    System.out.println(orders);
    //手动让 bean 实例销毁
    context.close();
 }

```
#### bean 的后置处理器，bean 生命周期有七步  
> （正常生命周期为五步，而配置后置处理器后为七步）   
>（1）通过构造器创建 bean 实例（无参数构造）  
>（2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）  
>（3）把 bean 实例传递 bean 前置处理器的方法 `postProcessBeforeInitialization`  
>（4）调用 bean 的初始化的方法（需要进行配置初始化的方法）  
>（5）把 bean 实例传递 bean 后置处理器的方法 `postProcessAfterInitialization`  
>（6）bean 可以使用了（对象获取到了）  
>（7）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）  
 
```java
public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化前");
        return null;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化后");
        return null;
    }
}
```

### IOC 操作 Bean 自动装配 
> 自动装配 根据指定装配规则，（属性名称或者属性类型），spring 自动将匹配的属性值进行注入   
> 实际用的少大部分用注解   


```xml
<!-- 手动装配 -->
    <bean id="emp" class="com.auto.Emp">
        <property name="dept" ref="dept"></property>
    </bean>
    <bean id="dept" class="com.auto.Dept"></bean>

    <!-- 实现自动装配
        bean标签 属性 autowire 配置自动装配
        autowire 属性两个常用值 byName根据属性名称注入，byType根据属性类型注入 
        注意：bean的id要和对应属性名称相同
          -->

    <bean id="emp" class="com.auto.Emp" autowire="byName">
        <!--<property name="dept" ref="dept"></property>-->
    </bean>
    <bean id="dept" class="com.auto.Dept"></bean>

    <!-- byType 相同类型不能用多个bean -->

```
### IOC 操作 Bean 管理(外部属性文件)
- 方式一：直接配置数据库信息  


```xml
<!--直接配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>

```

- 引入外部属性文件配置数据库  

> （1）创建外部属性文件，properties 格式文件，写数据库信息（jdbc.properties）  

```properties
    prop.driverClass=com.mysql.jdbc.Driver
    prop.url=jdbc:mysql://localhost:3306/userDb
    prop.userName=root
    prop.password=root

```
> （2）把外部 properties 属性文件引入到 spring 配置文件中 —— 引入 context 名称空间
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"><!--引入context名称空间-->
    
        <!--引入外部属性文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.password}"></property>
    </bean>
    
</beans>

```

### 注解 IOC容器 Bean 管理——基于注解方式
#### 什么是注解：
> 1. 注解是代码特殊标记 `@注解名称（属性名称=属性值，属性名称=属性值...）`
> 2. 使用注解，注解可以作用在类上面，方法上面，属性上面
> 3. 使用注解目的：简化 xml 配置

#### Spring 针对 Bean 管理中创建对象提供注解  
> 基于注解创建 bean 实例
- `@Component`
- `@Service`
- `@Controller`
- `@Repository`

> 基于注解方式实现对象创建  
- 第一步 引入依赖 （引入spring-aop jar包）
- 第二步 开启组件扫描  

```xml
<!--开启组件扫描
 1 如果扫描多个包，多个包使用逗号隔开
 2 扫描包上层目录
-->
<context:component-scan base-package="com.atguigu"></context:component-scan>
<context:component-scan base-package="com.atguigu,com.ra"></context:component-scan>

```

- 第三步 创建类，在类上面添加创建对象注解

```java
//在注解里面 value 属性值可以省略不写，
//默认值是类名称，首字母小写
//UserService --> userService
@Component(value = "userService") //注解等同于XML配置文件：<bean id="userService" class=".."/>
public class UserService {
 public void add() {
 System.out.println("service add.......");
 }
}

```
- 开启组件扫描细节配置

```xml
<!--示例 1
 use-default-filters="false" 表示现在不使用默认 filter，自己配置 filter
 context:include-filter ，设置扫描哪些内容
-->

    <context:component-scan base-package="com.zhujie" use-default-filters="false">
    <!-- 不扫描有Component注解的类 -->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Component"/>
        <!-- 只扫描有Component注解的类  这两个配置不能同时存在-->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
    </context:component-scan>
<!--示例 2
 下面配置扫描包所有内容
 context:exclude-filter： 设置哪些内容不进行扫描
-->
<context:component-scan base-package="com.atguigu">
 <context:exclude-filter type="annotation"

expression="org.springframework.stereotype.Controller"/><!--表示Controller注解的类之外一切都进行扫描-->
</context:component-scan>

```
> 基于注解方式实现属性注入
- `@Autowired`  
- `@Qualifier`   
- `@Resource`  

> `@Autowired`：根据属性类型进行自动装配  
> `@Qualifier`：根据属性名称进行注入 需要和上面的一起使用  
> `@Resource`：`JSR-250`可以根据属性名称或者类型进行注入   
> `@Inject` :`JSR-330` 根据属性类型进行自动装配  
> `@Value` : 注入普通类型属性  

> 第一步 把 service 和 dao 对象创建，在 service 和 dao 类添加创建对象注解   
> 第二步 在 service 注入 dao 对象，在 service 类添加 dao 类型属性，在属性上面使用注解    

```java
@Service
public class UserService {
 //定义 dao 类型属性
 //不需要添加 set 方法
 //添加注入属性注解
 @Autowired
 private UserDao userDao;
 public void add() {
 System.out.println("service add.......");
 userDao.add();
 }
}

//Dao实现类
@Repository
//@Repository(value = "userDaoImpl1")
public class UserDaoImpl implements UserDao {
    @Override
    public void add() {
        System.out.println("dao add.....");
    }
}

```
> @Qualifier ：根据名称进行注入，这个@Qualifier 注解的使用，和上面@Autowired 一起使用  

```java
//定义 dao 类型属性
//不需要添加 set 方法
//添加注入属性注解
@Autowired //根据类型进行注入
//根据名称进行注入（目的在于区别同一接口下有多个实现类，根据类型就无法选择，从而出错！）
@Qualifier(value = "userDaoImpl1") 
private UserDao userDao;

```
> @Resource：可以根据类型注入，也可以根据名称注入（**它属于javax包下的注解**，不推荐使用！） 

```java
//@Resource //根据类型进行注入
@Resource(name = "userDaoImpl1") //根据名称进行注入
private UserDao userDao;

```
> @Value：注入普通类型属性  

```java
@Value(value = "abc")
private String name

```
#### 全部使用注解
> 1）创建配置类，替代 xml 配置文件

```java
@Configuration //作为配置类，替代 xml 配置文件
@ComponentScan(basePackages = {"com.atguigu"})
public class SpringConfig {
    
}

```

> 2）编写测试类  `AnnotationConfigApplicationContext`

```java
@Test
public void testService2() {
 //加载配置类
 ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
 UserService userService = context.getBean("userService",UserService.class);
 System.out.println(userService);
 userService.add();
}

```

### 循环依赖问题   

> 所谓的循环依赖是指，A 依赖 B，B 又依赖 A，它们之间形成了循环依赖。或者是 A 依赖 B，B 依赖 C，C 又依赖 A，形成了循环依赖。更或者是自己依赖自己

- Spring 为了解决单例的循环依赖问题而设计了**三级缓存**    

```java
/** Cache of singleton objects: bean name --> bean instance 一级 */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name --> ObjectFactory 三级 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name --> bean instance 二级 */ 
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```

- `singletonObjects` 用于存放完全初始化好的bean，从该缓存中取出的bean可以直接使用
- `earlySingletonObjects` 存放原始bean对象(尚未填充属性),用于解决循环依赖
- `singletonFactories` 存放bean工厂对象,用于解决循环依赖 


1. A创建过程中需要B，于是先将A放到三级缓存，去实例化B。
2. B实例化的过程中发现需要A，于是B先查一级缓存寻找A，如果没有，再查二级缓存，如果还没有，再查三级缓存，找到了A，然后把三级缓存里面的这个A移动到二级缓存里面，并删除三级缓存里面的A。
3. B顺利初始化完毕，将自己放到一级缓存里面（此时B里面的A依然是创建中的状态）。然后回来接着创建A，此时B已经创建结束，可以直接从一级缓存里面拿到B，去完成A的创建，并将A放到一级缓存。


> **非单例Bean**:对于 `prototype` 作用域的 bean，Spring 容器无法完成依赖注入，因为 Spring 容器不进行缓存 prototype 作用域的 bean ，因此无法提前暴露一个创建中的bean    

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
             // 三级缓存获取，key=beanName value=objectFactory，objectFactory中存储					//getObject()方法用于获取提前曝光的实例
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    // 三级缓存有的话，就把他移动到二级缓存
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

> 使用构造器注入，循环依赖场景是无法解决的,在实际**开发中要避免循环依赖**    
> 如果A的构造其中依赖了B,B的构造器中又依赖了A ,在 getSingleton 中三级缓存需要调用getObject()构造器构造提早暴露但未设置属性的bean  

> PS:Spring Boot2.6.0 之后默认禁止循环依赖 `spring.main.allow-circular-references=true`

## AOP 切面
### 基本概念
> 1. 面向切面编程（方面），利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得 业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
> 2. 通俗描述：不通过修改源代码方式，在主干功能里面添加新功能

### AOP（底层原理）
a）AOP 底层使用动态代理 ，动态代理有两种情况：

1. 有接口的情况，使用JDK动态代理

> 创建接口实现类的代理对象增强类的方法
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/20200702135134128.png)  


2. 没有接口情况，使用Cglib代理  
> 使用 CGLIB 动态代理；创建子类的代理对象，增强类的方法
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/2020070213514980.png)


### AOP（JDK 动态代理）
> 使用 JDK 动态代理，使用 Proxy 类里面的方法创建代理对象  
> 调用 newProxyInstance 方法，方法有三个参数：
```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
```
> 第一个参数：类加载器
> 第二个参数：增强方法所在的类，这个类实现的接口，支持多个接口
> 第三个参数：InvocationHandler 实现这个接口，床架你代理对象，写增强方法

### JDK动态代理代码  
```java
//（1）创建接口，定义方法
public interface UserDao {
 public int add(int a,int b);
 public String update(String id);
}

```
```java
//（2）创建接口实现类，实现方法
public class UserDaoImpl implements UserDao {
 @Override
 public int add(int a, int b) {
 return a+b;
 }
 @Override
 public String update(String id) {
 return id;
 }
}

```

```java
//（3）使用 Proxy 类创建接口代理对象
public class JDKProxy {
 public static void main(String[] args) {
 //创建接口实现类代理对象
 Class[] interfaces = {UserDao.class};
 UserDaoImpl userDao = new UserDaoImpl(); 
/** 第一参数，类加载器 
	第二参数，增强方法所在的类，这个类实现的接口，(支持多个接口)
	第三参数，实现这个接口 InvocationHandler，创建代理对象，写增强的部分  */
 UserDao dao =(UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces,
					new UserDaoProxy(userDao));
 int result = dao.add(1, 2);
 System.out.println("result:"+result);
 }
}

//创建代理对象代码
class UserDaoProxy implements InvocationHandler {
 //1 把创建的是谁的代理对象，把谁传递过来
 //有参数构造传递
 private Object obj;
 public UserDaoProxy(Object obj) {
 this.obj = obj;
 }
 //增强的逻辑
 @Override
 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
 //方法之前
 System.out.println("方法之前执行...."+method.getName()+" :传递的参数..."+ Arrays.toString(args));
 //被增强的方法执行
 Object res = method.invoke(obj, args);
 //方法之后
 System.out.println("方法之后执行...."+obj);
 return res;
 }
}

```
### AOP（术语）
> 1. 连接点，类中可以被增强的方法叫做连接点
> 2. 切入点，真正被增强的方法
> 3. 通知（增强），实际增强的逻辑部分称为通知
> > 通知有多种类型，前置通知，后置通知，环绕通知，异常通知，最终通知
> 4. 切面，动词，把通知应用到切入点的过程

### AOP操作
> Spring 框架一般基于AspectJ实现AOP操作
> AspectJ不是Spring的组成部分，独立AOP框架，一般把AspectJ和Spring一起使用，进行AOP操作
> 1.基于xml方式实现 
> 2.基于注解方式实现   

> 切入点表达式，如下：
```java
（1）切入点表达式作用：知道对哪个类里面的哪个方法进行增强 
（2）语法结构： execution([权限修饰符] [返回类型] [类全路径] [方法名称]([参数列表]) )
（3）例子如下：
    例 1：对 com.atguigu.dao.BookDao 类里面的 add 进行增强
		execution(* com.atguigu.dao.BookDao.add(..))
 	例 2：对 com.atguigu.dao.BookDao 类里面的所有的方法进行增强
		execution(* com.atguigu.dao.BookDao.* (..))
    例 3：对 com.atguigu.dao 包里面所有类，类里面所有方法进行增强
		execution(* com.atguigu.dao.*.* (..))

```

```java
//1、创建类，在类里面定义方法
public class User {
 public void add() {
 System.out.println("add.......");
 }
}
//2、创建增强类（编写增强逻辑）
//（1）在增强类里面，创建方法，让不同方法代表不同通知类型
//增强的类
public class UserProxy {
 public void before() {//前置通知
 System.out.println("before......");
 }
}


```

```xml
<!--3、进行通知的配置-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.atguigu.spring5.aopanno"></context:component-scan>

    <!-- 开启Aspect生成代理对象-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>

```

```java

//增强的类
@Component
@Aspect  //生成代理对象
public class UserProxy {}

//被增强的类
@Component
public class User {}

```

```java
//4、配置不同类型的通知
@Component
@Aspect  //生成代理对象
public class UserProxy {
      //相同切入点抽取
    @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void pointdemo() {

    }

    //前置通知
    //@Before注解表示作为前置通知
    @Before(value = "pointdemo()")//相同切入点抽取使用！
    public void before() {
        System.out.println("before.........");
    }

    //后置通知（返回通知）
    @AfterReturning(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterReturning() {
        System.out.println("afterReturning.........");
    }

    //最终通知
    @After(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void after() {
        System.out.println("after.........");
    }

    //异常通知
    @AfterThrowing(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterThrowing() {
        System.out.println("afterThrowing.........");
    }

    //环绕通知
    @Around(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前.........");

        //被增强的方法执行
        proceedingJoinPoint.proceed();

        System.out.println("环绕之后.........");
    }
}


```
### 有多个增强类对同一个方法进行增强，设置增强类优先级
```java
//（1）在增强类上面添加注解 @Order(数字类型值)，数字类型值越小优先级越高
@Component
@Aspect
@Order(1)
public class PersonProxy{ }

```
### AOP 操作（AspectJ 配置文件）
```xml
<!--1、创建两个类，增强类和被增强类，创建方法（同上一样）-->
<!--2、在 spring 配置文件中创建两个类对象-->
<!--创建对象-->
<bean id="book" class="com.atguigu.spring5.aopxml.Book"></bean>
<bean id="bookProxy" class="com.atguigu.spring5.aopxml.BookProxy"></bean>
<!--3、在 spring 配置文件中配置切入点-->
<!--配置 aop 增强-->
<aop:config>
 <!--切入点-->
 <aop:pointcut id="p" expression="execution(* com.atguigu.spring5.aopxml.Book.buy(..))"/>
 <!--配置切面-->
 <aop:aspect ref="bookProxy">
 <!--增强作用在具体的方法上-->
 <aop:before method="before" pointcut-ref="p"/>
 <aop:after method="After" pointcut-ref="p"></aop:after>
 </aop:aspect>
</aop:config>


```
### 注意
> 注意：xml配置`<aop:aspectj-autoproxy></aop:aspectj-autoproxy>`时after和afterReturning执行顺序是：afterReturning > after
> 注解配置`@EnableAspectJAutoProxy`@after和@afterReturning的执行顺序是：after > afterReturning
> 这个在数据库事务rollback会产生影响。



## Spring 事务
### 事务
> 事务是数据库最基本的操作单元，一组逻辑操作，要么都成功，有一个失败都失败
> 典型场景：银行转账 
> 事务四个特性（ACID）：
> - 原子性
> - 一致性
> - 隔离性
> - 持久性

### 事务操作
1. 事务添加到service层
2. Spring 进行事务管理操作，有两种方式：编程式事务和**声明式事务**

> 声明式事务，1.基于注解方式，2. 基于xml配置文件方式 底层使用AOP原理

`PlantformTransactionManager`

### 声明式（注解）
1. 配置事务管理器
```xml
    <!--配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!-- 获取properties文件内容，根据key获取，使用spring表达式获取 -->
        <property name="driverClassName" value="${prop.driverClass}"/>
        <property name="url" value="${prop.url}"/>
        <property name="username" value="${prop.username}"/>
        <property name="password" value="${prop.password}"/>
    </bean>
    <!-- 创建事务管理器  -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入 数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>
```

2. 引入tx名称空间 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 开启扫描   -->
    <context:component-scan base-package="com.Dancebytes"></context:component-scan>

    <!--引入外部属性文件-->
    <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>

    <!--配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!-- 获取properties文件内容，根据key获取，使用spring表达式获取 -->
        <property name="driverClassName" value="${prop.driverClass}"/>
        <property name="url" value="${prop.url}"/>
        <property name="username" value="${prop.username}"/>
        <property name="password" value="${prop.password}"/>
    </bean>

    <!-- JdbcTemplate对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--注入dataSource-->
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!-- 创建事务管理器  -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入 数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 开启 事务 注解  -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

3. 在service类中加入事务注解   

> `@Transactional` 可以加到类上，也可以加到方法上
> 如果加到类上，那么类中所有方法都会添加事务

### 事务操作（注解参数配置）
`@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.REPEATABLE_READ, timeout = -1, readOnly = false, rollbackFor = {}, noRollbackFor = {})`

- `propagation` **事务传播行为** 7种 默认为`Propagation.REQUIRED`
> 多事务方法直接进行调用，这个过程中事务是如何进行管理的，如：有事务方法调用无事务方法
> **前两个重要**

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/spring事务.png)

- isolation 事务隔离级别
> 事务的隔离特性，多事务操作之间不会产生影响，不考虑隔离性会产生很多问题
> 譬如：脏读，不可重复读，幻读 
> 通过设置事务隔离级别
> `READ_UNCOMMITTED` 读未提交  有脏读，有不可重复读，有幻读
> `READ_COMMITTED` 读已提交  无脏读，有不可重复读，有幻读
> `REPEATABLEREAD` 可重复读  无脏读，无不可重复读，有幻读
> `SERIALIZABLE` 串行  无脏读，无不可重复读，无幻读
- timeout 超时时间 
> 默认值是-1 设置值以秒为单位  

- readOnly 是否只读
> 默认值false ，设置为true之后只能查询操作不能修改  

- rollbackFor 回滚
> 设置出现那些异常进行回滚

- noRollbackFor 不回滚
> 设置出现那些异常不进行回滚

> 在项目中，@Transactional(rollbackFor=Exception.class)，如果类加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。在@Transactional注解中如果不配置rollbackFor属性,那么事物只会在遇到`RuntimeException`的时候才会回滚,加上rollbackFor=Exception.class,可以让事物在遇到非运行时异常时也回滚,`IOException`

### 基于xml实现声明式事务管理

```xml
<!-- 创建事物管理器  -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入 数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 开启 事物 注解  -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
    <!--配置通知-->
    <tx:advice id="txadvice">
        <!--配置事务参数-->
        <tx:attributes>
            <!--指定那个方法上添加事务-->
            <tx:method name="add" isolation="REPEATABLE_READ" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    <!--切入点配置和切面-->
    <aop:config>
        <!--配置切入点-->
        <aop:pointcut id="pt" expression="execution(* com.Dancebytes.spring5.dao.UserDao.*(..))"/>
        <!--配置切面-->
        <aop:advisor advice-ref="txadvice" pointcut-ref="pt"/>
    </aop:config>
```
### 完全注解方式 实现声明式事务管理（主要就是注入bean 和配置）
```java

package config;


@Configuration// 声明是 配置类
@ComponentScan(basePackages = "com.Dancebytes")// 开启扫描
@EnableTransactionManagement// 开启事物
public class TXConfig {
    //创建数据库的 连接池
    @Bean
    public DruidDataSource getDruidDataSource() throws IOException {
        // 1.读取配置文件中的4个基本信息
        InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
        Properties pros = new Properties();
        pros.load(is);
        String driverClass = pros.getProperty("prop.driverClass");
        String url = pros.getProperty("prop.url");
        String user = pros.getProperty("prop.username");
        String password = pros.getProperty("prop.password");

        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName(driverClass);
        druidDataSource.setUrl(url);
        druidDataSource.setUsername(user);
        druidDataSource.setPassword(password);
        return druidDataSource;
    }

    //JdbcTemplate对象
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

    //创建事物管理器
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }

}

```
## Spring5 新特性
### Nullable
> @Nullable 注解 可以用在属性，参数，方法(返回值可以为空)上 
> 如果可以传入NULL值，则标记为@Nullable，如果不可以，则标注为@Nonnull。那么在我们做一些不安全严谨操作的编码操作时，这些注释会给我们一些警告。




### 函数式创建对象 交给Spring进行管理
```java
    //函数式创建对象 交给Spring进行管理
    @Test
    public void test3(){
        //1 创建GenericApplicationContext对象
        GenericApplicationContext context = new GenericApplicationContext();
        //2 调用context的方法对象注册
        context.refresh();
//        context.registerBean(User.class, () ->new User());
        context.registerBean("user1", User.class, () ->new User());
        //3 获取在spring注册的对象
//        User user = (User)context.getBean("com.Dancebytes.spring5.test.User");
        User user1 = (User)context.getBean("user1");
        System.out.println(user1);
    }
```

### JUnit5
```java
@RunWith(SpringJUnit4ClassRunner.class) //单元测试框架
@ContextConfiguration("classpath:bean1.xml") //加载配置文件 
public class JTest4 {
    @Autowired
    private UserService userService;

    @Test
    public void test1(){
        userService.accountMoney();

    }
}


//@ExtendWith(SpringExtension.class)
//@ContextConfiguration("classpath:bean1.xml")
//下面这个注解相当于上面两个组合
@SpringJUnitConfig(locations = "classpath:bean1.xml")
public class JTest5 {
    @Autowired
    private UserService userService;

    @Test
    public void test1(){
        userService.accountMoney();
    }
}

```

## WebFlux
https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html 

> 异步非阻塞框架,支持响应式编程
> was added later in `version 5.0`. It is fully `non-blocking`, supports `Reactive Streams` back pressure, 
> and runs on such servers as `Netty, Undertow, and Servlet 3.1+` containers

特点:  
- 非阻塞,在有限的资源下提高系统吞吐量和伸缩性,以 `Reactor` 为基础实现响应式编程（面向数据流和变化传播的编程范式） 
- 函数式编程 Spring5框架基于java8 , webflux使用java8 函数式编程方式实现路由请求

### Java8 Observer示例  
> java8 `Observer` 和 `Observable` 
> java9 中的 `Flow` 是 `Reactor` 的标准实现


```java
public class ObserverDemo extends Observable {


    public static void main(String[] args) {
        ObserverDemo observer = new ObserverDemo();
        observer.addObserver((o,a)-> System.out.println("发生变化"));

        observer.addObserver((o,a)-> System.out.println("发生变化1111"));
        //数据变化
        observer.setChanged();
        //通知(所有)观察者
        observer.notifyObservers();

    }
}
```

### 响应式编程(Reactor 实现) 
- 响应式编程操作中,`Reactor` 是满足 `Reactive` 规范框架 
- `Reactor` 有两个核心类 `Mono` 和 `Flux` ,这两个类实现接口 `Publisher` ,提供丰富的操作符,
  - `Flux` 对象实现发布者,返回N个元素
  - `Mono` 对象实现发布者,返回 0 或者 1 个元素
- `Mono` 和 `Flux` 都是数据流的发布者, `Mono` 和 `Flux` 都可以发出三种数据信号: `元素值`,`错误信号`,`完成信号`,**错误信号和完成信号都代表终止信号**,终止信号用于告诉订阅者数据流结束了,错误信号终止数据流同时把错误信息传递给订阅者  

三种信号的特点
- 错误信号和完成信号都是终止信号,不能共存
- 如果没有发送任何元素值,而是直接发送错误或者完成信号,表示是空数据流
- 如果没有错误信号,没有完成信号,表示是无限数据流 



引入依赖
```xml
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
            <version>3.4.9</version>
        </dependency>
```
代码演示
```java
        Flux.just(1,2,3,4);
        Mono.just(1);

        Integer[] array = {1,2,3,4};
        Flux.fromArray(array);

        List<String> strings = new ArrayList<>();
        Flux.fromIterable(strings);

        
        Stream<Integer> integerStream = Stream.of(1, 2, 3);
        //声明数据流
        Flux<Integer> integerFlux = Flux.fromStream(integerStream);
        //订阅数据流
        integerFlux.subscribe(System.out::println);

```
> 调用`just` 或者其他方法只是声明数据流,数据流并没有发出,只有进行订阅之后才会触发数据流.

> 常用 操作符: `map` `flatMap` 参考java8 流

- `SpringWebFlux` 执行流程和核心API
  - `SpringWebFlux` 基于Reactor ,默认容器是Netty,NIO 
  - `SpringWebFlux` 执行过程和SpringMVC很相似
  - `SpringWebFlux` 核心控制器是`DispatchHandler` ,实现接口`WebHandler` 
    - `HandlerMapping` 请求查询到处理的方法
    - `HandlerAdapter` 真正负责请求处理
    - `HandlerResultHandler` 响应结果处理
  - 实现函数式编程两个接口,`RouterFunction` 和 `HandlerFunction` 


### SpringWebFlux 基于注解编程模型
类似于SpringMVC 只需把相关依赖配置到项目中,SpringBoot 自动配置相关运行容器,默认情况下使用Netty服务器 
- `SpringMVC`方式,同步阻塞, 基于 `SpringMVC+Servlet+Tomcat`
- `SpringWebFlux` ,异步非阻塞,基于 `SpringWebFlux+Reactor+Netty` 

代码案例

```java
@Controller
@ResponseBody
public class UserController {

    @Autowired
    private UserService userService;


    //id查询
    @GetMapping("/user/{id}")
    public Mono<User> getUserById(@PathVariable Integer id){

        return userService.getUserById(id);
    }

    //查询所有
    @GetMapping("/alluser")
    public Flux<User> getUser(){
        return userService.getAllUser();
    }


    //保存
    @PostMapping("/saveUser")
    public Mono<Void> saveUser(@RequestBody User user){

        Mono<User> userMono = Mono.justOrEmpty(user);
        return userService.saveUserInfo(userMono);
    }

}

```

```java
@Repository
public class UserServiceImpl implements UserService {


    private final Map<Integer,User> users  = new HashMap<>();

    public UserServiceImpl() {
        this.users.put(1,new User("zhangsn","男",20));
        this.users.put(2,new User("zhangsn1","男1",21));
        this.users.put(3,new User("zhangsn2","男2",22));
        this.users.put(4,new User("zhangsn3","男3",23));
        this.users.put(5,new User("zhangsn4","男4",24));
    }

    @Override
    public Mono<User> getUserById(Integer id) {
        Mono<User> userMono = Mono.justOrEmpty(this.users.get(id));
        return userMono;
    }

    @Override
    public Flux<User> getAllUser() {
        Map<Integer, User> users = this.users;
        Flux<User> tFlux = Flux.fromIterable(users.values());
        return tFlux;
    }

    @Override
    public Mono<Void> saveUserInfo(Mono<User> userMono) {
        Mono<Void> voidMono = userMono.doOnNext(pe -> {
            int i = users.size() + 1;
            this.users.put(i, pe);
        }).thenEmpty(Mono.empty());
        return voidMono;
    }
}

```

### SpringWebFlux 基于函数式编程模型
- 在使用函数式编程模型操作时,需要自己初始化服务器
- 在基于函数式编程模型时,有两个核心接口,
  - `RouterFunction`  实现路由功能,请求转发给对应的handler
  - `HandlerFunction` 处理请求生成响应的函数
- 核心任务就是定义两个函数式接口的实现并且启动需要的服务器
- SpringWebFlux 请求和响应不再是 `ServletRequest` 和 `ServletResponse` 而是 `ServerRequest` `ServerResponse`

示例代码,userservice省略
```java
package com.ric.handler;

import com.ric.entity.User;
import com.ric.service.UserService;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import static org.springframework.web.reactive.function.BodyInserters.fromValue;

/**
 * 类描述
 */
public class UserHandler {

    private final UserService userService;

    public UserHandler(UserService userService) {
        this.userService = userService;
    }

    //根据id查询
    public Mono<ServerResponse> getUserById(ServerRequest request) {
        //获取id值
        Integer id = Integer.valueOf(request.pathVariable("id"));
        // 空值
        Mono<ServerResponse> build = ServerResponse.notFound().build();
        //查询
        Mono<User> userById = this.userService.getUserById(id);
        //把user转换返回
        //使用Reactor操作符返回flatmap
        Mono<ServerResponse> serverResponseMono =
                userById
                        .flatMap(pe -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(fromValue(pe)))
                        .switchIfEmpty(build);
        return serverResponseMono;
    }

    //查询所有
    public Mono<ServerResponse> getAll(ServerRequest request){
        Flux<User> allUser = this.userService.getAllUser();
        Mono<ServerResponse> body = ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(allUser,
                User.class);
        return body;
    }

    //添加
    public Mono<ServerResponse> saveUser(ServerRequest request){
        Mono<User> userMono = request.bodyToMono(User.class);
        return ServerResponse.ok().build(this.userService.saveUserInfo(userMono));
    }
}

```

```java
package com.ric;

import com.ric.handler.UserHandler;
import com.ric.service.UserService;
import com.ric.service.impl.UserServiceImpl;
import org.springframework.http.MediaType;
import org.springframework.http.server.reactive.HttpHandler;
import org.springframework.http.server.reactive.ReactorHttpHandlerAdapter;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.RouterFunctions;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.netty.http.server.HttpServer;

import java.io.IOException;

import static org.springframework.web.reactive.function.server.RequestPredicates.*;
import static org.springframework.web.reactive.function.server.RouterFunctions.toHttpHandler;

/**
 * 运行main方法启动 端口在控制台
 */
public class Server {
    //最终调用
    public static void main(String[] args) throws IOException {
        Server server = new Server();
        server.createReactorServer();
        System.out.println("enter to exit");
        System.in.read();
    }
    //创建Router路由
    public RouterFunction<ServerResponse> routerFunction() {
        UserServiceImpl userService = new UserServiceImpl();
        UserHandler userHandler = new UserHandler(userService);

        RouterFunction<ServerResponse> route = RouterFunctions.route(
                GET("/user/{id}").and(accept(MediaType.APPLICATION_JSON)), userHandler::getUserById)
                .andRoute(GET("/getAll").and(accept(MediaType.APPLICATION_JSON)), userHandler::getAll)
                .andRoute(POST("/saveUser").and(accept(MediaType.APPLICATION_JSON)), userHandler::saveUser);

        return route;
    }
    //创建服务器完成适配
    public void createReactorServer() {
        //路由和handler适配
        RouterFunction<ServerResponse> route = routerFunction();
        HttpHandler httpHandler = toHttpHandler(route);

        ReactorHttpHandlerAdapter reactorHttpHandlerAdapter = new ReactorHttpHandlerAdapter(httpHandler);
        //创建服务器
        HttpServer httpServer = HttpServer.create();
        httpServer.handle(reactorHttpHandlerAdapter).bindNow();
    }
}

```
- 使用WebClient调用

```java
public class RicClient {

    public static void main(String[] args) {
        WebClient webClient = WebClient.create("http://localhost:65414");
        User block = webClient.get().uri("/user/{id}", 1).accept(MediaType.APPLICATION_JSON).retrieve()
                .bodyToMono(User.class).block();
        System.out.println(block);

        Flux<User> userFlux =
                webClient.get().uri("/getAll").accept(MediaType.APPLICATION_JSON).retrieve().bodyToFlux(User.class);
        userFlux.map(User::getName).buffer().doOnNext(System.out::println).blockFirst();
    }
}
```

## 总结
- 概述
  - 轻量级开源javaEE框架,核心 `IOC` `AOP`
- IOC
  - 原理: 工厂 反射
  - IOC接口:`BeanFactory`
  - IOC操作Bean管理(基于xml) 
  - IOC操作Bean管理(基于注解) 

- AOP
  - AOP底层原理,动态代理 有接口jdk动态代理,没有接口cglib动态代理
  - 切面,切入点,增强(通知)
  - 基于AspectJ 实现AOP,注解方式
  - 前置后置环绕异常

- 事务管理
  - 事务概念
  - 隔离级别,传播行为
  - 基于注解实现声明式事务管理
  - 完全注解实现

- Spring 新功能
  - 整合日志框架
  - 相关注解@Nullable 
  - 函数式注册对象
  - JUnit5
  - SpringWebFlux


























































