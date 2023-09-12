# 配置  

## nodejs
[官网](https://nodejs.cn/download/)

[安装详见](https://blog.csdn.net/qq_42006801/article/details/124830995)

## 包管理工具

### npm
Node.js 自带的包管理器,通过`npm install`命令来安装

npm 的包的版本锁定文件是`package-lock.json`

```shell
# 获取镜像地址
npm config get registry

#修改镜像地址，二选一
npm config set registry https://registry.npm.taobao.org
npm config set registry https://registry.npmmirror.com

```


### cnpm 
cnpm 是阿里巴巴推出的包管理工具，安装之后默认会使用 https://registry.npmmirror.com 这个镜像源。

通过 `cnpm install`命令来安装

```shell
# 通过npm安装cnpm
npm install -g cnpm
# 或者
npm install -g cnpm --registry=https://registry.npmmirror.com

```

### yarn
yarn 也是一个常用的包管理工具，和 npm 十分相似， npmjs 上的包，也会同步到 yarnpkg   
但是安装命令上会有点不同， `yarn` 是用 `yarn add` 代替 `npm install` ，用 `yarn remove` 代替`npm uninstall`

```shell
#安装yarn
npm install -g yarn  

# 安装单个包
yarn add vue-router
 
# 安装全局包
yarn global add typescript
 
# 卸载包
yarn remove vue-router
```

运行脚本的时候，可以直接用 `yarn` 来代替` npm run` ，例如 `yarn dev` 相当于 `npm run dev` 。 升级的时候用 `yarn upgrade` 代替 `npm update`命令

```shell
#yarn 默认绑定的是 https://registry.yarnpkg.com 的下载源，如果包的下载速度太慢，也可以配置镜像源，但是命令有所差异
# 查看镜像源
yarn config get registry
 
# 绑定镜像源
yarn config set registry https://registry.npmmirror.com
 
# 删除镜像源（注意这里是 delete ）
yarn config delete registry
```

yarn 的 版本锁定文件是`yarn.lock`

### pnpm
pnpm 是包管理工具的一个后起之秀，主要优点在于快速的、节省磁盘空间，如果你的包在一个项目中已经下载了，其它项目再用到这个包就不需要再次下载，而是通过软链接的方式关联

```shell
#全局安装pnpm
npm install -g pnpm
```

!>  npm 和 yarn 的命令都支持,所以推荐使用pnpm


## 打包工具

### webpack


### vite


# 基础  

## 模板语法

- 插值语法
  - `{{xxx}}`，xxx是js表达式，且可以直接读取到data中的所有属性
  - 一般用于标签体内
- 指令语法
  - `v-bind:href="xxx"` 或简写为 `:href="xxx"`，xxx同样要写js表达式,且可以直接读取到data中的所有属性
  - 一般用于标签属性(推荐)


## 数据绑定 
- 单项绑定`v-bind`数据只能从data流向页面，页面的改动不会影响data内容
- 双向绑定`v-model:value`可以简写为`v-model`，`v-model`默认收集的就是value值

!> `v-model`只能应用在表单类元素（输入类元素）上

## el 和 data的两种写法
el 

```js
//第一种
new Vue({
	el:'#root',
	data:{
		name:'尚硅谷'
	}
})

// 第二种 vm.$mount('#root')指定el的值
// v即 vm对象
const v = new Vue({
	data:{
		name:'尚硅谷'
	}
})
console.log(v)
v.$mount('#root')

```

data

```js
//data的两种写法
new Vue({
	el:'#root',
	//data的第一种写法：对象式
	/* data:{
		name:'尚硅谷'
	} */
	//data的第二种写法：函数式
	data(){
		console.log('@@@',this) //此处的this是Vue实例对象
		return{
			name:'尚硅谷'
		}
	}
})
```

## vue和MVVM模型
MVVM模型
1. M：模型(Model) ：data中的数据
2. V：视图(View) ：模板代码
3. VM：视图模型(ViewModel)：Vue实例

观察发现：
1. data中所有的属性，最后都出现在了vm身上。
2. vm身上所有的属性，及Vue原型上所有属性，在Vue模板中都可以直接使用。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/vue-mvvm模型示意图.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/vue-mvvm模型示意图代码.png)


## 数据代理 defineProperty
Object.defineproperty 的作用就是直接在一个对象上定义一个新属性，或者修改一个已经存在的属性


```js

let number = 18
let person = {
	name:'张三',
	sex:'男',
}
Object.defineProperty(person,'age',{
	// value:18,
	// enumerable:true, //控制属性是否可以枚举，默认值是false
	// writable:true, //控制属性是否可以被修改，默认值是false
	// configurable:true //控制属性是否可以被删除，默认值是false
	//当有人读取person的age属性时，get函数(getter)就会被调用，且返回值就是age的值
	get(){
		console.log('有人读取age属性了')
		return number
	},
	//当有人修改person的age属性时，set函数(setter)就会被调用，且会收到修改的具体值
	set(value){
		console.log('有人修改了age属性，且值是',value)
		number = value
	}
})

// console.log(Object.keys(person))

console.log(person)
```

数据代理，通过一个对象代理对另一个对象中属性的操作（读/写）  
访问 obj2的x属性  

```js
let obj = {x:100}
let obj2 = {y:200}

Object.defineProperty(obj2,'x',{
	get(){
		return obj.x
	},
	set(value){
		obj.x = value
	}
})
```

Vue中的数据代理  

1. Vue中的数据代理：通过vm对象来代理data对象中属性的操作（读/写）
2. Vue中数据代理的好处：更加方便的操作data中的数据
3. 基本原理：通过`Object.defineProperty()`把data对象中所有属性添加到vm上。为每一个添加到vm上的属性，都指定一个getter/setter。在getter/setter内部去操作（读/写）data中对应的属性。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/vue中的数据代理.png)

# 事件处理  

## 事件绑定

1. 使用`v-on:xxx` 或 `@xxx` 绑定事件，其中xxx是事件名；
2. 事件的回调需要配置在methods对象中，最终会在vm上；
3. methods中配置的函数，不要用箭头函数！否则this就不是vm了；
4. methods中配置的函数，都是被Vue所管理的函数，this的指向是vm 或 组件实例对象；
5. `@click="demo"` 和 `@click="demo($event)"` 效果一致，但后者可以传参；

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>事件的基本使用</title>
		<!-- 引入Vue -->
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<!-- 准备好一个容器-->
		<div id="root">
			<h2>欢迎来到{{name}}学习</h2>
			<!-- <button v-on:click="showInfo">点我提示信息</button> -->
			<button @click="showInfo1">点我提示信息1（不传参）</button>
			<button @click="showInfo2($event,66)">点我提示信息2（传参）</button>
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

		const vm = new Vue({
			el:'#root',
			data:{
				name:'尚硅谷',
			},
			methods:{
				showInfo1(event){
					// console.log(event.target.innerText)
					// console.log(this) //此处的this是vm
					alert('同学你好！')
				},
				showInfo2(event,number){
					console.log(event,number)
					// console.log(event.target.innerText)
					// console.log(this) //此处的this是vm
					alert('同学你好！！')
				}
			}
		})
	</script>
</html>
```

## 事件修饰  
1. prevent：阻止默认事件,常用`@click.prevent="showInfo"`
2. stop：阻止事件冒泡（常用）`@click.stop="showInfo"`
3. once：事件只触发一次（常用）`@click.once="showInfo"`
4. capture：使用事件的捕获模式；
5. self：只有event.target是当前操作的元素时才触发事件；
6. passive：事件的默认行为立即执行，无需等待事件回调执行完毕；


## 键盘事件 
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>键盘事件</title>
		<!-- 引入Vue -->
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<!-- 
			1.Vue中常用的按键别名：
				回车 => enter
				删除 => delete (捕获“删除”和“退格”键)
				退出 => esc
				空格 => space
				换行 => tab (特殊，必须配合keydown去使用)
				上 => up
				下 => down
				左 => left
				右 => right

			2.Vue未提供别名的按键，可以使用按键原始的key值去绑定，但注意要转为kebab-case（短横线命名） CapsLock => caps-lock

			3.系统修饰键（用法特殊）：ctrl、alt、shift、meta
				(1).配合keyup使用：按下修饰键的同时，再按下其他键，随后释放其他键，事件才被触发。
				(2).配合keydown使用：正常触发事件。

			4.也可以使用keyCode去指定具体的按键（不推荐）

			5.Vue.config.keyCodes.自定义键名 = 键码，可以去定制按键别名
		-->
		<!-- 准备好一个容器-->
		<div id="root">
			<h2>欢迎来到{{name}}学习</h2>
			<input type="text" placeholder="按下回车提示输入" @keydown.huiche="showInfo">
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。
		Vue.config.keyCodes.huiche = 13 //定义了一个别名按键

		new Vue({
			el:'#root',
			data:{
				name:'尚硅谷'
			},
			methods: {
				showInfo(e){
					// console.log(e.key,e.keyCode)
					console.log(e.target.value)
				}
			},
		})
	</script>
</html>

```

# 计算属性

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>姓名案例_计算属性实现</title>
		<!-- 引入Vue -->
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<!-- 
        计算属性：
            1.定义：要用的属性不存在，要通过已有属性计算得来。
            2.原理：底层借助了Objcet.defineproperty方法提供的getter和setter。
            3.get函数什么时候执行？
                (1).初次读取时会执行一次。
                (2).当依赖的数据发生改变时会被再次调用。
            4.优势：与methods实现相比，内部有缓存机制（复用），效率更高，调试方便。
            5.备注：
                1.计算属性最终会出现在vm上，直接读取使用即可。
                2.如果计算属性要被修改，那必须写set函数去响应修改，且set中要引起计算时依赖的数据发生改变。
		 -->
		<!-- 准备好一个容器-->
		<div id="root">
			姓：<input type="text" v-model="firstName"> <br/><br/>
			名：<input type="text" v-model="lastName"> <br/><br/>
			测试：<input type="text" v-model="x"> <br/><br/>
			全名：<span>{{fullName}}</span> <br/><br/>
			<!-- 全名：<span>{{fullName}}</span> <br/><br/>
			全名：<span>{{fullName}}</span> <br/><br/>
			全名：<span>{{fullName}}</span> -->
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

		const vm = new Vue({
			el:'#root',
			data:{
				firstName:'张',
				lastName:'三',
				x:'你好'
			},
			methods: {
				demo(){
					
				}
			},
			computed:{
				fullName:{
					//get有什么作用？当有人读取fullName时，get就会被调用，且返回值就作为fullName的值
					//get什么时候调用？1.初次读取fullName时。2.所依赖的数据发生变化时。
					get(){
						console.log('get被调用了')
						// console.log(this) //此处的this是vm
						return this.firstName + '-' + this.lastName
					},
					//set什么时候调用? 当fullName被修改时。
					set(value){
						console.log('set',value)
						const arr = value.split('-')
						this.firstName = arr[0]
						this.lastName = arr[1]
					}
				},
                //简写 这里就是默认getter
				fullName1(){
					console.log('get被调用了')
					return this.firstName + '-' + this.lastName
				}
			}
		})
	</script>
</html>
```


# 监视属性
监视属性watch：
- 当被监视的属性变化时, 回调函数自动调用`handler`, 进行相关操作
- 监视的属性必须存在，才能进行监视！！
- 监视的两种写法：
  - new Vue时传入`watch`配置
  - 通过`vm.$watch`监视
- 在不需要`immediate`属性和`deep`属性时，可以采用简写形式

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>天气案例_监视属性</title>
		<!-- 引入Vue -->
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<!-- 
        监视属性watch：
            1.当被监视的属性变化时, 回调函数自动调用, 进行相关操作
            2.监视的属性必须存在，才能进行监视！！
            3.监视的两种写法：
                (1).new Vue时传入watch配置
                (2).通过vm.$watch监视
		 -->
		<!-- 准备好一个容器-->
		<div id="root">
			<h2>今天天气很{{info}}</h2>
			<button @click="changeWeather">切换天气</button>
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。
		
		const vm = new Vue({
			el:'#root',
			data:{
				isHot:true,
			},
			computed:{
				info(){
					return this.isHot ? '炎热' : '凉爽'
				}
			},
			methods: {
				changeWeather(){
					this.isHot = !this.isHot
				}
			},
			watch:{
				isHot:{
					immediate:true, //初始化时让handler调用一下
					//handler什么时候调用？当isHot发生改变时。
					handler(newValue,oldValue){
						console.log('isHot被修改了',newValue,oldValue)
					}
				},
                //简写
				isHot(newValue,oldValue){
					console.log('isHot被修改了',newValue,oldValue,this)
				}
			}
		})

		vm.$watch('isHot',{
			immediate:true, //初始化时让handler调用一下
			//handler什么时候调用？当isHot发生改变时。
			handler(newValue,oldValue){
				console.log('isHot被修改了',newValue,oldValue)
			}
		})
        //简写
		vm.$watch('isHot',(newValue,oldValue)=>{
			console.log('isHot被修改了',newValue,oldValue,this)
		})
	</script>
</html>
```

## 深度监视
- Vue中的watch默认不监测对象内部值的改变（一层）。
- 配置deep:true可以监测对象内部值改变（多层）。

> Vue自身可以监测对象内部值的改变，但Vue提供的watch默认不可以    
> 使用watch时根据数据的具体结构，决定是否采用深度监视。

```js
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。
		
		const vm = new Vue({
			el:'#root',
			data:{
				isHot:true,
				numbers:{
					a:1,
					b:1,
					c:{
						d:{
							e:100
						}
					}
				}
			},
			computed:{
				info(){
					return this.isHot ? '炎热' : '凉爽'
				}
			},
			methods: {
				changeWeather(){
					this.isHot = !this.isHot
				}
			},
			watch:{
				isHot:{
					// immediate:true, //初始化时让handler调用一下
					//handler什么时候调用？当isHot发生改变时。
					handler(newValue,oldValue){
						console.log('isHot被修改了',newValue,oldValue)
					}
				},
				//监视多级结构中某个属性的变化
				'numbers.a':{
					handler(){
						console.log('a被改变了')
					}
				}
				//监视多级结构中所有属性的变化
				numbers:{
					deep:true,
					handler(){
						console.log('numbers改变了')
					}
				}
			}
		})
```

## watch的异步特性
computed和watch之间的区别：
- computed能完成的功能，watch都可以完成。  
- watch能完成的功能，computed不一定能完成，例如：watch可以进行异步操作。

两个重要的小原则
- 所被Vue管理的函数，最好写成普通函数，这样this的指向才是vm 或 组件实例对象。
- **所有不被Vue所管理的函数（定时器的回调函数、ajax的回调函数等、Promise的回调函数），最好写成箭头函数，这样this的指向才是vm 或 组件实例对象**。

如果有一个延迟上面属性变化1s的需求，那么就只能使用watch进行实现 

```js
		const vm = new Vue({
			el:'#root',
			data:{
				firstName:'张',
				lastName:'三',
				fullName:'张-三'
			},
			watch:{
				firstName(val){
					setTimeout(()=>{
						console.log(this)
						this.fullName = val + '-' + this.lastName
					},1000);
				},
				lastName(val){
					this.fullName = this.firstName + '-' + val
				}
			}
		})
```

# 绑定样式 
- class绑定
  - `class="xxx"` xxx可以是字符串、对象、数组。
  - 字符串写法适用于：类名不确定，要动态获取。
  - 对象写法适用于：要绑定多个样式，个数不确定，名字也不确定。
  - 数组写法适用于：要绑定多个样式，个数确定，名字也确定，但不确定用不用。
- style绑定
  - `:style="{fontSize: xxx}"`其中xxx是动态值。
  - `:style="[a,b]"`其中a、b是样式对象。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>绑定样式</title>
		<style>
			.basic{
				width: 400px;
				height: 100px;
				border: 1px solid black;
			}
			
			.happy{
				border: 4px solid red;;
				background-color: rgba(255, 255, 0, 0.644);
				background: linear-gradient(30deg,yellow,pink,orange,yellow);
			}
			.sad{
				border: 4px dashed rgb(2, 197, 2);
				background-color: gray;
			}
			.normal{
				background-color: skyblue;
			}

			.atguigu1{
				background-color: yellowgreen;
			}
			.atguigu2{
				font-size: 30px;
				text-shadow:2px 2px 10px red;
			}
			.atguigu3{
				border-radius: 20px;
			}
		</style>
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<!-- 准备好一个容器-->
		<div id="root">
			<!-- 绑定class样式--字符串写法，适用于：样式的类名不确定，需要动态指定 -->
			<div class="basic" :class="mood" @click="changeMood">{{name}}</div> <br/><br/>

			<!-- 绑定class样式--数组写法，适用于：要绑定的样式个数不确定、名字也不确定 -->
			<div class="basic" :class="classArr">{{name}}</div> <br/><br/>

			<!-- 绑定class样式--对象写法，适用于：要绑定的样式个数确定、名字也确定，但要动态决定用不用 -->
			<div class="basic" :class="classObj">{{name}}</div> <br/><br/>

			<!-- 绑定style样式--对象写法 -->
			<div class="basic" :style="styleObj">{{name}}</div> <br/><br/>
			<!-- 绑定style样式--数组写法 -->
			<div class="basic" :style="styleArr">{{name}}</div>
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false
		
		const vm = new Vue({
			el:'#root',
			data:{
				name:'尚硅谷',
				mood:'normal',
				classArr:['atguigu1','atguigu2','atguigu3'],
				classObj:{
					atguigu1:false,
					atguigu2:false,
				},
				styleObj:{
					fontSize: '40px',
					color:'red',
				},
				styleObj2:{
					backgroundColor:'orange'
				},
				styleArr:[
					{
						fontSize: '40px',
						color:'blue',
					},
					{
						backgroundColor:'gray'
					}
				]
			},
			methods: {
				changeMood(){
					const arr = ['happy','sad','normal']
					const index = Math.floor(Math.random()*3)
					this.mood = arr[index]
				}
			},
		})
	</script>
```








