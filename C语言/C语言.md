# 基础知识  
C 语言是一种 编译型语言 ，源码都是 文本文件 ，本身无法执行。必须通过编译器，生成二进制的可执行文件，才能执行。   
目前，最常见的 C 语言编译器是自由软件基金会推出的 GCC 编译器 ，可以免费使用。Linux 和 Mac 系统可以直接安装 `GCC`，Windows 系统可以安装 `MinGW`

?> 补充知识：MinGW和GCC的区别：GCC是一个跨平台的编译器集合，可用于多种操作系统和处理器架构，包括Windows；而MinGW是GCC在Windows平台上的移植版本，主要用于在Windows上本地编译C和C++代码   
  
?> MinGW 的全称是：`Minimalist GNU on Windows` 。它实际上是将经典的开源 C语言 编译器 GCC 移植到了 Windows 平台下，并且包含了 Win32API ，因此可以将源代码编译为可在 Windows 中运行的可执行程序。而且还可以使用一些 Windows 不具备的，Linux平台下的开发工具。一句话来概括：**MinGW 就是 GCC 的 Windows 版本** 。以上是 MinGW 的介绍，MinGW-w64 与 MinGW 的区别在于 MinGW 只能编译生成32位可执行程序，而 `MinGW-w64` 则可以编译生成 64位 或 32位 可执行程序。

?> 正因为如此，`MinGW` 现已被 `MinGW-w64` 所取代，且 MinGW 也早已停止了更新，内置的 GCC 停滞在了 4.8.1 版本


[MinGW 下载](https://sourceforge.net/projects/mingw/files/)   

[MinGW-w64 官网](https://www.mingw-w64.org/)

## C程序运行机制

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/c语言运行机制.png)


## 标准库、头文件
printf() 是在标准库的头文件 stdio.h 中定义的。要想在程序中使用这个函数，必须在源文件头部引入这个头文件

```c
#include <stdio.h>
```

C 语言自带的所有这些功能，统称为`标准库`(standard library) ，包含C内置函数、常量和头文件   
如果系统自带某一个功能，就一定会自带描述这个功能的头文件，比如 printf() 的头文件就是系统自带的`stdio.h` 。头文件的后缀通常是 `.h`

## 预处理命令：#include命令

如果要使用某个功能，就必须先加载其对应的头文件，加载使用的是 `#include` 命令，声明在各文件模块的开头。C语言中以 # 号开头的命令称为 预处理命令 。    
顾名思义，在编译器对当前C程序进行编译前执行预处理操作

```c
#include <stdio.h>： //编译系统在系统头文件所在目录搜索
#include "stdio.h"： //编译系统首先在当前的源文件目录中查找stdio.h，找不到的话，再转向系统头文件所在目录搜索。
```

- 引用系统头文件，两种形式都会可以， `#include <>` 效率高。
- 引用用户头文件，只能使用 `#include ""`

## 占位符

- `%a` ：浮点数(仅C99有效)  
- `%A` ：浮点数(仅C99有效) 
- `%c` ：char型数据 
- `%d` ：十进制整数(int) 
- `%e` ：使用科学计数法的浮点数，指数部分的 e 为小写 
- `%E` ：使用科学计数法的浮点数，指数部分的E 为大写 
- `%i` ：整数，基本等同于 %d 
- `%f` ：浮点数(float) 
- `%g` ：6个有效数字的浮点数。整数部分一旦超过6位，就会自动转为科学计数法，指数部分的 e 为小写 
- `%G` ：等同于 %g ，唯一的区别是指数部分的 E 为大写 
- `%hd` ：十进制 short int 类型 
- `%ho` ：八进制 short int 类型 
- `%hx` ：十六进制 short int 类型 
- `%hu` ：unsignedshort int 类型 
- `%ld` ：十进制整数(long) 
- `%lo` ：八进制 long int 类型 
- `%lx` ：十六进制 long int 类型 
- `%lu` ：unsigned long int 类型
- `%lld` ：十进制 long long int 类型 
- `%llo` ：八进制 long long int 类型
- `%llx` ：十六进制 long long int 类型 
- `%llu` ：unsigned long long int类型 
- `%le` ：科学计数法表示的 long double 类型浮点数 
- `%lf` ：十进制浮点数(double) 
- `%n` ：已输出的字符串数量。该占位符本身不输出，只将值存储在指定变量之中 
- `%o` ：八进制整数 
- `%p` ：指针 
- `%s` ：字符串 
- `%u` ：十进制无符号整数（unsigned int）
- `%x` ：十六进制整数 
- `%zd` ： size_t 类型 
- `%%` ：输出一个百分号


# 基本数据类型  
## 整数类型  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/c数据类型.png)

在C语言中，没有`字符串类型` ，使用**字符数组**表示字符串

- 整型：短整型(short)、整型(int)、长整型(long)、更长的整型(long long)
- 每种类型都可以被 signed 和unsigned 修饰
  - 使用 `signed` 修饰 ，表示该类型的变量是带符号位的，有正负号，可以表示负值
  - 使用 `unsigned` 修饰 ，表示该类型的变量是不带符号位的，没有有正负号，只能表示`零和正整数`

- `bit(位)`：计算机中的最小存储单位。
- `byte(字节)`：计算机中基本存储单元

> 1byte  = 8 bit 也就是8个二进制单元，能组成2^8 种可能最大值 为 11111111 即十进制的255，表示范围 0-255     
> 据此可以推算 如下图：  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/c语言数据类型范围.png)


不同计算机的int 类型的大小是不一样的，比较常见的是使用4个字节存储一个int类型的值  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/c数据类型不同编译器结果.png)

?> C标准虽然没有具体规定各种类型数据所占用存储单元的长度，但几条铁定的原则（ANSI/ISO制订的）`sizeof(shortint) ≤ sizeof(int) ≤ sizeof(long int) ≤ sizeof(long long)`，具体由各编译系统自行决定的。其中，sizeof是测量类型或变量长度的运算符。

?> short int至少应为2字节，long int至少应为4字节。


## 精确宽度类型
C 语言的整数类型（short、int、long）在不同计算机上，占用的字节宽度可能是不一样的，无法提前知道它们到底占用多少个字节

精确宽度类型(exact-width integer type)：保证某个整数类型的宽度是确定的。这些类型需要引入头文件 `stdint.h` 

**在物联网终端开发比较常见**

- `int8_t` ：8位有符号整数
- `int16_t` ：16位有符号整数
- `int32_t` ：32位有符号整数
- `int64_t` ：64位有符号整数
- `uint8_t` ：8位无符号整数
- `uint16_t` ：16位无符号整数
- `uint32_t` ：32位无符号整数
- `uint64_t` ：64位无符号整数

上面这些都是类型别名，编译器会指定它们指向的底层类型。比如，某个系统中，如果 int 类型为32位， int32_t 就会指向 int ；如果long 类型为32位， int32_t 则会指向 long

## 浮点类型  

- float类型占用 4字节空间
- double类型占用 8字节空间
- long double类型占用 16字节空间

?> 所有编译器16，32，64位编译器中 float 和double 都是4字节和8字节，long double 32位编译器12字节，64位编译器 16字节

浮点型变量不能使用signed或unsigned修饰符，最常用的浮点类型为：double 类型，因为精度比float高  
浮点型常量，默认为 double 类型，如果希望指定为float类型，需要在小数后面添加后缀 `f` 或 `F`     
同理可以后缀 `l` 或者 `L` 表示**long double类型**


## 字符类型 
使用 `char` 关键字来表示字符型，用于存储一个单一字符 
每个字符变量，在16位、32位或64位编译器中都是 占用 `1 个字节(=8位)`    


每个字符对应一个整数（由 ASCII 码确定），比如 A 对应整数 65  

```c
char c = 65;
//等价于
char c = 'A';

char a = 'B';//66
char b = 'C';//77
printf("%d/n",a+b);//输出133

```

## 布尔
`C89`中没有为布尔值设置一个类型,使用0代表false,非0代表true  

```c
int main(){
  int handsome = 1;
  if(handsome){
    printf("handsome\n");
  }
  return 0;
}
```

大部分情况下使用宏定义来实现,可以更加直观 

```c
#define BOOL int
#define TRUE 1
#define FALSE 0

int main(){
  BOOL handsome = TRUE;
  if(handsome){
    printf("handsome\n");
  }
  return 0;
}

```

C99标准添加了类型`_Bool`来标识布尔值,这个类型的值其实只是整数类型的别名，还是使用 0 表示false， 1 表示true，其它非0的值都会被存储为1  

## 变量间运算规则  

### 隐式转换  

```c

//整数赋值给浮点变量,会自动转为浮点
float y = 12*2;

//char类型和int类型混合运算,会自动提升为int
int a = 10;
char b = 2;
int c = a + b;

//宽度小于int的类型 char short 二者运算的结果是int类型
char c1 = 10;
short s1 = 10;
int i1 = c1 + s1;

//字节宽度较大 的类型，赋值给 字节宽度较小 的变量时，会发生类型降级，自动转为后者的类型
//可能发生截值  
double pi = 3.14159;
int i = pi; // i 的值为 3

float f1 = 1.1f; //ok
double d2 = 4.58667435;
f1 = d2; // 出现精度损失 (double -> float )

```

### 运算溢出
每一种数据类型都有数值范围，如果存放的数值超出了这个范围（小于最小值或大于最大值），需要更多的二进制位存储，就会发生溢出。大于最大值，叫做向上溢出（overflow）；小于最小值，叫做向下溢出（underflow）

```c
// unsign char 类型，最大值是255 （二进制 11111111 ），加 1 后就发生了溢出， 256 （二进制 100000000 ）的最高位 1 被丢弃，剩下的值就是 0 
unsigned char x = 255;
x = x + 1;
printf("%d\n", x); // 0


//常量 UINT_MAX 是 unsigned int 类型的最大值。如果加 1 ，对于该类型就会溢出，从而得到 0 ；而 0 是该类型的最小值，再减 1 ，又会得到 UINT_MAX 
unsigned int ui = UINT_MAX;  // 4,294,967,295
ui++;
printf("ui = %u\n", ui); // 0
ui--;
printf("ui = %u\n", ui); // 4,294,967,295

```

?> 溢出很容易被忽视,编译器不会报错,必须小心

## 常量  
C语言中常量分为一下几种:
- 字面常量
- `#define` 定义的宏常量
- `const` 限定符定义的常量
- `enum` 定义的枚举常量

```c
#include <stdio.h>
#define PI 3.14 //宏常量

//枚举
enum Sex{
    MALE,
    FEMALE,
    OTHER
};

int main() {
    double area;
    double radius = 1.2;
    //const
    const float PIX = 3.14f;
    printf("PI = %f\n",PI);
    printf("PIX = %f\n",PIX);
    printf("%d\n", MALE);
    printf("%d\n", FEMALE);
    return 0;

}
```

# 指针  
为了能够有效的访问到内存的**每个单元(即一个字节)** ，就给内存单元进行了编号，这些编号被称为该内存单元的地址。因为**每个内存单元都有地址**，所以变量存储的数据也是有地址的    

通过地址能找到所需的变量单元，**地址指向该变量单元,这个(内存)地址也叫指针**

?> 注意区分变量名,变量值,变量地址(指针),指针变量只能存放地址    

```c  
//三种指针写法 ,一般使用最后一种  
int   *intPtr; 
int * intPtr;
int*  intPtr;

// 两个指针变量
int * foo, * bar;

// 只有foo是指针变量,bar是整数变量
int* foo, bar;

// 指向指针的指针变量
int **foo1;
int ** foo1;
int** foo1;

```

## 取值运算符 *
指针代表内存地址,如果要获取某个指针的值,就需要使用取值运算符 `*`

```c
// intPtr 是一个指向 int 类型变量的指针, *intPtr 表示 intPtr 所指向的 int 类型变量的值
int   *intPtr;  


void increment(int* p) {
  *p = *p + 1;
}

```

?> 上面的方法参数是一个整数指针`p`,`*p`就标识指针所指的值,`*p+1`就表示指针所指的值加1,也就是将指针所指的值加1,但是这个方法不需要返回值,因为直接操作内存中的值.对于需要大量存储空间的大型变量，复制变量值传入函数，非常浪费时间和空间，不如传入指针来得高效

## 取址运算符 & 
取址运算符 `&` 用于获取变量的地址,`&` 放在变量名前,返回变量的地址

```c
int num = 10;
printf("num = %d\n", num); // 输出变量的值。 num = 10
printf("&num = %p\n", &num); // 输出变量的内存地址。&num = 00000050593ffbbc
```

结合上面的 `*` 运算符可以这样  

```c
void increment(int* p) {
  *p = *p + 1;
}

int x = 1;
increment(&x);
printf("%d\n", x); // 2
```

?> &运算符与*运算符互为逆运算，下面的表达式总是成立。

```c
int i = 5;
int* p = &i;

if (i == *(&i)) // true正确
if(&*p == &i) //true
if(*&i == i) //true
```

## 指针初始化

使用 `int* p;`创建一个指针的时候,编译器会为指针变量本身分配一个内存空间，但是这个内存空间里面的值是**随机的**,这时一定不能去读写指针变量指向的地址，因为那个地址是随机地址，很可能会导致严重后果,更不能对 `*p`进行赋值,这样也会改变那个随机地址的值,导致错误结果    

```c
int* p;
*p = 1; // 错误
```

正确做法是指针变量声明后，必须先让它指向一个分配好的地址，然后再进行读写，这叫做指针变量的初始化,同时为了防止读写未初始化的指针变量,可以将未初始化的指针变量设置为`NULL`   
`NULL`在 C 语言中是一个常量，表示地址为0的内存空间，这个地址是无法使用的，读写该地址会报错

```c
int* p;
int i;

p = &i;
*p = 13;

int* p = NULL;
```

## 指针运算
指针和整数值的加减运算  

```c
short *s;
s = (short *) //0x1234;
printf("%hx\n", s + 1); //0x1236

printf("%hx\n", s - 1); //0x1232

```

`s + 1` 表示指针向内存地址的高位移动一个单位，而**一个单位的 `short` 类型占据两个字节的宽度**，所以相当于向高位移动两个字节    

如果是 int类型(移动四字节) 可以参考下面的图片,其中的`+1`操作并不是`地址+1`,而是`指针p`指向数组的下一个数据   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/c指针加1int示例.png)


对于长度是 N 的一维数组 a，当使用指针 p 指向其首元素后，即可通过指针 p 访问数组的各个元素  
- `a[0]`就是`*p`
- `a[1]`就是`*(p+1)`
- `a[i]`就是`*(p+i)`



```c
int arr[5] = {10,20,30,40,50};

int* p = arr;
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/c语言指针计算加法.png ':size=60%')

对`数组arr`我们有两种遍历方法   

```c
int main() {
  int arr[5] = {10,20,30,40,50};
  //方式1：传统直接访问的方式
  for(int i = 0;i < 5;i++){
    printf("%d ",arr[i]);
 }

  printf("\n");

  //方式2：使用指针访问
  int *p = &arr[0];
  for(int i = 0;i < 5;i++){
    printf("%d ",*(p+i));
 }
  return 0;
}
```

**指针的自增自减**

```c
int main() {
    int arr[5] = {1, 2, 3, 4, 5};
    int *p1 = &arr[0];
    int *p2 = &arr[3];
    printf("p1的值为：%d\n", *p1); //1
    printf("++p1的值为：%d\n", *(++p1)); //2

    printf("++p1的值为：%d\n", *(p1++)); //2 这里执行完*p1已经是3了 
    printf("p1的值为：%d\n", *p1); //3

    printf("p1的地址为：%p\n", p1);//00000055c0bff704
    printf("p1++的地址为：%p\n", ++p1);//00000055c0bff708
    printf("p2的值为：%d\n", *p2); //4
    printf("--p2的值为：%d\n", *(--p2)); //3
    printf("p2的值为：%d\n", *p2); //3

    return 0;
}

```

指针之间的相减运算返回它们之间的距离, 也就是**相隔多少个数据单位（注意：非字节数）** ,高位地址减去低位地址，返回的是正值；低位地址减去高位地址，返回的是负值    
返回的值属于 `ptrdiff_t` 类型，这是一个带符号的整数类型别名，具体类型根据系统不同而不同。这个类型的原型定义在头文件`stddef.h` 里面

指针之间的比较运算，比如 ==、!= 、<、 <= 、 >、 >=。比较的是各自的内存地址的大小，返回值是整数 1`true`或 0`false`    

## 野指针  
指针指向的位置不可知的指针,称为野指针  

野指针的成因:

1. **指针未初始化** ,没有显式初始化的情况下,对其的操作都是错误的,对于未初始化的指针,不能赋值(指向未知),不能获取它的值(值未知)

```c
int main() {
    int* p;
    printf("%d\n",*p);//未知值

    *p = 10;//因为地址是未知的,赋值可能导致错误发生

    int num = 10;
    p = num;//错误
    return 0;
}

```

2. **指针越界访问** ,数组 int arr[6] ,访问 arr[7], 数组下标越界
3. 指针指向已释放的空间,方法返回一个局部变量的指针,方法执行完后,该指针已经无效


总结: 
- 使用前必须初始化,赋值NULL或者指向一个已知的地址(显式赋值)  
- 使用数组时小心指针越界
- 避免返回局部变量地址
- 指针指向空间释放,即时置NULL
- 指针使用前检查有效性   

## 二级指针  

多余套娃


# 数组 

```c
//声明数组 
int scores[100];

//指定元素赋值
scores[0] = 13;
scores[99] = 42;

//数组越界,不会报错,使得紧跟在scores后面的那块内存区域被赋值，会造成不可预估的问题
int scores[100];
scores[100] = 51;

//声明时，使用大括号，同时对每一个成员赋值
int a[5] = {22, 37, 3490, 18, 95};

//使用声明后不能再进行集体赋值,也就是大括号赋值 
int a[5];
a = {22, 37, 3490, 18, 95}; // 报错


//如果大括号里面的值，少于数组的成员数量，那么未赋值的成员自动初始化为0
int a[5] = {22, 37, 3490};
// 等同于
int a[5] = {22, 37, 3490, 0, 0};

// 整个数组的每一个成员都设置为零
int a[100] ={0}

//指定初始化成员,其他的自动设置为0
int a[15] = {[2] = 29, [9] = 7, [14] = 48};

// 指定位置的赋值与顺序赋值，可以结合使用 
// 分别对 0 5 6 10 11赋值
int a[15] = {1, [5] = 10, 11, [10] = 20, 21}


//赋值后自动确定长度 
int a[] = {22, 37, 3490};
// 等同于
int a[3] = {22, 37, 3490};

//同样可以确定长度 10 最后是9号元素
int a[] = {[2] = 6, [9] = 12};
```

数组是一连**串连续储存的同类型值,地址也是连续地址**，只要获得起始地址（首个成员的内存地址），就能推算出其他成员的地址(根据成员大小)   
数组一旦声明,数组名指向的地址就不可更改(所以在上面,声明后的数组是不能重新赋值的)    

```c
int a[5] = {1,2,3,4,5};
```

上面的数组对应的内存结构是连续的,如图   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/C语言数组内存结构.png)   

```c
int a[5] = {11, 22, 33, 44, 55};
int* p;

p = &a[0];

printf("%d\n", *p);  // Prints "11"
```

`&a[0]`就是数组`a`的首个成员11的内存地址，也是整个数组的起始地址。反过来，从这个地址可以获得首个成员的值11`*p`  

**C语言中,数组名等同于起始地址,也就是说,数组名就是指向第一个成员的指针**

```c
int a[5] = {11, 22, 33, 44, 55};

int* p = &a[0];
// 等同于
int* p = a;
```

## sizeof运算符

`sizeof` 用来计算一个类型/对象的所占用的内存大小。对于数组就会返回整个数组的字节长度   

```c
int a[] = {22, 37, 3490};
int arrLen = sizeof(a); // 12  4*3

// 数组中的乘员数量可以由下面的公式获得
sizeof(a) / sizeof(a[0])

```

`sizeof`返回值的数据类型是`size_t`，所以`sizeof(a) / sizeof(a[0])`的数据类型也是`size_t`。在`printf()`里面的占位符，要用`%zd`或`%zu` 

```c
int x[12];

printf("%zu\n", sizeof(x));     // 48
printf("%zu\n", sizeof(int));  // 4
printf("%zu\n", sizeof(x) / sizeof(int)); // 12

```


## 变长数组 

数组声明的时候,通过变量指定数组大小,所以变长数组的大小只有在运行时才能确定   

```c
int n = 10;
int arr[n];

```

变长数组在C99标准中引入,在C11标准中被标记为可选特性,在不支持变长数组的编译器版本中,可以使用动态内存分类函数`malloc()`来创建动态大小的数组 

```c
int length = 5;
//将arr的地址指向分配的内存空间
int *arr = (int *)malloc(sizeof(int) * length);

//使用完毕释放内存,否则内存泄漏
free(arr)

```

## 数组的复制  
数组名是指针，所以复制数组不能简单地复制数组名  

```c
int* a;
int b[3] = {1, 2, 3};

a = b;
```

这样只能把a也指向b，而不是复制b。如果需要复制b，可以循环赋值或者使用`memcpy()`函数(定义在头文件`string.h`)


```c

for (i = 0; i < N; i++)
  a[i] = b[i];

//或者
memcpy(a, b, sizeof(b));
```

## 多维数组  

```c
//二维数组
int board[10][10];

//三维数组
int c[4][5][6];

//一次性赋值
int a[2][5] = {
  {0, 1, 2, 3, 4},
  {5, 6, 7, 8, 9}
};

//指定位置 ,其他位置为0
int a[2][2] = {[0][0] = 1, [1][1] = 2};

int a[2][2] = {1, 0, 0, 2};  //会自动匹配到各行各列

//下面两种是相同的
int a[][3] = {1, 2, 3, 4, 5, 6};
int a[][3] = {{1, 2, 3},{4, 5, 6}};

//这样也是可以的,缺少的补位0
int arr5[][2]={1,2,3,4,5};

```

**在定义二维数组时，必须指定列数**   

```c
int array[][];  //错误，必须指定列数
int array[3][]; //错误，必须指定列数
```

C语言中,二维数组中元素排列的顺序是`按行存放`的,先顺序存放第一行的元素,在存放第二行的元素   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/二维数组的内存顺序.png)


二维数组的长度  

```c
int b[3][3];

printf("%d\n",sizeof(b)); //36
printf("%d\n",sizeof(b)/sizeof(int)); //9
```

## 数组作为函数的参数

数组作为函数的参数，一般会同时传入`数组名`和`数组长度`

```c
int sum_array(int a[], int n) {
  // ...
}

int a[] = {3, 5, 7, 3};
int sum = sum_array(a, 4);

//对二维数组来说 n作为第一维的长度

int sum_array(int a[][4], int n) {
  // ...
}

int a[2][4] = {
  {1, 2, 3, 4},
  {8, 9, 10, 11}
};
int sum = sum_array(a, 2);

// 数组变量作为参数
int a[] = {2, 3, 4, 5};
int sum = sum_array(a, 4);

// 数组字面量作为参数
int sum = sum_array((int []){2, 3, 4, 5}, 4);

```


# 数组和指针

##  数组指针访问 
数组名可以直接进行加减法运算,等同于指针在数组间前后移动,即从一个成员的内存地址移动到下一个成员的内存地址,比如 对于数组`a`来说 `a+1` 返回下一个成员的`内存地址`,`a-1` 返回上一个成员的`内存地址`    


```c

int a[5] = {11, 22, 33, 44, 55};

//*(a + i) 取出地址的值 
for (int i = 0; i < 5; i++) {
  printf("%d\n", *(a + i));
}

```

上面代码中的`*(a + i)`等同于 `a[i]`   
**数组名等价于指针,所以`a[b] ==*(a+b)`**

```c
//新的遍历方式
int a[] = {11, 22, 33, 44, 55, 999};

int* p = a;

while (*p != 999) {
  printf("%d\n", *p);
  p++;
}
```

数组名指向的地址是不能变的，所以上例中，不能直接对数组`a`进行自增，即`a++`的写法是错的，必须将a的地址赋值给指针变量p，然后对p进行自增  

## 数组的地址 

```c
int a[5] = {11, 22, 33, 44, 55};
int* p;

// 下面两个写法等价
p = &a[0];
p = a;


//在此基础上,可以用p 或a 进行遍历
int main() {
    int arr[5] = {1, 2, 3, 4, 5};
    int* p = &arr[0];

    //1
    for (int i = 0; i <5; i++) {
        printf("%d\n", arr[i]);
    }
    //2
    for (int i = 0; i <5; i++) {
        printf("%d\n", *(arr+i));
    }
    //3
    for (int i = 0; i <5; i++) {
        printf("%d\n", *(p + i));
    }
    //4
    for (int i = 0; p <arr+5; p++) {
        printf("%d\n", *p);
    }

    return 0;

}

```

上面的遍历方式中,第一种和第二种是等价的,C编译系统将`arr[i]`转换为`*(arr+i)`进行处理,即先计算元素地址,再根据地址取值    
第三种方法是直接根据地址取值,大大提高效率


```c
int main() {
    int arr[5] = {1, 2, 3, 4, 5};
    int* p = &arr[2];

    printf("%p\n", arr);//000000891f5ffa20
    printf("%p\n", p);//000000891f5ffa28 地址差值 2位 int 8位
    printf("%d\n", *p);//3
    printf("%d\n", p[0]);//3
    printf("%d\n", p[1]);//4
    printf("%d\n", p[-1]);//2
    return 0;

}

```

如果`p` 指向数组`arr`的第4个元素,那么`*p`就是`arr` 的第3个元素,`p[1]`就是`arr` 的第4个元素,`p[-1]`就是`arr` 的第2个元素



```c
int main() {
    int arr[5] = {1, 2, 3, 4, 5};
    int* p = arr;

    printf("%p\n", p);//0000006071fff7c0
    printf("%p\n", &p);//0000006071fff7b8


    printf("%p\n", arr);//0000006071fff7c0
    printf("%p\n", &arr);  //0000006071fff7c0
    printf("%p\n", &arr+1);//0000006071fff7d4

    return 0;

}
```

上面的`p` 指向数组`arr`的地址,也就是`arr[0]`的地址;   
`&p`指向`p`的地址;   
`&arr`指向数组`arr`,二者是相同的;    
`&arr+1` 相当于移动了一个`arr`的长度,也就是5个int的长度,可以看到地址差值是20个字节



## 二维数组和指针   

```c
int main() {
    int arr[3][4] = {{1,2,3,4},
                     {5,6,7,8},
                     {9,10,11,12}};
    //第一部分
    printf("%p\n",arr); //000000dac49ffa50
    printf("%p\n",arr[0]);//000000dac49ffa50
    printf("%p\n",arr[1]);//000000dac49ffa60
    printf("%p\n",arr+1);//000000dac49ffa60

    printf("%d\n",arr[0][0]);//1
    printf("%p\n",&arr[0][0]);//000000dac49ffa50
    printf("%p\n",&arr[0][1]);//000000dac49ffa54
    printf("%p\n",&arr);//000000dac49ffa50

    return 0;

}
```

从第一部分中可以看出,`arr`,`arr[0]`,`arr[0][0]`是相同的,都是数组`arr`的地址也就是`a[0][0]`的地址,`arr[1]`的地址和`arr[0]`的地址差了16个字节,也就是4个int的大小, `arr[0][0]`的地址和`arr[0][1]`差了4个字节,也就是一个int的大小   

















# 字符串
C 语言没有单独的字符串类型，字符串被当作字符数组，即`char`类型的数组,字符串`"Hello"`是当作数组`{'H', 'e', 'l', 'l', 'o'}`处理的    
在字符串结尾，C 语言会自动添加一个全是二进制`0`的字节，写作`\0`字符，表示字符串结束,字符串`"Hello"`实际储存的数组是`{'H', 'e', 'l', 'l', 'o', '\0'}`   

`char localString[10];`声明了一个10个成员的字符数组，可以当作字符串。由于必须留一个位置给`\0`，所以最多只能容纳9个字符的字符串

**C语言中双引号之中的字符，会被自动视为字符数组**

```c
{'H', 'e', 'l', 'l', 'o', '\0'}

// 等价于
"Hello"

//如果字符串内部包含双引号，则该双引号需要使用反斜杠转义
"She replied, \"It does.\""

//如果字符串过长，可以在需要折行的地方，使用反斜杠（\）结尾，将一行拆成多行,第二行必须顶格书写
"hello \
world"

```

即使双引号里面只有一个字符（比如`"a"`），也依然被处理成字符串（存储为2个字节），而不是字符`'a'`（存储为1个字节）


```c
char greeting[50] = "Hello, ""how are you ""today!";
// 等同于
char greeting[50] = "Hello, how are you today!";
//等同于
char greeting[50] = "Hello, "
  "how are you "
  "today!";

```

## 声明字符串
字符串可以声明成一个指针或字符数组  

```c
// 写法一
char s[14] = "Hello, world!";
// 或者
char s[] = "Hello, world!";

// 写法二
char* s = "Hello, world!";

```


上述两种写法略有差异,区别如下    

```c

//1. 指针形式不能修改某个字符
char* s = "Hello, world!";
s[0] = 'z'; // 错误

char s[] = "Hello, world!";
s[0] = 'z';

//2. 数组形式不能重新赋值,这个和数组的定义相同
char* s = "hello";
s = "world";

char s[] = "hello";
s = "world"; // 报错

```

**思考**    
针对`char s[10] = "hello";` 字符数组s的长度是`10`，但是字符串“hello”的实际长度只有6（包含结尾符号`\0`），所以后面空出来的4个位置，都会被初始化为`\0`

```c
int main() {
    char s[10] = "hello";
    printf("%s", "==");
    for (int i = 0; i < sizeof(s);i++) {
        if(s[i]== '\0'){
            printf("%c", 'f');
        }else{
            printf("%c", s[i]);
        }
    }
    printf("%s", "==");
    return 0;
}

//输出 ==hellofffff==
```

那么如果刚好相等又是什么情况?`char s[5] = "hello";` 会导致什么问题呢?

```c
int main() {
    char s[5] = "hello";
    printf("%s", "==");
    for (int i = 0; i < sizeof(s);i++) {
        if(s[i]== '\0'){
            printf("%c", 'f');
        }else{
            printf("%c", s[i]);
        }
    }
    printf("%s\n", "==");
//上面打印内容 ==hello==

    printf("%s\n", s);//hello 这里hello后面是有一位的'\0' 也就是系统自动补全了'\0'
    printf("%d\n", strlen(s)); //6 
    printf("%d\n", sizeof(s)); //5
    return 0;
}

```

如果是`char s[6] = "hello";`呢?

```c
int main() {
    char s[6] = "hello";
    printf("%s", "==");
    for (int i = 0; i < sizeof(s);i++) {
        if(s[i]== '\0'){
            printf("%c", 'f');
        }else{
            printf("%c", s[i]);
        }
    }
    printf("%s\n", "==");
    //==hellof==

    printf("%s\n", s);//hello 这里就是一个hello 没有后缀的'\0'
    printf("%d\n", strlen(s)); //5
    printf("%d\n", sizeof(s)); //6
    return 0;
}
```

**注意`strlen()`返回值的差异**


## 常用函数 

`strlen()` 函数,返回字符串的字节长度,不包括末尾的空字符`\0`,使用时需要加载头文件`string.h`   

```c
// string.h
size_t strlen(const char* s);

char* str = "hello";
int len = strlen(str); // 5
```

注意和`sizeof()`的区别,`sizeof()`返回的是字节数,`strlen()`返回的是字符个数

```c

char s[50] = "hello";
printf("%d\n", strlen(s));  // 5
printf("%d\n", sizeof(s));  // 50 

```

`strcpy()`函数，用于将一个字符串的内容复制到另一个字符串，相当于字符串赋值

```c
//API 
strcpy(char dest[], const char source[])

#include <stdio.h>
#include <string.h>

int main(void) {
  char s[] = "Hello, world!";
  char t[100];

  strcpy(t, s);

  t[0] = 'z';
  printf("%s\n", s);  // "Hello, world!"
  printf("%s\n", t);  // "zello, world!"
}
```

`strcpy()`的返回值是一个字符串指针（即`char*`），指向第一个参数

```c
char* s1 = "beast";
char s2[40] = "Be the best that you can be.";
char* ps;

ps = strcpy(s2 + 7, s1);

puts(s2); // Be the beast
puts(ps); // beast
```

从`s2`的第7个位置开始拷贝字符串`beast`，前面的位置不变。这导致s2后面的内容都被截去了，因为会连beast结尾的空字符一起拷贝,读取字符串时,发现了`\0`就会返回,如果使用如下代码测试即可看到`s2`的全貌

```c

int main() {

    char* s1 = "beast";
    char s2[40] = "Be the best that you can be.";
    char* ps;

    ps = strcpy(s2+7,s1);
    puts(s2);
    puts(ps);

    printf("%s", "==");
    for (int i = 0; i < sizeof(s2);i++) {
        if(s2[i]== '\0'){
            printf("%c", 'f');
        }else{
            printf("%c", s2[i]);
        }
    }
    printf("%s\n", "==");
    return 0;
}
```

代码输出 `==Be the beastfhat you can be.ffffffffffff==`,可以看到原来 `best `(带空格)的位置被`beast`替换,并顺序补入了`\0`,导致that的`t`被替换,其实对于`字符数组s2`来说,是可以看到所有改动的,但是对于`字符串s2`来说就只有`\0`前的内容了    

`strcpy()`函数有安全风险，因为它并不检查目标字符串的长度，是否足够容纳源字符串的副本，可能导致写入溢出。如果不能保证不会发生溢出，建议使用`strncpy()`函数代替

`strncpy()跟`strcpy()`的用法完全一样，只是多了第3个参数，用来指定复制的最大字符数，防止溢出目标字符串变量的边界,第三个参数n定义了复制的最大字符数。如果达到最大字符数以后，源字符串仍然没有复制完，就会停止复制     

```c
char s1[40];
char s2[12] = "hello world";

strncpy(s1, s2, 5);
s1[5] = '\0';

printf("%s\n", s1); // hello
```

`strcat()`函数用于连接字符串。它接受两个字符串作为参数，把第二个字符串的副本添加到第一个字符串的末尾,这个函数会改变第一个字符串，但是第二个字符串不变

```c
char s1[12] = "hello";
char s2[6] = "world";

strcat(s1, s2);
puts(s1); // "helloworld"
```


`strncat()`用于连接两个字符串，用法与`strcat()`完全一致，只是增加了第三个参数，指定最大添加的字符数,在添加过程中，一旦达到指定的字符数，或者在源字符串中遇到空字符`\0`，就不再添加了   

```c
char s1[10] = "Monday";
char s2[8] = "Tuesday";

strncat(s1, s2, 3);
puts(s1); // "MondayTue"
```

`sprintf()`函数跟`printf()`类似，但是用于将数据写入字符串，而不是输出到显示器。该函数的原型定义在`stdio.h`头文件里面  
`snprintf()`只比`sprintf()`多了一个参数n，用来控制写入变量的字符串不超过`n - 1`个字符，剩下一个位置写入空字符`\0`

```c

char first[6] = "hello";
char last[6] = "world";
char s[40];

sprintf(s, "%s %s", first, last);

printf("%s\n", s); // hello world

```

## 练习题

下面是一些C Primer Plus的习题

```c
//输出什么？？
int main() {
    char note[] = "see you at the snack bar. ";
    char*  ptr;
    ptr = note;

    puts(ptr);//see you at the snack bar. 
    puts(++ptr);//ee you at the snack bar. 
    note[7] = '\0';
    puts(note);//see you
    puts(ptr);//ee you
    puts(++ptr);//e you
    return 0;
}

//输出什么？？
int main() {
    char food[] = "Yummy";
    char*  ptr;
    ptr = food+ strlen(food);//将ptr的指针指向了food末尾

    while (--ptr>=food)
        puts(ptr);

    return 0;
}
//输出
// y
// my
// mmy
// ummy
// Yummy
```


# 函数  

## 函数声明定义
函数的声明有一下几点   

1. 函数的返回值类型,函数必须给出返回值类型,如果没有返回值,使用void代替   
2. 参数,如果没有参数也是用void代替
3. 函数体
4. `return` 语句,没有返回值可以省略

?> 没有返回值的函数，使用void关键字表示返回值的类型。  
没有参数的函数，声明时要用void关键字表示参数类型(可以省略)   


如果返回值类型非 void，但被调函数中没有 return 语句，函数会返回一个不确定的值   

```c
int test(){

    printf("hello\n");
    printf("hello dfasdf\n");
    printf("he\n");
//    return 0;
}

int main() {
    int  i =test();
    printf("%d\n",i);
    return 0;
}
```

上述代码打印结果是`he3`,如果最后一个打印内容是`printf("he");` 那么打印结果是`he2`,如果是`printf("hello");`那么打印结果是`he5`,也就是最后一个打印内容的字符长度,这么看也不是不确定的值哈哈

**如果省略函数的返回值类型，默认为int类型,但是不要这么写**
在CLion中会得到提示`Type specifier missing, defaults to int'; ISO C99 and later do not support implicit int`,编译通过可以运行,但是会有警告

## main函数

`main()`是程序的入口函数，即所有的程序一定要包含一个main()函数。程序总是从这个函数开始执行，如果没有该函数，程序就无法启动。其他函数都是通过它引入程序的

```c
int main(void) {
  printf("Hello World\n");
  return 0;
}
```

?> C 语言约定，返回值0表示函数运行成功，如果返回其他非零整数，就表示运行失败，代码出了问题。  
系统根据`main()`的返回值，作为整个程序的返回值，确定程序是否运行成功  
如果`main()`里面省略`return 0`这一行，编译器会自动加上(其他函数不会)，**即main()的默认返回值为0**   


## 参数的传值

C语言的函数传参都是**值传递**，即函数的参数是实参的拷贝，对形参的修改不会影响实参。
但是,可以通过传递指针的方法实现引用传递


```c
void Swap(int x, int y) {
  int temp;
  temp = x;
  x = y;
  y = temp;
}

int a = 1;
int b = 2;
Swap(a, b); // 无效

//指针实现引用传递
void Swap(int* x, int* y) {
  int temp;
  temp = *x;
  *x = *y;
  *y = temp;
}

int a = 1;
int b = 2;
Swap(&a, &b);
```


## 函数指针   

函数本身就是内存中的一段代码,C语言中可以通过指针获取函数    

```c
void print(int a) {
  printf("%d\n", a);
}

void (*print_ptr)(int) = &print;  
void (*print_ptr)(int) = print;  

//通过指针调用函数  
(*print_ptr)(10)
```

C语言中函数名本身就是指向函数代码的指针(跟数组类似),通过函数名就能获取函数地址,所以`&print`等同于`print`

对于任意函数，都有五种调用函数的写法,实际上是三种,主要是 `*` `&` 和自身的函数调用 

```c
void print(int a) {
    printf("%d\n", a);
}


int main() {
    void (*print_ptr)(int) = &print;
//    void (*print_ptr)(int) = print;
// 写法一
    print(10);

// 写法二
    (*print)(10);

// 写法三
    (&print)(10);

// 写法四
    (*print_ptr)(10);

// 写法五
    print_ptr(10);

    return 0;
}

```

带`*` `&` 的函数调用一般用函数参数或者返回值是一个函数的时候   

```c
//函数compute()的第一个参数也是一个函数
int compute(int (*myfunc)(int), int, int);
```


## 函数原型  
由于函数必须先声明，后使用。但是程序总是先运行`main()`函数，导致所有其他函数都必须在`main()`函数之前声明    
`main()`是整个程序的入口，也是主要逻辑，放在最前面比较好。另一方面，对于函数较多的程序，保证每个函数的顺序正确，会变得很麻烦
所以C语言中,可以在程序开头给出函数原型即可(类似接口和实现),后续函数具体实现放在`main`函数后面

```c
int twice(int);
// 等同于
int twice(int num);


int main(int num) {
  return twice(num);
}

int twice(int num) {
  return 2 * num;
}
```

## exit()

`exit()`函数用来终止整个程序的运行。一旦执行到该函数，程序就会立即结束。该函数的原型定义在头文件`stdlib.h`里面
`exit()`可以向程序外部返回一个值，它的参数就是程序的返回值。一般来说，使用两个常量作为它的参数：`EXIT_SUCCESS`（相当于 0）表示程序运行成功，`EXIT_FAILURE`（相当于 1）表示程序异常中止,这两个常数也是定义在`stdlib.h`里面

```c
// 程序运行成功
// 等同于 exit(0);
exit(EXIT_SUCCESS);

// 程序异常中止
// 等同于 exit(1);
exit(EXIT_FAILURE);
```

C 语言还提供了一个`atexit()`函数，用来登记exit()执行时额外执行的函数，用来做一些退出程序时的收尾工作。该函数的原型也是定义在头文件`stdlib.h`

```c
int atexit(void (*func)(void));
```

`atexit()`的参数是一个函数指针。注意，它的参数函数（下例的print）不能接受参数，也不能有返回值

```c
//exit()执行时会先自动调用atexit()注册的print()函数，然后再终止程序
void print(void) {
  printf("something wrong!\n");
}

atexit(print);
exit(EXIT_FAILURE);
```

## 函数说明符

### extern 

对于多文件的项目，源码文件会用到其他文件声明的函数,用`extern`说明该函数的定义来自其他文件

```c
extern int foo(int arg1, char arg2);

int main(void) {
  int a = foo(2, 3);
  // ...
  return 0;
}
```

?> 函数原型默认就是`extern`，所以这里不加`extern`，效果是一样的

### static 
默认情况下，每次调用函数时，函数的内部变量都会重新初始化，不会保留上一次运行的值   
`static`用于函数内部声明变量时，表示该变量只需要初始化一次，不需要在每次调用时都进行初始化。也就是说，它的值在两次调用之间保持不变  
`static`修饰的变量初始化时，**只能赋值为常量，不能赋值为变量**

```c
#include <stdio.h>

void counter(void) {
  static int count = 1;  // 只初始化一次
  printf("%d\n", count);
  count++;
}

int main(void) {
  counter();  // 1
  counter();  // 2
  counter();  // 3
  counter();  // 4
}
```

在块作用域中，`static`声明的变量有默认值`0`

```c
static int foo;
// 等同于
static int foo = 0;
```

`static`可以用来修饰函数本身,表示该函数只能在当前文件里使用，如果没有这个关键字，其他文件也可以使用这个函数（通过声明函数原型）

```c
static int Twice(int num) {
  int result = num * 2;
  return(result);
}
```

`static`也可以用在参数里面，修饰参数数组,`static`对程序行为不会有任何影响，只是用来告诉编译器，该数组长度至少为3，某些情况下可以加快程序运行速度。另外，需要注意的是，对于多维数组的参数，**static仅可用于第一维的说明**

```c
int sum_array(int a[static 3], int n) {
  // ...
}
```

### const 
`const`说明符，表示函数内部不得修改该参数变量   

```c
// 函数f()的参数是一个指针p，函数内部可能会改掉它所指向的值*p，从而影响到函数外部
void f(int* p) {
  // ...
}

//修改为  
void f(const int* p) {
  *p = 0; // 该行报错,无法修改p的值
}

```

上面的`f(const int* p)` 只能限制修改`*p`的值,不能限制修改p的值,也就是p的地址是可以修改的  

```c
void f(const int* p) {
  int x = 13;
  p = &x; // 允许修改
}
```

想限制修改`p`，可以把`const`放在`p`前面,但是此时`*p`的值是可以修改的

```c
void f(int* const p) {
  int x = 13;
  p = &x; // 该行报错
}
```


如果想同时限制修改`p`和`*p`，需要使用两个const

```c
void f(const int* const p) {
  // ...
}
```

## 可变参数

```c
int printf(const char* format, ...);
```
























# 参考
**内容来自以下平台学习总结:** 

- [B站C语言](https://www.bilibili.com/video/BV1Bh4y1q7Nt)
- [C Primer Plus](https://book.douban.com/subject/1240002/)
- [网道C语言教程](https://wangdoc.com/clang/)