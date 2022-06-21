# Java8 实际案例

# 快速搜索方法

---
## Lambda 和方法引用 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战2.png) 

## 流
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战8.png) 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战10.png) 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战11.png) 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战12.png) 

## Optional
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战7.png) 

## 时间 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战18.png) 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/实战19.png) 


## 函数接口

接口 | 参数 | 返回值| 示例 
----|------|-------|--
 `Predicate<T>` | `T` | `void`|`s->System.out.print(s)`
 `Consumer<T>` | `T` | `boolean`|`s->s.isEmpty()`
 `Function<T,U>` | `T` | `U`|`(String s) -> s.length()`
 `Supplier<T>` | 无 | `T` |`s->new String()`


# 案例


# 参考资料
> - 《Java 8 In Action》
