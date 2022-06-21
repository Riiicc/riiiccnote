# MyBatis 学习

## 核心配置文件 

mybatis-config.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--引入配置文件 -->
    <properties resource="jdbc.properties"/>

    <!--类型别名 ,类型别名不区分大小写 默认为类名,且不区分大小写-->
    <typeAliases>
        <typeAlias type="com.ric.entity.TUser" />
        <!--可以为包下所有类设置默认别名-->
        <package name="com.ric.entity"/>
    </typeAliases>

    <!--配置连接数据库环境
        default 设置默认使用的环境id
     -->
    <environments default="development">
        <!--
        environment 配置某个具体的环境
        属性:
            id 表示连接数据库的唯一环境表示,不能重复
        -->
        <environment id="development">

            <!--
            transactionManager 设置事务管理方式
             属性:
                type: JDBC|MANAGED
                JDBC :当前环境执行sql 使用的是JDBC中原生的事务管理方式,事务的提交和回滚需要手动处理
                MANAGED:被管理 如 Spring
            -->
            <transactionManager type="JDBC"/>

            <!--
            dataSource 配置数据源
                type :设置数据源的类型 POOLED|UNPOOLED|JNDI
                    POOLED:表示使用数据库连接池缓存数据库连接
                    UNPOOLED:表示不使用数据库连接池
                    JNDI:表示使用上下文中的数据源

            -->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>

        <environment id="test">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://121.89.244.48:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="5K5Wnkfk4M$lekFq"/>
            </dataSource>
        </environment>
    </environments>
    
    <!--引入配置文件
            以包为单位引入映射文件
                要求:
                    1.mapper 接口所在包要和映射文件所在包一致,即:*mapper.java和*mapper.xml 文件价路径要相同
                    2.mapper 接口要和映射文件的名字一致
    -->
    <mappers>
        <mapper resource="mappers/TUserMapper.xml"/>
        <!-- <package name="com.ric.mappers"/> -->
    </mappers>
</configuration>
```

## 测试
```java
public class MyBatisTest {

    @Test
    public void test() throws IOException {
        //加载核心配置文件
        InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
        //获取SqlSessionFactoryBuilder,通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory,生产session对象
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        //获取SqlSessionFactory 是生产SQL Session的工厂
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(resourceAsStream);
        //获取SqlSession,代表java程序和数据库之间的会话（如：httpsession是java程序和浏览器之间的会话）
        //通过它去执行语句 true 提交事务 默认不自动提交事务
        SqlSession sqlSession = build.openSession(true);

        //获取mapper接口对象
        TUserMapper mapper = sqlSession.getMapper(TUserMapper.class);
        int i = mapper.insertTUser();
//        mapper.updateTUser();
        //或者使用这种提交事务
//        sqlSession.commit();
        System.out.println(i);

    }

    @Test
    public void testSelect() throws IOException {
        InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = build.openSession(true);
        TUserMapper mapper = sqlSession.getMapper(TUserMapper.class);
        List<TUser> allUser = mapper.getAllUser();
        System.out.println(allUser);

    }
    @Test
    public void testDelete() throws IOException {
        InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = build.openSession(true);
        TUserMapper mapper = sqlSession.getMapper(TUserMapper.class);
        mapper.deleteTUser();

    }
    @Test
    public void testUpdate() throws IOException {
        InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = build.openSession(true);
        TUserMapper mapper = sqlSession.getMapper(TUserMapper.class);
        mapper.updateTUser();

    }

}

```

## SqlSession
```java

public class SqlSessionUtils {

    public static SqlSession getSqlSession() throws IOException {
        InputStream stream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(stream);
        SqlSession sqlSession = build.openSession(true);
        return sqlSession;

    }
}




    @Test
    public void test() throws IOException {

        SqlSession sqlSession = SqlSessionUtils.getSqlSession();
        ParameterMapper mapper = sqlSession.getMapper(ParameterMapper.class);
        List<TUser> allUser = mapper.getAllUser();
        allUser.forEach(System.out::println);

    }
```


## MyBatis 获取参数的两种方式 

> MyBatis 获取参数的两种方式 `${}` `#{}` 

- `${}` 本质是字符串拼接
- `#{}` 本质占位符赋值
- 可以交叉使用,注意`${}` 要使用 单引号包裹

- mapper 接口方法的参数为**单个**的**字面量类型**
    - 参数名称不影响查询(只有一个参数)
    - `${}`需要加单引号(实质是字符串拼接)
`select * from t_user where id= #{id}`
`select * from t_user where id= '${id}'`

- mapper 接口方法参数为**多个字面量类型**
  - 不能使用上面参数名称获取对应值
  - mybatis 会将这些参数放在map集合中,以`arg0,arg1...` `param1,param2...` 为键
  - 可以使用 `arg0/arg1` 或者 `param1/param2` 替代第一个参数和第二个参数即可(二者不可混用)
  - 同样注意`${}`需要加单引号进行访问

- mapper 接口方法参数有多个时,可以手动将这些参数放在一个`map`中进行传参
  - 相当于手动指定上面的键值,键就是参数名称

- mapper 接口方法的参数是实体类类型的参数
  - 只需通过`${}` 或`#{}` 以属性的方式访问即可,注意 `${}` 的单引号问题

- 通过`@Param("username")` 参数注解会将这些参数放在map集合中,自定义访问参数键值(以`username` 作为键) 
  - 同时仍然可以使用 `param1/param2...` 进行访问,并且可以交叉使用自定义名称键值和`param1/param2...`

> 实际开发中建议使用 实体类型或者`@Param` 注解指定传参,这两种情况 
## @Param 源码

## Mybatis 的各种查询功能

- 若查询出的数据只有一条,可以通过实体类对象接收,
  - 也可以通过集合接收(list接收就是只有一个元素的list),
  - 通过map接收空数据字段会被省略
- 若查出多条数据,可以用list集合接收,
  - 也可以使用`Map`
  - `@MapKey("id")` 将id作为键获取,返回Map参考`{1={password=12, sex=, id=1, username=1},2={password=12, sex=, id=2, username=1}}`

```java
@MapKey("id")
Map<String,Object> getAllUserMapKey();
```

```xml
    <select id="getAllUserMapKey" resultType="java.util.Map">
        select * from t_user where username like '%${username}%';
    </select>
```

## Mybatis 内部类型别名
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mybatis1.png)

## Mybatis 特殊SQL

### 模糊查询 
- 三种拼接,建议使用后两种  

```sql
 
 select * from t_user where username like '%${username}%'
 
 select * from t_user where username like "%"#{username}"%"

 select * from t_user where username like concat('%',#{username},'%')

```


### 批量删除
> 不能使用 `in (#{ids})` 

```java
int deleteIds(@Param("ids") String ids);
```

```xml
    <delete id="deleteIds">
        delete from t_user where id in (${ids})
    </delete>
```
### 动态设置表名

```java
    List<TUser> getUserByTableName(@Param("tableName") String tableName);
```

```xml
    <select id="getUserByTableName" resultType="com.ric.entity.TUser">
        select * from ${tableName}
    </select>
```

### 添加记录获取自增主键
- `useGeneratedKeys` 开启自增,标识其使用了自增id
- `keyProperty` 将自增的主键的值赋值给对应实体类的属性(赋值给id属性)

```xml
    <insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
        insert  into t_user values (null,#{username},'${password}',23,'男','111@qq.com')
    </insert>
```

### 配置字段对应属性 
默认不会自动将表字段映射为对应实体类的属性

- 通过配置 自动将下划线映射为 驼峰属性  


```xml
<!-- 通过配置 自动将下划线映射为 驼峰属性 -->
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
```


- 通过resultMap 自定义映射关系 

> `property` 是实体类属性名, `column` 是数据表字段名   
> `<id>` 设置主键 ,`<result>` 设置普通属性  


```xml
    <resultMap id="empResult" type="com.ric.entity.Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
    </resultMap>
```


### 多对一映射关系处理的三种方法
员工列表中的 部门属性

- 通过**级联属性赋值**解决多对一

```xml
    <resultMap id="empAndDept" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <!-- 注意这里 -->
        <result property="dept.did" column="did"/>
        <result property="dept.deptName" column="dept_name"/>
    </resultMap>

    <select id="getEmpAndDept" resultMap="empAndDept">
        select * from t_emp left join t_dept on t_emp.did = t_dept.did where t_emp.eid = #{eid};
    </select>
```

- 通过 `association`标签 解决多对一 

```xml
<resultMap id="empAndDept1" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <association property="dept" javaType="Dept">
            <id property="did" column="did"/>
            <result property="deptName" column="dept_name" />
        </association>
    </resultMap>
```

- 分步查询 解决多对一 

```xml
<resultMap id="empAndDeptStepOne" type="Emp">
        <id property="eid" column="eid"/>
        <result property="empName" column="emp_name"/>
        <result property="age" column="age"/>
        <result property="sex" column="sex"/>
        <result property="email" column="email"/>
        <!--
        select 设置分布查询的sql的唯一标识,(namespace.SqlId或者mapper接口的全类名.方法名)
        column 设置分步查询的条件(关联字段)
        -->
        <association property="dept" select="com.ric.mapper.DeptMapper.getDeptStepTwo" column="did">
        </association>
    </resultMap>
    <select id="getEmpAndDeptStepOne" resultMap="empAndDeptStepOne">
        select * from t_emp where eid = #{eid};
    </select>
```

```xml
<mapper namespace="com.ric.mapper.DeptMapper">
    <select id="getDeptStepTwo" resultType="com.ric.entity.Dept">
        select * from t_dept where did=#{did}
    </select>
</mapper>
```

> 分步查询的优点:可以实现延迟加载,但是必须再核心配置文件中设置全局配置信息(settings标签)  
> `lazyLoadingEnabled` :延迟加载的全局开关,当开启时,所有关联对象(分布查询的第二步,第三步...)都会延迟加载  
> `aggressiveLazyLoading` :当开启时,任何方法的调用都会加载该对象的所有属性,否则每个属性会按需加载  
> 此时就可以实现按需加载,获取的数据是什么,就只会执行响应的sql,此时可以通过 `association` 和 `collection`   
> 中的 `fetchType` 属性设置当前的分布查询是否使用延迟加载,`fetchType="lazy" (延迟加载)|eager(立即加载)`  
> **注: `fetchType` 属性 必须在开启全局延迟加载后才能生效** 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mybatis3.png) 

```xml
  <settings>
        <!-- 开启全局延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
```

或者

```properties
mybatis.configuration.lazy-loading-enabled=true
#false 为按需加载
mybatis.configuration.aggressive-lazy-loading=false

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mybatis2.png)



###  解决一对多映射
部门下的所有员工   
- `collection` 标签解决  


```xml
<resultMap id="deptAndEmp" type="Dept">
        <id property="did" column="did" />
        <result property="deptName" column="dept_name"/>
        <!-- ofType 集合中的类型 -->
        <collection property="emps" ofType="Emp">
            <id property="eid" column="eid"/>
            <result property="empName" column="emp_name"/>
            <result property="age" column="age"/>
            <result property="sex" column="sex"/>
            <result property="email" column="email"/>
        </collection>
    </resultMap>

    <select id="getDeptAndEmps" resultMap="deptAndEmp">
        select * from t_dept left join t_emp on t_dept.did = t_emp.did where t_dept.did = #{did}
    </select>
```

- 分步查询 

> 可以使用 `collection` 标签,也可以使用 `association` 标签

```xml
    <resultMap id="deptOne" type="Dept">
        <id property="did" column="did" />
        <result property="deptName" column="dept_name"/>
        <collection property="emps" select="com.ric.mapper.EmpMapper.getDeptStepTwo" column="did" fetchType="lazy"/>
    </resultMap>

    <select id="getDeptAndEmpOne" resultMap="deptOne">
        select * from t_dept where did= #{did}
    </select>

    <select id="getDeptStepTwo" resultType="com.ric.entity.Emp">
        select * from t_emp where did = #{did};
    </select>
```


## Mybatis 动态SQL  

### if
> 可以根据标签中 `test` 属性所对应的表达式决定标签中的内容是否需要拼接到SQL中 

```xml
    <select id="getEmpByCondition" resultType="com.ric.entity.Emp">
        select * from t_emp where 1=1
        <if test="empName!=null and empName!=''">
           and emp_name = #{empName}
        </if>
        <if test="age!=null and age!=''">
            and age = #{age}
        </if>
        <if test="sex!=null and sex!=''">
           and sex = #{sex}
        </if>
        <if test="email!=null and email!=''">
           and email = #{email}
        </if>
    </select>
```
> **注意:** 上面的 `where 1=1` 会导致如果没有条件 会返回所有信息 
> 如果没有 `1=1` 那么第一个empName会报错,没有empName条件时会报错


### where
> 当 `where` 标签中有内容时,会自动生成 `where` 关键字,并且将内容前的多余的 `and` 或 `or` 去掉
> 当 `where` 标签中没有内容时,此时 `where` 标签不会自动生成 `where` 关键字
> **注意:**  `where` 标签中不能再条件后添加 `and` 或 `or` 如` age = #{age} and `

```xml
        <where>
            <if test="empName!=null and empName!=''">
             emp_name = #{empName}
            </if>
            <if test="age!=null and age!=''">
                and age = #{age}
            </if>
            <if test="sex!=null and sex!=''">
                or sex = #{sex}
            </if>
            <if test="email!=null and email!=''">
                and email = #{email}
            </if>
        </where>
```

### trim 标签  
> `prefix`|`suffix` :将trim 标签中内容 前面或后面添加指定内容
> `prefixOverrides`|`suffixOverrides` 将trim 标签中 内容前面或后面去掉指定内容
> 标签中没有内容 trim 也不会有任何效果  

```xml
    <select id="getEmpByCondition" resultType="com.ric.entity.Emp">
        select * from t_emp
        <trim prefix="where" suffixOverrides="and|or">
            <if test="empName!=null and empName!=''">
             emp_name = #{empName} and
            </if>
            <if test="age!=null and age!=''">
                 age = #{age} or
            </if>
            <if test="sex!=null and sex!=''">
                 sex = #{sex} and
            </if>
            <if test="email!=null and email!=''">
                 email = #{email} and
            </if>
        </trim>

    </select>
```
### choose when otherwise 
> `choose when otherwise` 相当于java中的 `if..else if..else..`
> when 的条件 只会成立一个,第一个成立后续就不会再进行判断 
> when 最少有一个, otherwise 最多有一个 

```xml
    <select id="getEmpByChoose" resultType="com.ric.entity.Emp">
        select * from t_emp
        <where>
            <choose>
                <when test="empName !=null and empName!=''">
                    emp_name = #{empName}
                </when>
                <when test="age !=null and age!=''">
                    age = #{age}
                </when>
                <when test="sex !=null and sex!=''">
                    sex = #{sex}
                </when>

                <otherwise> did = 1</otherwise>
            </choose>
        </where>

    </select>
```

### foreach 

```java
    // 没有使用 @param 默认键值为 array 或者 arg0
    int deleteMoreByArray(int[] eids);
    int deleteMoreByArray(@Param("eids") int[] eids);
```
> `collection` 数组变量名,默认或者通过@Param设置  
> `item` 循环元素  
> `separator` 分隔符(分隔符前后会默认添加空格)  
> `open`  循环内容 起始符号  
> `close` 循环内容 结束符号  

- 批量删除 
```xml
    <delete id="deleteMoreByArray">
        delete from t_emp where eid in
        (
        <foreach collection="array" item="eid" separator=",">
            #{eid}
        </foreach>
        )

<!-- 去除了上面的小括号 -->
    </delete>
        <delete id="deleteMoreByArray">
        delete from t_emp where eid in
        <foreach collection="eids" item="eid" separator="," open="(" close=")">
            #{eid}
        </foreach>

    </delete>

    <!-- 使用 or 作为分隔符 -->
    </delete>
        <delete id="deleteMoreByArray">
        delete from t_emp where eid in
        <foreach collection="eids" item="eid" separator="or">
            #{eid}
        </foreach>

    </delete>
```

- 批量添加  

```xml
<!--     int insertMoreByList(List<Emp> empList); -->
    <insert id="insertMoreByList">
        insert into t_emp values
        <foreach collection="list" item="emp"  separator=",">
            (null,#{emp.empName},#{emp.age},#{emp.sex},#{emp.email},null)
        </foreach>
    </insert>

<!--     int insertMoreByList(@Param("emps") List<Emp> empList); -->
    <insert id="insertMoreByList">
        insert into t_emp values
        <foreach collection="emps" item="emp"  separator=",">
            (null,#{emp.empName},#{emp.age},#{emp.sex},#{emp.email},null)
        </foreach>
    </insert>
```


### sql 标签(片段)
> 设置sql片段和引用sql片段
```xml
  <sql id="empCloumns">
        eid,emp_name,age,sex,email
    </sql>

    <select id="getEmpByCondition" resultType="com.ric.entity.Emp">
        select <include refid="empCloumns"/> from t_emp </select>
```


## 一级缓存
> 一级缓存是 `SqlSession` 级别的,默认开启,通过同一个 `SqlSession` 查询的数据会被缓存,下次查询相同的数据,就会从缓存中直接获取,不会从数据库重新访问


- 使一级缓存失效的四种情况:
  - 不同的`SqlSession` 对应不同的一级缓存
  - 同一个`SqlSession` 但是查询条件不同
  - 同一个`SqlSession` 两次查询期间执行了任何一次增删改操作
  - 同一个`SqlSession` 两次查询间手动清空了缓存 `sqlSession.clearCache();`

## 二级缓存 
> 二级缓存是 `SqlSessionFactory` 级别, 通过同一个`SqlSessionFactory` 创建的 `SqlSession`查询的结果会被缓存,此后若再次执行相同的查询语句,结果就会从缓存中获取

二级缓存开启的条件:
- 在核心配置文件中,设置全局配置属性 `cacheEnable='true'`,默认为 true 不需要设置
- 在映射文件中设置标签 `<cache />`
- 二级缓存必须存在`SqlSession`关闭或提交之后有效,提交之前在一级缓存中
- 查询的数据所转换的**实体类型必须实现序列化接口** 

二级缓存失效的情况: 两次查询之间执行了任意的增删改操作,会使一级和二级缓存同时失效 


![缓存命中率](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mybatis5.png)


### 二级缓存的属性配置
官网说明: https://mybatis.org/mybatis-3/sqlmap-xml.html#cache  

`<cache eviction="" flushInterval="" size="" readOnly="" blocking="" type=""/>`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mybatis6.png) 

type 用来指定二级缓存类型(使用第三方缓存工具)

## 缓存查询顺序
- 先查询二级缓存,
- 二级没有命中,查一级
- 一级没有命中,查数据库
- `SqlSession`关闭后,一级缓存中的数据会写入二级缓存 

## 整合第三方缓存EHCache 

## MyBatis的逆向工程
- 正向工程：先创建Java实体类，由框架负责根据实体类生成数据库表。Hibernate是支持正向工程的
- 逆向工程：先创建数据库表，由框架负责根据数据库表，反向生成如下资源：  
	- Java实体类  
	- Mapper接口  
	- Mapper映射文件
### 添加依赖和插件
```xml
<dependencies>
	<!-- MyBatis核心依赖包 -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.9</version>
	</dependency>
	<!-- junit测试 -->
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.13.2</version>
		<scope>test</scope>
	</dependency>
	<!-- MySQL驱动 -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.27</version>
	</dependency>
	<!-- log4j日志 -->
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>1.2.17</version>
	</dependency>
</dependencies>
<!-- 控制Maven在构建过程中相关配置 -->
<build>
	<!-- 构建过程中用到的插件 -->
	<plugins>
		<!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
		<plugin>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.0</version>
			<!-- 插件的依赖 -->
			<dependencies>
				<!-- 逆向工程的核心依赖 -->
				<dependency>
					<groupId>org.mybatis.generator</groupId>
					<artifactId>mybatis-generator-core</artifactId>
					<version>1.3.2</version>
				</dependency>
				<!-- 数据库连接池 -->
				<dependency>
					<groupId>com.mchange</groupId>
					<artifactId>c3p0</artifactId>
					<version>0.9.2</version>
				</dependency>
				<!-- MySQL驱动 -->
				<dependency>
					<groupId>mysql</groupId>
					<artifactId>mysql-connector-java</artifactId>
					<version>8.0.27</version>
				</dependency>
			</dependencies>
		</plugin>
	</plugins>
</build>
```
### 创建MyBatis的核心配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"/>
    <typeAliases>
        <package name=""/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name=""/>
    </mappers>
</configuration>
```
### 创建逆向工程的配置文件
- 文件名必须是：`generatorConfig.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--
    targetRuntime: 执行生成的逆向工程的版本
    MyBatis3Simple: 生成基本的CRUD（清新简洁版）
    MyBatis3: 生成带条件的CRUD（奢华尊享版）
    -->
    <context id="DB2Tables" targetRuntime="MyBatis3Simple">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.atguigu.mybatis.pojo" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- SQL映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="com.atguigu.mybatis.mapper"
                         targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.atguigu.mybatis.mapper" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 逆向分析的表 -->
        <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
        <!-- domainObjectName属性指定生成出来的实体类的类名 -->
        <table tableName="t_emp" domainObjectName="Emp"/>
        <table tableName="t_dept" domainObjectName="Dept"/>
    </context>
</generatorConfiguration>
```
### 配置分页插件
- 在MyBatis的核心配置文件（mybatis-config.xml）中配置插件
```xml
<plugins>
	<!--设置分页插件-->
	<plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>
```
## 分页插件的使用
### 开启分页功能
- 在查询功能之前使用`PageHelper.startPage(int pageNum, int pageSize)`开启分页功能
	- pageNum：当前页的页码  
	- pageSize：每页显示的条数
```java
@Test
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	//访问第一页，每页四条数据
	PageHelper.startPage(1,4);
	List<Emp> emps = mapper.selectByExample(null);
	emps.forEach(System.out::println);
}
```

### 分页相关数据
#### 方法一：直接输出
```java
@Test
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	//访问第一页，每页四条数据
	Page<Object> page = PageHelper.startPage(1, 4);
	List<Emp> emps = mapper.selectByExample(null);
	//在查询到List集合后，打印分页数据
	System.out.println(page);
}
```
- 分页相关数据：
```json
	Page{count=true, pageNum=1, pageSize=4, startRow=0, endRow=4, total=8, pages=2, reasonable=false, pageSizeZero=false}[Emp{eid=1, empName='admin', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=2, empName='admin2', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=3, empName='王五', age=12, sex='女', email='123@qq.com', did=3}, Emp{eid=4, empName='赵六', age=32, sex='男', email='123@qq.com', did=1}]
```
#### 方法二使用PageInfo
- 在查询获取list集合之后，使用`PageInfo<T> pageInfo = new PageInfo<>(List<T> list, intnavigatePages)`获取分页相关数据
	- list：分页之后的数据  
	- navigatePages：导航分页的页码数
```java
@Test
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	PageHelper.startPage(1, 4);
	List<Emp> emps = mapper.selectByExample(null);
	PageInfo<Emp> page = new PageInfo<>(emps,5);
	System.out.println(page);
}
```
- 分页相关数据：
	```
	PageInfo{
	pageNum=1, pageSize=4, size=4, startRow=1, endRow=4, total=8, pages=2, 
	list=Page{count=true, pageNum=1, pageSize=4, startRow=0, endRow=4, total=8, pages=2, reasonable=false, pageSizeZero=false}[Emp{eid=1, empName='admin', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=2, empName='admin2', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=3, empName='王五', age=12, sex='女', email='123@qq.com', did=3}, Emp{eid=4, empName='赵六', age=32, sex='男', email='123@qq.com', did=1}], 
	prePage=0, nextPage=2, isFirstPage=true, isLastPage=false, hasPreviousPage=false, hasNextPage=true, navigatePages=5, navigateFirstPage=1, navigateLastPage=2, navigatepageNums=[1, 2]}
	```
- 其中list中的数据等同于方法一中直接输出的page数据
#### 常用数据：
- pageNum：当前页的页码  
- pageSize：每页显示的条数  
- size：当前页显示的真实条数  
- total：总记录数  
- pages：总页数  
- prePage：上一页的页码  
- nextPage：下一页的页码
- isFirstPage/isLastPage：是否为第一页/最后一页  
- hasPreviousPage/hasNextPage：是否存在上一页/下一页  
- navigatePages：导航分页的页码数  
- navigatepageNums：导航分页的页码，\[1,2,3,4,5]



























## 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV1VP4y1c7j7)
> - []()
