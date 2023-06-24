# Effective Java 笔记

# 创建和销毁对象 

## 1. 用静态工厂方法替代构造器 
获取一个类实例，一般有两种方法:  
- 公有构造器
- 公有静态工厂方法(并非工厂模式)

为什么要采用静态工厂方法？   
1. 静态工厂方法有名称，通过名称可以判定工厂方法的作用，提供的构造对象的内容等  
2. 静态工厂方法**不必在每次调用它们时都创建一个新对象**,下例  

```java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE;
}

```

3. 静态工厂方法可以返回**原类型的任何子类型对象**  
4. 静态工厂方法返回的对象类，可以随着每次调用参数不同而发生变化
5. 静态工厂方法返回的对象所属的类，在编写包含该静态工厂方法的类时可以不存在

第五点，以一个服务提供者框架来理解

```java
//四大组成之一：服务接口
public interface LoginService {//这是一个登录服务
    public void login();
}
 
//四大组成之二：服务提供者接口
public interface Provider {//登录服务的提供者。通俗点说就是：通过这个newLoginService()可以获得一个服务。
    public LoginService newLoginService();
}
 
/**
 * 这是一个服务管理器，里面包含了四大组成中的三和四
 * 解释：通过注册将 服务提供者 加入map，然后通过一个静态工厂方法 getService(String name) 返回不同的服务。
 */
public class ServiceManager {
    private static final Map<String, Provider> providers = new HashMap<String, Provider>();//map，保存了注册的服务
 
    private ServiceManager() {
    }
 
    //四大组成之三：提供者注册API  (其实很简单，就是注册一下服务提供者)
    public static void registerProvider(String name, Provider provider) {
        providers.put(name, provider);
    }
 
    //四大组成之四：服务访问API   (客户端只需要传递一个name参数，系统会去匹配服务提供者，然后提供服务)  (静态工厂方法)
    public static LoginService getService(String name) {
        Provider provider = providers.get(name);
        if (provider == null) {
            throw new IllegalArgumentException("No provider registered with name=" + name);
 
        }
        return provider.newLoginService();
    }
}
```
