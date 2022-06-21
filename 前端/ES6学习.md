笔记根据尚硅谷es6教程记录， 结合阮一峰大佬教程，目录以视频为准，缺失内容后续补充

## let 和 const  

### let
- let变量不能重复声明
- let变量只在其所在代码块有效（块级作用域）  
- 不存在变量提升  
- 不存在作用域链  


```javascript
{
    let  x = 3;
}
console.log(x) // not defiend

{
    let x = 3;
    function fn (){
        console.log(x)  // 3
    }
}
//不影响作用域链 ，上面的方法 内部可以访问父级let变量
```
---

### const
- const 必须赋初始值
- 一般常量使用大写
- 常量值不能修改
- 同样是块级作用域
- 对其内部的对象修改不算做对常量的修改（常量地址指向没有发生变化）
```javascript
const foo = {};
//正常操作
foo.name ='132';
foo.id = '3';

foo = {};//报错 TypeError ：foo is read-only

```

## 变量的解构赋值
> `ES6` 允许按照一定模式从数组和对象中提取值，对变量进行赋值。  
### 数组解构

```javascript
const X = ['111','222','333'];
let [one,two,three] = X;
console.log(one)//111
console.log(two)//222
console.log(three)//333   

//指定默认值
let [foo = true] = [];
foo //true  

let [x = 1 ] = [undefined] 
x //1
let [x = 1 ] = [null];
x //null


//解构失败
let [foo] = [];
let [bar,foo] = [1];
let [foo] = 1;
let [foo] = {};
// 上述foo解构失败，值为undefined

```
> 如果等号右边不是数组（或者说可遍历结构），那么就会报错  

> ES6 内部使用严格相等运算符（===）， 判断一个位置是否有值。所以只有当一个数组成员严格等于`undefined` ，默认值才会生效 。

> 如果 默认值是一个表达式，那么这个表达式是惰性求值的，只有再用到的时候才会求值。

```javascript
function f(){
    console.log('1111');
}

let [x = f()] = [1];
//上面的函数f()不会执行

```
### 对象解构
```javascript

//对象解构
let {foo,bar} = {foo:'1111',bar:'bbb'}
foo//1111
bar//bbb 
```
### 字符串解构
```javascript

//字符串解构
const [a,b,c,d] = 'helo';
a//h
b//e
```

### 函数参数解构
```javascript
function add([x,y]){
    return x+y;
}

add([1,2]);//3 

```


## 字符串的扩展

### 模板字符串  
```javascript
//1 声明
let str =  `字符串`;
str // 字符串

// 2 可以直接换行
let str = `字符1
djfai
adasf`

//3 变量拼接
let x = '字符串';
let out = `${x}111111`
out// 字符串111111

```
## 箭头函数
ES6 允许使用箭头`=>` 定义函数

```javascript
let fn = (a,b)=>{
    return a+b;
}
var result  = fn(1,2)

console.log(result)//3
```




## 参考资料
> - [ES6教程](https://es6.ruanyifeng.com/#docs/let)
> - [尚硅谷ES6](https://www.bilibili.com/video/BV1uK411H7on)
