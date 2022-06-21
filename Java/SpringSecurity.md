# Spring Security

> SpringSecurity 本质上是一个过滤器链，三个重点过滤器 `FilterSecurityInterceptor` `ExceptionTranslationFilter` `UsernamePasswordAuthenticationFilter`

- `FilterSecurityInterceptor` 是一个方法级别的过滤器,基本位于过滤链的最底部  
- `ExceptionTranslationFilter` 是一个异常过滤器,用来处理在认证授权过程中抛出的异常
- `UsernamePasswordAuthenticationFilter` 对`/login` 的POST请求做拦截,校验表单中 用户名,密码 

过滤器如何进行加载
- 使用Spring Security配置过滤器 `DelegatingFilterProxy` 

## WEB权限方案
- 1.认证 
- 2.授权

### 设置登录系统的账号密码
- 通过配置文件实现
```properties
spring.security.user.name=root
spring.security.user.password=root
```
- 编写接口实现类

```java
/* 
配置类
 */
@Configuration
public class MyConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        BCryptPasswordEncoder bt = new BCryptPasswordEncoder();
        String encode = bt.encode("123456");
        auth.inMemoryAuthentication().withUser("root").password(encode).roles("admin");

    }

    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}

```

- 查询数据库认证

启动类添加`@MapperScan("com.ric.mapper")`
```java
/**
 * 配置类
 */
@Configuration
public class MyConfigUser extends WebSecurityConfigurerAdapter {

    @Autowired
    private MyUserDetailsService myUserDetailsService;
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailsService);
    }
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

```java
/**
 * 查询数据库 校验信息
 */
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private TUserMapper userMapper;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //调用mapper查询数据库
        TUser tUser = userMapper.selectOne(new LambdaQueryWrapper<TUser>().eq(TUser::getUsername,username));
        if(null==tUser){
            throw new UsernameNotFoundException("用户名不存在");
        }
        List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
        String encode = bCryptPasswordEncoder.encode(tUser.getPassword());
        return new User(tUser.getUsername(),encode,role);

    }
}
```

### 自定义登录页面 
配置类,继承 `WebSecurityConfigurerAdapter` 重写方法 `protected void configure(HttpSecurity http)`

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        
        //权限不足 错误页面配置
        http.exceptionHandling().accessDeniedPage("/403.html");
        //登出配置
        http.logout().logoutUrl("/logout").logoutSuccessUrl("/user/hello");

        http.formLogin()
                .loginPage("/user/login")//登陆页面
                .loginProcessingUrl("/user/doLogin")//登录访问路径
                .defaultSuccessUrl("/user/index")//登录成功跳转路径
                .and()
                .authorizeHttpRequests()
                .antMatchers("/user/hello","/user/login","/user/doLogin")//不需要认证可以访问的路径
                .permitAll()
                //只有具有admin权限的才可以访问这个路径
//                .antMatchers("/test/index").hasAuthority("admins")
//                .antMatchers("/test/index").hasAnyAuthority("admins","user")

                .antMatchers("/user/index").hasRole("sale")
//                .antMatchers("/user/index").hasAnyRole("sale","boss")
                .anyRequest()
                .authenticated()
                //rememberMe 功能
                .and().rememberMe().tokenRepository(persistentTokenRepository())
                .tokenValiditySeconds(60)
                .userDetailsService(myUserDetailsService)

                .and()
                .csrf()
                .disable();//关闭csrf防护

    }
```
> 登录页面`/user/login` 需要加入免认证列表,否则无法访问,登录访问路径 `/user/doLogin` 只需要配置,不需要重写接口
> 登录成功路径 `/user/index` 需要有实际映射地址

### 基于角色或权限访问控制 

#### 基于权限
`hasAuthority` **仅支持单个权限校验**方法,如果当前主题具有指定权限,则返回true 否则返回 false

1. 配置类中设置当前访问地址有哪些权限
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec3.png)

2. 在 `MyUserDetailsService` 中设置,若没有访问权限就会返回`403` 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec2.png)

`hasAnyAuthority`  支持多个权限 
`.antMatchers("/test/index").hasAnyAuthority("admins","user")`



#### 基于角色
> 如果用户具备给定角色就允许访问，否则返回 `403`
> 如果当前主体具有指定角色，则返回 `true`

判断角色(两种) `hasRole` `hasAnyRole`
```
.antMatchers("/test/index").hasRole("sale")
.antMatchers("/test/index").hasAnyRole("sale","boss")
```
`hasRole` 源码会默认添加前缀 `ROLE_`
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec5.png)

设置角色(要带`ROLE_` 前缀)
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec4.png)


### 自定义403页面 
> 1. 配置类中configure 方法 `http.exceptionHandling().accessDeniedPage("/accessDenied.html")` 
> 2. 或者使用springboot 自定义的error文件夹下的对应页面`403.html`

### 注解使用

#### @Secured 
用户具有某个角色，可以访问方法
1. 启动类或配置类开启注解操作 `@EnableGlobalMethodSecurity(securedEnabled = true)` 
2. 在Contorller 方法上使用注解 `@Secured({"ROLE_sale","ROLE_boss"})` ,只允许sale 角色 和boss角色
3. 只能使用 `ROLE_**` 格式进行匹配,也只能匹配相同格式的**角色代码**(Spring Security中角色默认都以ROLE_开头)

#### @PreAuthorire
> 在方法之前进行校验 


1. `@EnableGlobalMethodSecurity(prePostEnabled = true)` 
2. 使用`hasAnyAuthority` 或者 `hasAnyRole` 方法进行校验,注意role相关的额前缀`ROLE_`

```java
    @GetMapping("update")
//    @PreAuthorize("hasAnyAuthority('admins')")
    @PreAuthorize("hasRole('ROLE_sale')")
    public String update(Model model){
        return "index";
    }
```

#### @PostAuthorire 
> 在方法之后进行权限验证,适合验证带有**返回值**的方法权限    


1. `@EnableGlobalMethodSecurity(prePostEnabled = true)`  使用方法同上

``` java

    @GetMapping("index")
    @PostAuthorize("hasRole('ROLE_sales')")
    public String index(Model model){
        System.out.println(1111);
        return "index";
    }
```
> 如果没有权限,仍然会输出1111 但是会返回403



#### @PostFilter
> 权限验证之后对数据进行过滤,留下用户名是admin1 的数据 
> 表达式 `filterObject`  引用的是方法返回值List中的某一个元素 

只会返回id为1的数据 list中只有一条id为1 的数据
```java
    @GetMapping("hello")
    @PostFilter(value = "filterObject.id ==1")
    @ResponseBody
    public List hello(Model model){
        ArrayList<TUser> tUsers = new ArrayList<>();
        TUser tUser = new TUser(1, "x1", "12", 1);
        TUser tUser1 = new TUser(2, "x1", "12", 1);
        tUsers.add(tUser);
        tUsers.add(tUser1);
        return tUsers;
    }
```

#### @PreFilter
> 对传入的参数进行过滤 

对参数进行过滤 ,age能被2整除才能传入
```java
    @GetMapping("hellox")
    @PreFilter(value = "filterObject.age%2 ==0")
    @ResponseBody
    public List hellox(@RequestBody List<TUser> tUsers){
        
        
        return tUsers;
    }
```


#### 权限表达式  

### 用户注销 
1. 在配置类中添加退出配置   
`http.logout().logoutUrl("/logout").logoutSuccessUrl("/user/hello");`


### 基于数据库实现 remember me功能  

1. 创建数据库表 `JdbcTokenRepositoryImpl`类中 `CREATE_TABLE_SQL` 内容创建数据库表
2. 配置类,注入数据源,配置操作数据库对象 
```java
    @Autowired
    private DataSource dataSource;

    @Bean
    public PersistentTokenRepository persistentTokenRepository(){
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        //设置是否自动创建表
//        jdbcTokenRepository.setCreateTableOnStartup(true);
        return jdbcTokenRepository;
    }
```
3. 配置类配置

```java
.and().rememberMe().tokenRepository(persistentTokenRepository())
                .tokenValiditySeconds(60)
                .userDetailsService(myUserDetailsService)
```
4. 前端配置 属性名称必须为`remember-me`
```html
    <p> 记住我:<input type="checkbox" name="remember-me"/></p>
```

### CSRF 
跨站请求伪造:其他网站可以访问已经登录的网站的登陆后信息

> 从Spring Security4 开始,默认情况下会启用csrf保护, 方式csrf攻击,针对 `PATCH` `POST`  `PUT` `DELETE` 方法进行防护.  

1. 开启csrf保护(默认开启,不主动关闭即可)
2. 登录页添加隐藏项

```html
<input type="hidden" th:name=${_csrf_parameterName} th:value="${_csrf.token}>
```
3. 需要引入thymeleaf 和Security 的整合包

## 微服务权限方案  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec8.png)
### 需求说明
1. 登录
2. 添加角色
3. 为角色分配菜单
4. 添加用户
5. 为用户分配角色
### 数据模型
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec9.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec7.png)
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec6.png)

### 相关插件
mybatisplus、spring cloud、gateway、nacos、redis、jwt、swagger
各模块作用 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec10.png)


### 核心代码
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec14.png)

1. 密码处理工具类
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec11.png)

2. Token 工具类（JWT）
> JWT的本质就是一个字符串，它是将用户信息保存到一个Json字符串中，然后进行编码后得到一个JWT token，并且这个JWT token带有签名信息，接收后可以校验是否被篡改，所以可以用于在各方之间安全地将信息作为Json对象传输,字符串通过 `.` 隔开为三个子串，每个子串表示一个功能块，总共分为一下三个部分，JWT头，有效载荷，和签名

 ![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec12.png)


3. 退出处理器

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec13.png)


4. 未授权统一处理类

```java
/**
 * 未授权统一处理类
 */
public class UnAuthEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        ResponseUtil.out(response, R.error());
    }
}
```

5. Security认证过滤器

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec15.png)


6. 授权过滤器
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec16.png)


## 源码分析
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec21.png)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec19.png)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec17.png)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec18.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sec20.png)




















## 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV15a411A7kP)
> - []()
