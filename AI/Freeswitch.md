# Freeswitch 笔记   

[官方文档](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/)

# 相关说明
## Freeswitch 和 Kamailio

`Kamailio` 和`FreeSWITCH`都是常用的开源通信软件，但它们的主要功能和使用场景不同。

`Kamailio`是一个`SIP服务器`，专注于呼叫路由、注册、认证、授权等SIP协议处理。它可以作为SIP代理服务器、注册服务器、SIP路由器等，用于构建VoIP网络中的各种应用场景，如`SIP-to-PSTN`网关、IP-PBX系统、实时通信服务等。

而`FreeSWITCH`则是一个电话交换机软件，支持多种语音通信协议（包括`SIP`、`H.323`、`WebRTC`等）以及多种音视频编解码格式，可用于构建高质量的实时语音、视频通信服务。它可以处理来自IP网络 `VoIP`和`PSTN`的通信

一般情况下，`Kamailio`作为SIP代理服务器、注册服务器等，处理来自客户端的SIP请求，将请求路由到正确的终端设备或应用程序。而`FreeSWITCH`则作为电话交换机软件，负责处理音视频通信、呼叫控制等。

具体来说，`Kamailio`和`FreeSWITCH`可以通过SIP协议进行集成，以便在VoIP网络中提供高质量的语音、视频通信服务。例如，可以使用`Kamailio`作为SIP代理服务器，将来自客户端的SIP请求路由到`FreeSWITCH`，由`FreeSWITCH`处理音视频通信部分，并将结果返回给`Kamailio`，再由`Kamailio`将数据传输回客户端


## VoIP 和 PSTN 
`VoIP`指的是`Voice over Internet Protocol(Voice Over IP)`，即互联网电话协议  

`PSTN(Public Switched Telephone Network)` 公共交换电话网,除了为用户提供基本的语音通话外，`PSTN`还能提供一些附加的业务，如叫醒服务、呼叫转移等，为人们的生产和生活提供方便。这些业务有的可以由运营商为用户设置，有的也可以由用户拨打某些特殊的号码自行激活和取消


## PCM
`PCM(Pulse Code Modulation)`的全称是脉冲编码调制。它是一种通用的将模拟信号转换成0和1的数字信号的方法   

一般来说，人的声音频率范围在300~3400Hz之间，通过滤波器将超过4000Hz的频率过滤出去，便得到4000Hz内的模拟信号。然后根据抽样定理，使用8000Hz进行抽样，便得到离散的数字信号。使用PCM方法得到的数字信号就称为PCM信号，一般一次抽样会得到16bit的信息


## 七号信令
七号信令（Signaling System No.7，SS7）是我国目前使用的主要信令方式，用于局间通信。我国的电话网络中有专门的七号信令网。

在此，我们先来看一次简单的固定电话的通话流程。如图1-8所示，用户a摘机，与其相连的交换机A根据电压、电流的变化检测到a摘机后，即向a发送拨号音，同时启动收号程序。a听到拨号音后开始拨号，待交换机A收齐号码后，即查找路由，发送IAM（Initial Address Message，初始地址消息）给交换机B。B向A发ACM（Adress Complete Message，地址全消息）并通知用户b（b的话机）振铃，A向a送回铃音。这时如果b接听电话，则B向A发送ANC（Answer Charge，应答计费消息），a与b开始通话，同时A对a进行计费

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/七号信令.png)

通话完毕，如果主叫挂机，则本端交换机A向对端B发送CLF（Clear Forward，前向释放消息），B向A返回RLG（ReleaseGard，释放监护消息），并向b传送催挂音（嘟嘟嘟……）。

如果被叫挂机，则B向A发送CBK（Clear Backword，后向释放消息），A回送CLF，最后B返回RLG。

上面在交换机A与B之间传递的为七号信令中的TUP（Telephone User Part，电话用户部分）。

## ISUP信令与TUP互通
目前，由于ISUP（ISDN User Part，ISDN用户部分）能与ISDN互联并提供比TUP更多的能力和服务，故其已基本取代TUP成为我国七号信令网采用的主要信令方式。ISUP信令与TUP互通时的对应关系如图

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ISUP信令.png)

端局A与端局B经过汇接局TM汇接通信中ISUP与TUP信令的例子。ISUP信令的初始地址消息有IAM和IAI（IAM With Additional Information，带附加信息的IAM）两种，后者能提供更多的信息（如主叫号码等）。另外ISUP信令的拆线信号不分前后向，只有REL（Release，释放）和RLC（Release Complete，释放完成）

## ISDN PRI信令

ISDN PRI使用SETUP/CONNECT/RELEASE消息分别对应ISUP中的IAM/ANM/REL消息

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ISDN-PRI信令.png)


## H.323与SIP信令
H.323与SIP属于VoIP领域的通信信令，它们适用于用户线信令和局间信令，由于IP终端比普通话机更加智能，因此这些信令在用户线信令及局间信令使用方式上已没有太大区别  

H.323和SIP设计之初都是作为多媒体通信的应用层控制（信令）协议，目前一般用于IP电话，都利用RTP作为媒体传输的协议

ISUP与SIP间的信令转换关系图如下  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ISUP与SIP间的信令转换关系.png)

**SIP借鉴了其他Internet标准和协议的设计思想，有其突出的优点。SIP是基于文本的协议**   
在SIP通信中，除文字外，媒体都是在RTP协议中传输的。由于媒体一般都是持续传输的，因此又称`RTP流`

## PBX
PBX（Private Branch eXchange）的全称是专用小交换机。该设备一般安装在企业内部。PBX的上端通过运营商提供的模拟或数字中继线连接到PSTN，而下端则直接接企业内部的话机

用户或企业PBX要想打通外面的电话，或者外面的电话需要打进来，需要走运营商提供的中继线，以接入到PSTN网上去  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/pbx示意图.png)

## IP-PBX
IP-PBX 首先是一个PBX（Private Branch eXchange），它具有传统PBX的绝大部分功能。另外，由于使用了IP通信，它能通过IP网提供语音、视频以及即时消息通信。这些通信不仅可以在企业内部网上进行，也可以通过Internet在外网甚至PSTN（Public Switched Telephone Network）电话间进行

与传统的PBX相比，IP-PBX支持更多的新特性（或更易于支持某些特性）：

- 无限分机数量、无限自动话务台、无限语音信箱；
- 更易于通过API与其他应用系统（如CRM等）集成；
- 支持远端电话分机、支持软电话；
- 支持高级用户接口，如电话跟随、统一消息、电话录音、语音邮箱、传真集成等；
- 根据时间路由、自定义路由规则；
- 支持从语音邮箱回拨、语音邮箱转Email；
- 支持电话监听、耳语；
- 支持根据名字呼叫；
- 支持话务员控制台、拖拽转移、点击拨号等；
- 支持多人会议室；
- 支持高级IVR（Interractive Voice Response）；
- 支持自动呼叫分配（Automatic Call Distribution，ACD）

## 呼叫中心

在呼叫中心中，有专门的话务员为客户提供服务。呼叫中心通常能同时处理大量的通话，并且为了给客户更好的服务体验，呼叫中心的通信系统通常通过技术手段与CRM（Customer Ralationship Management，客户关系管理）系统集成

## DID
`Direct Inbound Dial`，即对内直接呼入,允许外部呼叫者通过直接拨打一个特定的号码来拨通内部电话系统中的个人或部门。因此，DID号码可以直接与内部分机号码对应，无需经过接待员或自动语音应答系统的转接

## UA
UA是"User Agent"（用户代理）的缩写，它是指与FreeSWITCH进行通信的实体或设备。UA可以是软件应用程序、硬件电话、软电话或其他VoIP终端设备   
在FreeSWITCH中，UA通常是通过SIP（Session Initiation Protocol，会话初始协议）与FreeSWITCH进行通信   

## Leg
单腿通话（one-legged connection）





# 安装配置
待 略


# 使用配置

## 默认号码及说明
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/freeswitch默认号码说明.png)

## 配置FreeSWITCH
 
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/freeswitch配置文件目录结构.png)

## 添加号码用户
FreeSWITCH默认设置了20个用户（1000～1019），如果你需要更多的用户，或者想通过添加一个用户来学习FreeSWITCH配置，只需要简单执行以下三步

- 在`conf/directory/default/`中增加一个用户配置文件`4000.xml`。
- 修改拨号计划（Dialplan）使其他用户可以呼叫到它。
- 重新加载配置使其生效

```xml
<!--添加 4000用户 -->
<include><!-- 引用其他配置文件或片段 -->
  <user id="4000"><!-- 定义一个用户，id为4000 -->
    <params><!-- 用户的参数列表 -->
      <param name="password" value="$${default_password}"/><!-- 设置用户的密码，使用了一个变量${default_password} 默认为1234 -->
      <param name="vm-password" value="4000"/><!-- 设置用户的语音信箱密码 -->
    </params>
    <variables><!-- 用户的变量列表 -->
      <variable name="toll_allow" value="domestic,international,local"/><!-- 设置用户的拨号规则，允许拨打国内、国际和本地电话 -->
      <variable name="accountcode" value="4000"/><!-- 设置用户的账户编码 -->
      <variable name="user_context" value="default"/><!-- 设置用户所属的上下文（context），即用户可以使用的呼叫路由等信息 -->
      <variable name="effective_caller_id_name" value="Extension 4000"/><!-- 设置用户发起呼叫时的主叫显示名称 -->
      <variable name="effective_caller_id_number" value="4000"/><!-- 设置用户发起呼叫时的主叫显示号码 -->
      <variable name="outbound_caller_id_name" value="$${outbound_caller_name}"/><!-- 设置用户发起外呼时的主叫显示名称，使用了一个变量${outbound_caller_name} -->
      <variable name="outbound_caller_id_number" value="$${outbound_caller_id}"/><!-- 设置用户发起外呼时的主叫显示号码，使用了一个变量${outbound_caller_id} -->
      <variable name="callgroup" value="techsupport"/><!-- 设置用户所属的呼叫组（callgroup），用于在多个用户之间进行呼叫分配 -->
    </variables>
  </user>
</include>

```

```shell
# 显示多少用户已注册
sofia status profile internal reg

# 相当于在软电话1000上拨打9999 1000必须注册
originate user/1000 9999  

# 同上
originate user/1000 9999 XML default

```

## 配置SIP网关拨打外部电话
添加一个网关只需要在`conf/sip_profiles/external/`中创建一个XML文件，名字可以随便起，如gw1.xml

```xml

<gateway name="gw1">

    <param name="realm" value="SIP服务器地址，可以是IP或IP:端口号"/>
    <param name="username" value="SIP用户名"/>
    <param name="password" value="密码"/>

</gateway>
```

使用以下命令使其生效

```shell
# 重新加载网关
sofia profile external rescan

# 网关状态
sofia status

```

**如果显示gateway gw1的状态是REGED，则表明已正确地注册到了网关上**

```shell
# 通过网关呼叫 18812345678 
originate sofia/gateway/gw1/18812345678 &echo
```

被叫号码接听电话后，FreeSWITCH会执行echo程序，也就是将你的声音返回给你自己


## 从某一分机上呼出
一般情况选择一个拨号开头就可以通过分机直接呼出到18812345678 手机上   

修改拨号计划，创建一个新的XML文件——`conf/dialplan/default/call_out.xml`

```xml

<include>

  <extension name="call out">
    <condition field="destination_number" expression="^0(\d+)$">
      <action application="bridge" data="sofia/gateway/gw1/$1"/>
    </condition>
  </extension>

</include>
```

`^0(\d+)$`为正则表达式，`(\d+)`匹配0后面的所有数字并存到变量`$1`中。然后通过bridge程序通过网关gw1打出该号码。当然，建立该XML后需要在控制台中执行reloadxml使之生效     
也就是说，若通过内部分机呼叫 `188123435678` 那么分机需要拨号 `018812345678`


# 启动和运行

## 命令解析
执行 `freeswitch -help` 可以看到如下帮助   

```
Usage: freeswitch [OPTIONS]  # FreeSWITCH的启动命令及参数

These are the optional arguments you can pass to freeswitch:  # 下面是可选的命令行参数列表
        -nf                    -- 不fork进程
        -reincarnate           -- 在不受控制的退出时重启switch
        -reincarnate-reexec    -- 在重启时运行execv（有助于升级）
        -u [user]              -- 指定要切换到的用户
        -g [group]             -- 指定要切换到的组
        -core                  -- 转储核心文件
        -help                  -- 显示此帮助信息
        -version               -- 打印版本并退出
        -rp                    -- 启用高优先级设置
        -lp                    -- 启用低优先级设置
        -np                    -- 启用正常优先级设置
        -vg                    -- 在valgrind下运行
        -nosql                 -- 禁用内部sql scoreboard
        -heavy-timer           -- Heavy Timer，可能更精确但代价更高
        -nonat                 -- 禁用自动nat检测
        -nonatmap              -- 禁用自动nat端口映射
        -nocal                 -- 禁用时钟校准
        -nort                  -- 禁用时钟clock_realtime
        -stop                  -- 停止freeswitch
        -nc                    -- 不要输出到控制台并在后台运行
        -ncwait                -- 不要输出到控制台并在后台运行，但等待系统就绪后再退出（意味着-nc）
        -c                     -- 输出到控制台并保持前台运行

        Options to control locations of files:  # 控制文件位置的选项
        -base [basedir]         -- 替代前缀目录
        -cfgname [filename]     -- FreeSWITCH主配置文件的替代文件名
        -conf [confdir]         -- FreeSWITCH配置文件的替代目录
        -log [logdir]           -- 日志文件的替代目录
        -run [rundir]           -- 运行时文件的替代目录
        -db [dbdir]             -- 内部数据库的替代目录
        -mod [moddir]           -- 模块的替代目录
        -htdocs [htdocsdir]     -- htdocs的替代目录
        -scripts [scriptsdir]   -- 脚本的替代目录
        -temp [directory]       -- 临时文件的替代目录
        -grammar [directory]    -- 语法文件的替代目录
        -certs [directory]      -- 证书的替代目录
        -recordings [directory] -- 录音的替代目录
        -storage [directory]    -- 语音信箱存储的替代目录
        -cache [directory]      -- 缓存文件的替代目录
        -sounds [directory]     -- 声音文件的替代目录

```

## 日志 
一般使用命令 `freeswitch -nc`进行后台启动freeswitch ，然后通过查看log/freeswitch.log跟踪系统运行情况，或者通过 `fs_cli`命令客户端连接freeswitch console



## 命令客户端
`fs_cli` 是一个类似Telnet的客户端（也类似于Asterisk中的“asterisk-r”命令），它使用FreeSWITCH的`ESL`（Event Socket Library）协议与FreeSWITCH通信。当然，使用该协议需要加载模块`mod_event_socket`，该模块是默认加载的

```shell
freeswitch@riiicc> status  # 输入status命令查看FreeSWITCH状态
UP 0 years, 0 days, 1 hour, 58 minutes, 25 seconds, 560 milliseconds, 182 microseconds  # FreeSWITCH已经运行了1小时58分钟25秒560毫秒182微秒。
FreeSWITCH (Version 1.6.20  64bit) is not ready  # FreeSWITCH当前尚未准备好，可能正在启动或出现了某些错误。
7 session(s) since startup  # 自启动以来，已经处理了7个会话（session）。
0 session(s) - peak 3, last 5min 0  # 过去5分钟内，最高同时有3个会话，当前没有会话。
0 session(s) per Sec out of max 30, peak 2, last 5min 0  # 过去5分钟内，每秒最多处理30个会话，最高同时处理2个会话，当前没有会话。
1000 session(s) max  # 最大允许同时处理1000个会话。绑定 F
min idle cpu 0.00/99.33  # 空闲CPU占用率为0%，最近5分钟的平均CPU占用率为99.33%。
Current Stack Size/Max 240K/8192K  # 当前堆栈大小为240K，最大可用堆栈大小为8192K。

```

为了调试方便，FreeSWITCH还在`conf/autoload_configs/switch.conf.xml`中定义了一些控制台快捷键。可以通过F1~F12这几个按键来使用它    
不过，在某些操作系统上，有些快捷键可能与操作系统相冲突，这时你就只能直接输入这些命令或重新定义  

```xml
<configuration name="switch.conf" description="Core Configuration">

<cli-keybindings>  <!-- 定义CLI键绑定 -->
    <key name="1" value="help"/>  <!-- 绑定 F1到help命令 -->
    <key name="2" value="status"/>  <!-- 绑定 F2到status命令 -->
    <key name="3" value="show channels"/>  <!-- 绑定 F3到show channels命令 -->
    <key name="4" value="show calls"/>  <!-- 绑定 F4到show calls命令 -->
    <key name="5" value="sofia status"/>  <!-- 绑定 F5到sofia status命令 -->
    <key name="6" value="reloadxml"/>  <!-- 绑定 F6到reloadxml命令 -->
    <key name="7" value="console loglevel 0"/>  <!-- 绑定 F7到设置控制台日志级别为0的命令 -->
    <key name="8" value="console loglevel 7"/>  <!-- 绑定 F8到设置控制台日志级别为7的命令 -->
    <key name="9" value="sofia status profile internal"/>  <!-- 绑定 F9到显示内部SIP配置文件状态的命令 -->
    <key name="10" value="sofia profile internal siptrace on"/>  <!-- 绑定 F10到打开内部SIP配置文件的SIP跟踪功能的命令 -->
    <key name="11" value="sofia profile internal siptrace off"/>  <!-- 绑定 F11到关闭内部SIP配置文件的SIP跟踪功能的命令 -->
    <key name="12" value="version"/>  <!-- 绑定 F12到显示FreeSWITCH版本信息的命令 -->
</cli-keybindings>

    ...
</configuration>
```

> 由于测试环境是`FreeSWITCH Version 1.6.20~64bit ( 64bit)` 在测试时，`F10`并不对应上面写的命令，而是执行了`fsctl pause`暂停FreeSWITCH的运行（不能处理新的呼叫，注册，消息和事件），需要执行命令 `fsctl resume` 来解除暂停,`F11`由于按键冲突未知，猜测可能是`fsctl resume`

常见错误 

```
# 说明FreeSWITCH没有启动或mod_event_socket没有正确加载，请检查TCP的8021端口是否处于监听状态或被其他进程占用

[ERROR] libs/esl/fs_cli.c:652 main() Error Connecting [Socket Connection Error]

```

**`fs_cli`也支持很多命令行参数，值得一提的是`-x`参数，它允许执行一条命令后退出，这在编写脚本程序时非常有用**   

- `fs_cli -x "version"` 查看版本  
- `fs_cli -x "status"` 查看状态
- `fs_cli -x "originate user/1000 &bridge(user/1001)"` 拨号转发

## 呼叫  
在FreeSWITCH中使用`originate`命令发起一次呼叫   

```shell

originate user/1000 &echo

```

`user/1000`称为呼叫字符串`Dial String`，有时也叫`Call URL`。`user`是一种特殊的呼叫字符串，在后面我们还会看到其他的呼叫字符串


## API 和 App

`originate`是FreeSWITCH内部的一个命令（Command），它用于控制FreeSWITCH发起一个呼叫。FreeSWITCH的命令不仅可以在控制台上使用，也可以在各种嵌入式脚本、Event Socket（fs_cli就是使用了ESL库）或HTTP RPC上使用，所有命令都遵循一个抽象的接口，因而这些命令又称`API Commands` 

- `echo`则是一个常用的应用程序（Application，App），它的作用是控制一个Channel的一端。我们知道，一个Channel有两端，在上面的例子中，alice是一端，另一端就是echo。电话接通后相当于alice在跟echo通话   
- 另一个常用的App是`park` `originate user/alice &park` park 就是挂起的意思，呼叫alice并挂起，此时alice接通也听不到任何声音，park的用户体验并不好，因为alice不知道要等多长时间才有人接电话，由于它听不到任何声音，这时它就会奇怪电话到底有没有接通    
- `originate user/alice &hold` hold能在等待的同时播放保持音乐（Music on Hold，MOH）
- `originate user/alice &playback(/root/welcome.wav)` playback可以直接播放一个特定的声音文件
- `originate user/alice &record(/tmp/voice_of_alice.wav)` record 直接录音

以上的例子实际上都只是建立一个`Channel`，相当于`FreeSWITCH`作为一个UA跟alice通话。它是个一条腿（只有a-leg）的通话。在大多数情况下，FreeSWITCH都是作为一个B2BUA来桥接两个UA进行通话的。在alice接听电话以后，bridge程序可以再启动一个UA呼叫bob    
`originate user/alice &bridge(user/bob)`

也可以用另一种方式建立他们之间的通话

```
# 将 alice 挂起
originate user/alice &park

# 将 bob 挂起
originate user/bob &park

# 找到二者channel
show channels

# 连接alice bob
uuid_bridge <alice_uuid> <bob_uuid>

```

### 区别和联系

> 一个App是一个程序（Application），它作为一个Channel一端与另一端的UA进行通信，相当于它工作在Channel内部；   
> 而一个API则是独立于一个Channel之外的，它只能通过找到Channel的UUID来控制一个Channel（如果需要的话），相当于一个第三者。 这就是API与App最本质的区别  

> 通常我们在控制台上输入的命令都是API；   
> 而在dialplan中执行的程序都是App（dialplan中也能执行一些特殊的API）

> 大部分公用的API都是在`mod_commands`模块中加载的；   
> 而App则在`mod_dptools`中，因而App又称为拨号计划工具（Dialplan Tools）

某些App有与其对应的API，如上述的`bridge`和`uuid_bridge`，还有`transfer`和`uuid_transfer`、`playback`和`uuid_playback`等。    

uuid一族的API都是在一个`Channel`之外对`Channel`进行控制的，它们应用于不能参与到通话中却又想对正在通话的Channel做点什么的场景中,例如alice和bob正在畅聊，有个“坏蛋”使用`uuid_kill`将电话切断，或使用`uuid_broadcast`对他们广播恶作剧音频，或者使用`uuid_record`对他们谈话的内容录音等


# FreeSWITCH架构 
FreeSWITCH由一个稳定的核心（Core）及一些外围模块组成。这些外围的模块根据其功能和用途的不同又分为Endpoint、Codec、Dialplan、Application等不同的类别

FreeSWITCH内部使用线程模型来处理并发请求，每个连接都在单独的线程中进行处理，不同的线程间通过Mutex互斥访问共享资源，并通过消息和异步事件等方式进行通信。这种架构能处理很高的并发，并且在多核环境中运算能均匀地分布到多颗CPU或单CPU的多个核心上

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/freeswitch架构说明.png)


## 数据库  
FreeSWITCH支持多种流行的关系型数据库。为了尽量减少对其他系统的依赖，FreeSWITCH默认使用的数据库类型是SQLite。SQLite是一种嵌入式数据库，FreeSWITCH可以直接调用它提供的库函数来访问数据。由于`SQLite`会进行读锁定，因此在使用SQLite时不建议通过外部应用直接读取核心数据库   

除`SQLite`外，系统也支持通过使用`ODBC`方式连接其他数据库，如`PostgreSQL`、`MySQL`等


## 接口 Interface
FreeSWITCH核心层定义了以下接口,来自`switch_types.h`

```c
typedef enum {

    SWITCH_ENDPOINT_INTERFACE,        // 终点

    SWITCH_TIMER_INTERFACE,           // 定时器

    SWITCH_DIALPLAN_INTERFACE,        // 拨号计划

    SWITCH_CODEC_INTERFACE,           // 编解码

    SWITCH_APPLICATION_INTERFACE,     // 应用程序

    SWITCH_API_INTERFACE,             // 命令

    SWITCH_FILE_INTERFACE,            // 文件

    SWITCH_SPEECH_INTERFACE,          // 语音合成

    SWITCH_DIRECTORY_INTERFACE,       // 用户目录

    SWITCH_CHAT_INTERFACE,            // 聊天计划

    SWITCH_SAY_INTERFACE,             // 分词短语

    SWITCH_ASR_INTERFACE,             // 语音识别

    SWITCH_MANAGEMENT_INTERFACE,      // 网管接口

    SWITCH_LIMIT_INTERFACE,           // 资源限制接口

    SWITCH_CHAT_APPLICATION_INTERFACE // 聊天应用程序接口

} switch_module_interface_name_t;

```

## 接口实现
- `终点Endpoint` 主要包含了不同呼叫控制协议的接口，如SIP、TDM硬件、H323以及Google Talk等
- `拨号计划Diaplan` Dialplan主要提供查找电话路由功能。系统默认的Dialplan由mod_dialplan_xml提供
- `聊天计划 Chatplan` 类似于Dialplan，不同的是Chatplan主要对文本消息进行路由
- `应用程序 Application` FreeSWITCH提供了许多App使复杂的任务变得异常简单，如mod_voicemail模块可以很简单地实现语音留言；而mod_conference模块则可以实现高质量的多方会议
- `命令接口FSAPI` 简称API，它是一种对外的命令接口
- `XMl接口 XML Interface`,XML接口支持多种获取XML的方式，它可以从本地的配置文件或数据库中读取，甚至**可以从一个能动态返回XML的远程HTTP服务器中读取**
- `编解码器 Codec`,FreeSWITCH支持最广泛的Codec，除了大多数VoIP系统支持的G711、G722、G729、GSM外，它还支持iLBC、BV16/32、SILK、iSAC、CELT、OPUS等
- `语音识别及语音合成 ASR/TTS` 支持语音自动识别（ASR）及文本/语音转换（TTS）
- `格式、文件接口 Format，File Interface` ,支持不同格式的声音文件回放、录音，如WAV、MP3等。mod_sndfile模块通过libsndfile库提供了对大部分音频文件格式的支持。MP3格式是在mod_shout中实现的 
- `日志 Logger`,日志可以写到控制台、日志文件、系统日志（syslog）以及远程的日志服务器。实现日志功能的模块有mod_console、mod_logfile、mod_syslog等
- `定时器 Timer`,实时的话音通话需要非常准确的定时器。在FreeSWITCH中，可以使用软时钟（soft timer）或内核提供的时钟来定时（如Linux中的timerfd或posix timer）
- `嵌入式语言 Embeded Language`,通过swig可支持多种嵌入式语言进而控制呼叫流程，如Lua、Javascript、Perl等
- `事件套接字 Event Socket`，通过Event Socket可以使用任何其他语言（只要支持Socket），通过TCP Socket可控制呼叫流程、扩展FreeSWITCH的功能




## 事件（Event）

## 目录结构

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/freeswitch目录结构.png)


## 配置文件
配置文件由许多XML文件组成。在系统装载时，XML解析器会将所有XML文件组织在一起，并读入内存，组成一个大的XML文档（Document），称为XML注册表。XML文档本身非常适合描述复杂的数据结构，在FreeSWITCH中可以非常灵活地使用这些数据。这种设计的好处是可以帮助实现非常高的可扩展性

### freeswitch.xml

freeswitch.xml是所有XML文件的黏合剂,配置了其他 xml的位置信息   
在这个文件中可以看到很多如下的xml代码  

```xml
  <section name="dialplan" description="Regex/XML Dialplan">
    <X-PRE-PROCESS cmd="include" data="dialplan/*.xml"/>
  </section>
```

`X-PRE-PROCESS`预处理指令，作用是将data参数指定的文件内容包含（include）到当前文件中来  

`X-PRE-PROCESS`是一个预处理指令，FreeSWITCH在加载阶段只对其进行简单替换，**并不进行语法分析，因此对它进行注释是没有效果的**   
如果不需要某条 `X-PRE-PROCESS`预处理指令 需要将其标签破环即可 也就是将`X-PRE-PROCESS`标签改成诸如 `aaa-X-PRE-PROCESS`即可    

### vars.xml 
vars.xml主要通过`X-PRE-PROCESS`指令定义了一些全局变量   

```xml
  <X-PRE-PROCESS cmd="set" data="domain=$${local_ip_v4}"/>
  <X-PRE-PROCESS cmd="set" data="domain_name=$${domain}"/>
  <X-PRE-PROCESS cmd="set" data="hold_music=local_stream://moh"/>

```

使用`X-PRE-PROCESS`设置的变量都称为**全局变量**，它们在FreeSWITCH运行期间永远都是有效的。**全局变量**以`$${var}`表示
而后面可能还会遇到**局部变量**，它们通常在拨号计划中，在一个呼叫的生命周期中才有效。局部变量以`${var}`表示

在加载vars.xml之前，FreeSWITCH就已经“算”出并设置了一些全局变量，也就是说有些变量是系统在运行时自动设置的，其有默认的值    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/freeswitch全局变量.png)


实际使用中，可以使用`global_getvar`或这个API命令来查看这些变量的值  

```shell
freeswitch@riiicc> global_getvar sound_prefix
/usr/local/freeswitch/sounds/en/us/callie

freeswitch@riiicc> global_getvar local_ip_v4
192.168.0.104
```

由于这些变量是在vars.xml加载前设置的，因而可以在varx.xml中覆盖它们

```xml
<X-PRE-PROCESS cmd="set" data="local_ip_v4=192.168.0.106"/>
```


### autoload_configs目录
autoload_configs目录下的各种配置文件会在系统启动时装入。一般来说都是模块级的配置文件，每个模块对应一个（注意，并不是所有的模块都有配置文件）。文件名一般以“模块名.conf.xml”的方式命名（模块名中不包含“mod_”，如sofia.conf.xml）

`sofia.conf.xml`的文件名并不重要，完全可以改成其他的名字，只要扩展名是.xml就可以正常被`freeswitch.xml`中的`X-PRE-PROCESS`预处理指令装入。这里重要的是，这个配置文件中的`configuration`标签的name属性，`mod_sofia`在启动时会向XML注册表中查找name为`sofia.conf`的configuration，进而访问其下面的配置参数。

autoload_configs目录中有一个特殊的`modules.conf.xml`，其决定了FreeSWITCH启动时自动加载哪些模块。如下面的配置片断，如果需要在FreeSWITCH启动时自动加载某个模块，就在这里添加一行，如果不需要，就注释掉或直接删除

```xml
<configuration name="modules.conf" description="Modules">
  <modules>
    <!-- Loggers (I'd load these first) -->
    <load module="mod_console"/>
    <!-- <load module="mod_graylog2"/> -->
    <load module="mod_logfile"/>
    <!-- <load module="mod_syslog"/> -->

    <!--<load module="mod_yaml"/>-->

    <!-- Multi-Faceted -->
    <!-- mod_enum is a dialplan interface, an application interface and an api command interface -->
    <load module="mod_enum"/>
    ...
</configuration>
```

### XML用户目录
XML用户目录决定了哪些用户可以注册到FreeSWITCH上,SIP并不要求一定要注册才可以打电话，但是通话前的用户认证参数仍需要在用户目录中进行配置。   
用户目录的默认配置文件在conf/directory/下，系统自带的配置文件为default.xml

```xml
<include>  <!-- 包含其他XML文件 -->
  <!-- 域名或IP地址（addr中@符号右侧的部分）-->
  <domain name="$${domain}">
    <params>
      <!-- dial-string参数指定呼叫字符串，包括SIP INVITE和Presence信息等 -->
      <param name="dial-string" value="{^^:sip_invite_domain=${dialed_domain}:presence_id=${dialed_user}@${dialed_domain}}${sofia_contact(*/${dialed_user}@${dialed_domain})},${verto_contact(${dialed_user}@${dialed_domain})}"/>
      <!-- 这些参数是Verto正常工作所需的 -->
      <param name="jsonrpc-allowed-methods" value="verto"/>
      <!-- <param name="jsonrpc-allowed-event-channels" value="demo,conference,presence"/> -->
    </params>

    <variables>
      <!-- 设置变量，例如录音是否为立体声 -->
      <variable name="record_stereo" value="true"/>
      <variable name="default_gateway" value="$${default_provider}"/>
      <variable name="default_areacode" value="$${default_areacode}"/>
      <variable name="transfer_fallback_extension" value="operator"/>
    </variables>

    <groups>
      <group name="default">
	<users>
	  <!-- 包含default目录下的所有XML文件 -->
	  <X-PRE-PROCESS cmd="include" data="default/*.xml"/>
	</users>
      </group>

      <group name="sales">
	<users>
	  <!--type="pointer"表示指针，可以在多个组中使用相同的用户-->
	  <user id="1000" type="pointer"/>
	  <user id="1001" type="pointer"/>
	  <user id="1002" type="pointer"/>
	  <user id="1003" type="pointer"/>
	  <user id="1004" type="pointer"/>
	</users>
      </group>
    </groups>
  </domain>
</include>

```


## 呼叫相关概念
如果Bob与Alice通话，典型的呼叫流程主要有以下两种：   
- Bob向FreeSWITCH发起呼叫，FreeSWITCH接着启动另一个UA呼叫Alice，两者通话。   
- FreeSWITCH同时呼叫Bob和Alice，两者接电话后FreeSWITCH将a-leg和b-leg桥接（bridge）到一起，两者通话  

> 第二种流程还有一种变种,有人利用上、下行通话的不对称性卖电话回拨卡获取利润, FreeSWITCH作为服务器，把回拨卡卖给Bob。Bob按回拨卡上的号码呼叫FreeSWITCH，FreeSWITCH不应答，而是在获得Bob的主叫号码后直接挂机。然后FreeSWITCH回拨Bob，Bob接听后FreeSWITCH启动一个IVR程序指示Bob输入Alice的号码。最后FreeSWITCH再呼叫Alice

### 来去话、Session、Channel与Call
Bob到FreeSWITCH的通话称为`来话`注意，在这里我们都是针对FreeSWITCH来说  
FreeSWITCH作为一个B2BUA再去呼叫Alice时，就称为`去话`

> 每一次呼叫，FreeSWITCH都会启动一个Session（会话，它包含SIP会话，SIP会在每对UAC-UAS之间生成一个SIP Session），用于控制整个呼叫，它会一直持续到通话结束。其中，每个Session都控制着一个Channel（通道，又称信道），Channel是一对UA间通信的实体，相当于FreeSWITCH的一条腿（leg），每个Channel都用一个唯一的UUID来标识，称为Channel UUID。另外，Channel上可以绑定一些呼叫参数，称为Channel Variable（通道变量）。Channel中可能包含媒体（音频或视频流），也可能不包含。通话时，FreeSWITCH的作用是将两个Channel（a-leg和b-leg，通常先创建的或占主动的叫a-leg）桥接（bridge）到一起，使双方可以通话。这两路桥接的通话（两条腿）在逻辑上组成一个通话，称为一个Call

### 回铃音与Early Media
在早期，B端所在的交换机b只向A端交换机a传送地址全（ACM）信号，证明呼叫是可以到达B的，A端听到的回铃音的铃流是由A端所在的交换机生成并发送的。但后来，为了在A端能听到B端特殊的回铃音（如“您拨打的电话正在通话中…”或“对方暂时不方便接听您的电话”尤其是现代交换机，支持各种个性化的彩铃），回铃音只能由B端交换机发送。在B接听电话前，回铃音和彩铃是不收费的（不收取主叫A的本次通话费。彩铃费用一般是在被叫端即B端以月租或套餐形式收取的）。这些回铃音就称为Early Media（早期媒体）。在SIP通信中，它是由SIP的183（带有SDP）消息描述的

> 理论上讲，B接听电话后交换机b可以一直不向交换机a发送应答消息，而是将真正的话音数据伪装成Early Media，以实现“免费通话”。但这种应用是有限制的，大多数交换机允许Early Media的时间不会太长，如1分钟，以防止不守规则的人进行免费通话  

### 全局变量与局部变量
使用`X-PRE-PROCESS`在FreeSWITCH中设置一些变量（包括自动生成的变量），在后续使用时可以用`$${var}`的形式来进行引用。这些变量是全局有效的，因而称为全局变量。    
另外一些变量是在`Dialplan`、`Application`或`Directory`中设置的，它们会影响呼叫流程且可以被动态改变。这些变量一般与一个呼叫有关，严格地说是与一个Channel有关，因而又称为Channel Variable，即通道变量。通道变量可以以`${var}`的形式引用。   

全局变量仅在预处理阶段（系统启动时或重新装载-reloadxml时）被求值，一般用于设置一些系统一旦启动就不会轻易改变的量，如`$${domain}`或`$${local_ip_v4}`等。     
而局部变量（即通道变量）仅在Channel的生命周期中有效。所以，两者最大的区别是，`$${var}`只在加载时求值一次，而`${var}`则在每次执行时都求值,如一个新电话进来时


# 拨号计划
拨号计划（Dialplan）是FreeSWITCH中至关重要的一部分。它的主要作用就是对电话进行路由（从这一点上来说，相当于一个路由表），决定和影响通话的流程。说得简明一点，就是当一个用户拨号时，对用户所拨的号码进行分析，进而决定下一步该做什么   

Dialplan是FreeSWITCH中一个抽象的部分，它可以支持多种不同的格式，如Asterisk风格的格式（由mod_dialplan_asterisk提供）、LUA、inline等。在实际使用中，用得最多的还是XML格式

## XML Dialplan
拨号计划的配置文件默认在conf/dialplan目录中，我们在第5章中讲过，它们是在freeswitch.xml中定义的，是由以下预处理指令装入的

`<X-PRE-PROCESS cmd="include" data="dialplan/*.xml"/>`

拨号计划由多个`Context` 组成。每个Context中有多个`Extension`。所以`Context`就是多个`Extension`的逻辑集合，它相当于一个分组。一个`Context`中的`Extension`与其他`Context`中的`Extension`在逻辑上是隔离的   

```xml
<include>
  <context name="default">
    <extension name="laugh break">
      <condition field="destination_number" expression="^9386$">
        <action application="answer"/>
        <action application="sleep" data="1500"/>
        <action application="playback" data="phrase:funny_prompts"/>
        <action application="hangup"/>
      </condition>
    </extension>
  </context>
</include>
```

Extension相当于路由表中的表项，其中，每一个Extension都有一个name属性，name可以是任意合法的字符串，本身对呼叫流程没有任何影响。一个合适的名字，有助于你在查看Log时发现它

在Extension中可以对一些condition（测试条件）进行判断，如果满足测试条件所指定的表达式，则执行对应的Action（动作）

在Dialplan中将下列Extension配置加入到`conf/dialplan/default.xml`中，并作为第一个Extension

```xml
  <extension name="My Echo Test">
  <condition field="destination_number" expression="^echo|1234$">
    <action application="answer" data=""/>
    <action application="echo" data=""/>
  </condition>
</extension>
```

使用软电话呼叫号码1234（或echo，如果你拨的是echo的话）

第一个要执行的动作是answer，它是一个App，用于对来话进行应答（由于这是一个SIP呼叫，因而底层的信令会给主叫发送200 OK消息），然后执行echo（它也是一个App，在第4章我们讲过，这些App大部分来自于mod_dptools），它的作用就相当于一个回音壁，你说什么都反原封不动地反弹回来，所以你能听到自己的声音。

实际上，在这个例子中，我们呼叫1234时创建了一个单腿的呼叫，与其说我们跟FreeSWITCH在通话，还不如说我们在跟FreeSWITCH中的一个App在通话。而XML Dialplan只是帮助我们找到这个（些）App


### 默认的配置文件
系统默认提供的配置文件包含三个Context，分别是default、features和public。它们分别在三个XML文件中

- default是默认的Dialplan，一般来说注册用户都可以通过它来打电话，如拨打其他分机或外部电话等  
- public一般用于接收外来呼叫，因为从外部进来的呼叫是不可信的，所以要进行更严格的控制

> 在default和public中，又通过include预处理指令分别加入了default和include目录中的所有.xml文件。这些目录中的文件仅包含一些额外的Extension。由于Dialplan在处理的时候是一定的顺序进行的，所以一定要注意这些文件的装入顺序。通常这些文件都是按文件名排序的，如00_、01_等等。如果你想添加新的Extension，可以在这些目录里创建文件。但要注意，这些文件的优先级一般比直接写在default.xml或public、xml中要低（取决于include语句的位置）

?> 实际，由于在处理Dialplan时要对每一项进行正则表达式匹配，这是非常影响效率的。所以在生产环境中，往往要删除这些默认的Dialplan，只保留或添加有用的部分

### 通道变量
































# 参考
> - [FreeSWITCH权威指南](https://book.douban.com/subject/25902109/)