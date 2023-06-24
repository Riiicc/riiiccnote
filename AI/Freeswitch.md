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

uuid一族的API都是在一个Channel之外对Channel进行控制的，它们应用于不能参与到通话中却又想对正在通话的Channel做点什么的场景中,例如alice和bob正在畅聊，有个“坏蛋”使用uuid_kill将电话切断，或使用uuid_broadcast对他们广播恶作剧音频，或者使用uuid_record对他们谈话的内容录音等

















