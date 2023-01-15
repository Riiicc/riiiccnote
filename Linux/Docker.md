# Docker

[官网](https://www.docker.com/)

[Docker Hub](https://hub.docker.com/)

特点：
- 一次构建，到处运行
- 快速交付和部署
- 简化升级扩容
- 便捷运维


# 结构说明

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/docker说明.png)

- 镜像`image`
  - 用来创建容器的`docker`模板,可以将项目打包为镜像发布
- 容器`container`
  - 独立运行的一个或一组应用，应用程序或服务运行在容器里面,容器是用镜像创建的运行实例
- 仓库`repo`
  - 集中存放镜像文件的场所`Docker Hub`


# 安装

> Docker依赖于已经存在并运行的Linux内核环境

[安装文档](https://docs.docker.com/engine/install/centos/)

```bash
yum install -y gcc
yum install -y gcc-c++

yum install -y yum-utils


#设置阿里镜像仓库
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新yum软件包索引
yum makecache fast

#安装Docker CE 社区版
sudo yum install -y docker-ce docker-ce-cli containerd.io

#启动
sudo systemctl start docker

# 查看版本
docker version

# 运行示例镜像
docker run hello-world

#查看运行的容器
docker ps 
#查看所有的容器
docker ps -a

#08d1e501beb7 CONTAINER ID（容器id）
docker stop 08d1e501beb7

# 删除容器
docker rm 08d1e501beb7

```

# 卸载

```bash
# 停止
systemctl stop docker

yum remove docker-ce docker-ce-cli containerd.io

rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

# 镜像加速
[阿里云镜像加速](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://eegsvod0.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

# 命令

## 帮助类启动命令

```bash
# 启动
systemctl start docker

# 停止
systemctl stop docker

#重启
systemctl restart docker

#查看状态
systemctl status docker

#开机启动
systemctl enable docker

#查看概要信息
docker info

#帮助命令文档
docker --help
```

## 镜像命令

### 列出镜像
```bash
#列出主机镜像
docker images 

# 所有本地镜像 
docker images -a

#只显示镜像ID
docker images -q

docker images -qa

#查看镜像/容器/数据卷所占空间
docker system df

```

### 搜索镜像

```bash
#搜索镜像
docker search [OPTIONS] [NAME]

#列出redi镜像 默认25个
docker search redis

# 列出前五个镜像
docker search --limit 5 redis
```

### 拉取镜像

```bash
docker pull 镜像名:[TAG]

docker pull redis:latest

#没有TAG默认为最新版
docker pull redis
#指定版本
docker pull redis:6.0.8
```

### 删除镜像

```bash

#删除镜像(使用种无法删除,需要使用 -f)
docker rmi 镜像ID

#强制删除
docker rmi -f 镜像ID

#全部删除
docker rmi -f $(docker images -qa)
```

## 容器命令

### 创建容器

```bash

docker run [OPTION] IMAGE [COMMAND] [ARGS...]

#启动Ubuntu,启动后进入Ubuntu系统
docker run -it ubuntu bash

```

`OPTIONS说明（常用）`：有些是一个减号，有些是两个减号
`--name="容器新名字"` 为容器指定一个名称；  
`-d`: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)；   

`-i`：以交互模式运行容器，通常与 `-t` 同时使用；  
`-t`：为容器重新分配一个伪输入终端，通常与 `-i` 同时使用；也即启动**交互式**容器(前台有伪终端，等待交互)；   

`-P`: 随机端口映射，大写P  
`-p`: 指定端口映射，小写p  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/docker-p说明.png)

```bash
#创建成功后通过命令查看容器
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
c4569fa10898   ubuntu    "bash"    2 minutes ago   Up 2 minutes             focused_hermann
```


```bash
#使用命令 创建一个新的名为 myu1的Ubuntu 容器实例
docker run -it --name=myu1 ubuntu bash

# 再次查看容器列表 变为两个容器
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
8493053d3f58   ubuntu    "bash"    9 seconds ago   Up 8 seconds             myu1
c4569fa10898   ubuntu    "bash"    8 minutes ago   Up 8 minutes             focused_hermann

```

### 查看容器
`docker ps `   
`-a`:列出当前所有正在运行的容器+历史上运行过的
`-l`:显示最近创建的容器。
`-n`：显示最近n个创建的容器。
`-q`:静默模式，只显示容器编号。


### 退出容器

- `exit` 退出时容器,停止运行
- `ctrl + p + q`退出时容器,容器后台继续运行

### 启动/停止/删除 容器

```bash
docker start 容器ID

docker restart 容器ID

dcoker stop 容器ID

#强制停止
docker kill 容器ID/容器名

#删除容器
docker rm 容器ID
#强制删除
docker rm -f 容器ID

```


### 启动守护容器进程

```bash
docker run -d 容器名

docker run -d ubuntu
```

> 执行命令守护启动后发现,容器没有启动:    
> Docker容器后台运行,就必须有一个前台进程.容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的

示例: 
前台启动一个redis容器  

```bash
#启动一个redis 容器
docker run -it redis:6.0.8
```

当我们 `Ctrl +C`  退出容器时,redis容器也会同时关闭  

后台启动一个redis容器

```bash
docker run -d redis:6.0.8 

[root@localhost ~]# docker run -d redis:6.0.8
de886bd96d1c477155a02f8d8878df41e964bb32fc9a08d23aff46311140bd95
```

### 容器日志/进程/信息

```bash
#容器日志
docker logs 容器ID

#容器内运行的进程
docker top 容器ID

#容器内部细节
docker inspect 容器ID

```

### 进入容器进行交互

- `docker exec` (推荐) 在容器种打开新的终端,可以启动新进程,使用exit退出**不会**导致容器停止
- `docker attach` 直接进入容器命令终端,不会启动新进程,使用exit退出**会**导致容器停止

```bash
docker exec -it 容器ID bashShell

#示例
docker exec -it 8493053d3f58 bash

docker attach 容器ID

```

进入redis 服务  
- `docker exec -it 容器ID bash` 进入redis容器
- `docker exec -it 容器ID redis-cli` 进去redis 数据库命令行中  

**上面二者区别演示** 

```bash
[root@localhost ~]# docker exec -it de886bd96d1c bash
root@de886bd96d1c:/data# redis-cli -p 6379
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> exit
root@de886bd96d1c:/data# exit
exit
[root@localhost ~]# docker exec -it de886bd96d1c redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> exit
```

> 一般情况下都是用 `-d` 后台启动程序，再使用`exec`进入对应容器实例


### 容器文件拷贝

容器内文件拷贝到主机上 

```bash
docker cp 容器ID:容器内路径 目的主机路径

docker cp 8493053d3f58:/usr/a.txt /usr/local
```

> 容器内路径默认为容器根路径 目的主机为当前路径  

### 导入导出  

- `import`
- `export`

```bash
# 导出
docker export 容器ID > 文件名.tar

docker export 8493053d3f58 > abc.tar

cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号  
cat abc.tar |docker import - ric/ubuntu:3.7

```

> 导出是将容器导出为一个镜像包   
> 导入是将包导入为镜像,再通过运行镜像来创建一个或多个和之前容器内容相同的容器   


```bash
[root@localhost ~]# cat abc.tar |docker import - ric/ubuntu:3.7
sha256:85cfddaa5fe7ba2731fe2b250ccaf3d64819388c8d6f3750ab8d86088f77da16
[root@localhost ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
ric/ubuntu   3.7       85cfddaa5fe7   20 seconds ago   72.8MB
ubuntu       latest    ba6acccedd29   13 months ago    72.8MB
redis        6.0.8     16ecd2772934   2 years ago      104MB
[root@localhost ~]# docker run -it 85cfddaa5fe7 bash
root@a7804676cbd2:/# ls
a.txt  bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@a7804676cbd2:/#
```


# 镜像

> 是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括**代码、运行时需要的库、环境变量和配置文件**等)，这个打包好的运行环境就是image镜像文件 

**只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)**

## 镜像的分层
我们在`docker pull` 时看到下载分几步,这就是镜像的分层,出现这种情况是Docker 镜像采用了 `UnionFS` 联合文件系统   

`Union文件系统（UnionFS）`是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。   

> bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是引导文件系统bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs

> **举例**:比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享  

> 当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。  
> 所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。只有容器层是可写的，容器层下面的所有镜像层都是只读的  


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/docker镜像说明.png)



## 镜像提交
提交容器副本使其成为新的本地镜像   

```bash
docker commit -m="chuangjian" -a="ric" a7804676cbd2 vimubuntu:1.0  

#根据imageid 再次创建Ubuntu容器  
docker run -it f396b421be95 bas
```



## 本地镜像发布到阿里云

- 进入阿里云容器镜像服务,创建个人实例
- 进入个人实例创建命名空间,并将命名空间设置为公开
- 创建个人仓库进行配置,最后代码源选择本地仓库,创建完成获得脚本  


```bash
# 登陆阿里云Docker Registry
docker login --username=ric_cc registry.cn-hangzhou.aliyuncs.com

#标记镜像
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/ricbeijing/ricrepo:[镜像版本号]
#推送镜像
docker push registry.cn-hangzhou.aliyuncs.com/ricbeijing/ricrepo:[镜像版本号]

#拉取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/ricbeijing/ricrepo:[镜像版本号]
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/docker推送阿里云.png)


## 本地镜像发布到私有仓库

### 搭建私有仓库
`Docker Registry`是官方提供的工具，可以用于构建私有镜像仓库,类似于`nexus`


```bash
# -d后台运行   -p端口映射 宿主机:容器  后面使用了-容器卷映射，方便于宿主机联调 
# 容器卷参看下面内容
docker run -d -p 5000:5000  -v /zzyyuse/myregistry/:/tmp/registry --privileged=true registry

```


### 发布到私有仓库

发布前要通过配置取消本机认证要求,编辑文件`/etc/docker/daemon.json`

```json
{
  "registry-mirrors": ["https://eegsvod0.mirror.aliyuncs.com"],
"insecure-registries":["192.168.0.103:5000"]
}
```

> 修改完成后,重启docker,并且启动registry   

```bash

docker tag ce6679015b7d 192.168.0.103:5000/ricifcfgubuntu:2.0

docker push 192.168.0.103:5000/ricifcfgubuntu:2.0
```

发布成功后通过 `GET` 访问 `http://192.168.0.103:5000/v2/_catalog`即可看到镜像名称

```json
{
  "repositories": [
    "ricifcfgubuntu"
  ]
}
```

通过拉取命令 `docker pull 192.168.0.103:5000/ricifcfgubuntu:2.0` 即可拉去镜像  


# 容器数据卷

Docker挂载主机目录访问如果出现`cannot open directory .: Permission denied`   
解决办法：在挂载目录后多加一个`--privileged=true`参数即可

> 在SELinux里面挂载目录被禁止掉了额，如果要开启，我们一般使用`--privileged=true`命令，扩大容器的权限解决挂载目录没有权限的问题   
> 使用该参数，container内的root拥有真正的root权限，否则，container内的root只是外部的一个普通用户权限

## 创建容器卷映射

命令`docker run -d -p 5000:5000  -v /zzyyuse/myregistry/:/tmp/registry --privileged=true registry` 详解

`-v` 冒号左边是宿主机路径,右边是容器内路径,`-v`可以有多组 `-v *** -v ***`

> 卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但**不属于联合文件系统**，因此能够**绕过**`Union File System`提供一些用于持续存储或共享数据的特性：    
> 卷的设计目的就是**数据的持久化**，完全**独立**于容器的生存周期，因此**Docker不会在容器删除时删除其挂载的数据卷**


运行一个带有容器卷存储功能的容器实例:    

```bash
docker run -it --privileged=true -v /宿主机目录:/容器内目录  镜像名/ImageID
```

**重点:数据共享,实时同步**


## 映射的读写规则

映射分为 `读写(默认)` 和 `只读`  

- 读写:`docker run -it --privileged=true -v /宿主机目录:/容器内目录:rw  镜像名/ImageID`
- 只读:`docker run -it --privileged=true -v /宿主机目录:/容器内目录:ro  镜像名/ImageID`
  - 容器内部实例受限(而非宿主机受限)

## 卷的继承和共享
1. 通过命令创建容器`u1` `docker run -it  --privileged=true -v /mydocker/u:/tmp --name u1 ubuntu`   
2. 在`u1` 映射文件夹中创建`u1data.txt` 
3. 创建容器`u2` 并继承`u1`卷  `docker run -it --privileged=true --volumes-from u1 --name u2 ubuntu`
4. 我们进入`u2`可以发现其`/temp`文件夹下存在`u1data.txt`  
5. 在`u2`的`/temp`文件夹下创建 `u2data.txt`,此时可以在宿主机,u1 容器中发现 `u2data.txt`
6. 即使停掉`u1`容器,`u2`和宿主机之间的目录共享仍然生效 
7. 重启`u1`后,共享文件夹会获取所有的最新共享文件  

> 继承和共享后三者 `宿主`,`父容器`,`子容器`是完全共享,完全互通的   



# Docker 的常规软件安装

## Tomcat
- Docker Hub 搜索tomcat
- `docker pull tomcat`
- `docker run -it -p 8080:8080 tomcat`

- `docker pull billygoo/tomcat8-jdk8`
- `docker run -d -p 8080:8080 --name mytomcat8 billygoo/tomcat8-jdk8`

## Mysql
- `docker pull mysql:5.7`
- `docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag`
  - `docker run -p 3306:3306 --name mymysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7`

> Docker 安装MySQL问题:     
> 默认字符集是 `latin1` 需要进行修改,否则数据库无法写入中文    
> MySQL数据需要进行数据卷映射,防止误删无法恢复数据   

**实战安装MySQL**

```bash
docker run -d -p 3306:3306 --privileged=true -v /zzyyuse/mysql/log:/var/log/mysql -v /zzyyuse/mysql/data:/var/lib/mysql -v /zzyyuse/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456  --name mysql mysql:5.7

#解析
docker run -d -p 3306:3306 --privileged=true 
-v /zzyyuse/mysql/log:/var/log/mysql 
-v /zzyyuse/mysql/data:/var/lib/mysql 
-v /zzyyuse/mysql/conf:/etc/mysql/conf.d 
-e MYSQL_ROOT_PASSWORD=123456  --name mysql mysql:5.7
```

> 需要对三个文件(夹)进行映射.然后对conf中的`my.cnf`进行`编辑/创建`来修改编码方式,同时还能防止误删容器造成的数据丢失   
> 修改完成后需要重启容器实例   

**完成数据卷映射后即使删除对应容器,只需要再重启一个相同数据卷映射的容器即可恢复数据库**

```conf
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8

```

```bash
mysql> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```


## Redis

- `docker pull redis:6.0.8`
- 上传修改完成的`redis.conf`包括其中密码等配置,按需求进行修改

**命令** 

```bash
docker run  -p 6379:6379 --name myr3 --privileged=true -v /app/redis/redis.conf:/etc/redis/redis.conf -v /app/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf

# 解析
docker run  -p 6379:6379 --name myr3 --privileged=true 
-v /app/redis/redis.conf:/etc/redis/redis.conf 
-v /app/redis/data:/data 
-d redis:6.0.8 redis-server /etc/redis/redis.conf

# 最后一行 指定 redis-server 的配置文件 

# 连接验证
docker exec -it myr3 redis-cli
```

同样,可以对宿主机的redis.conf 配置文件进行修改,并重启,来改变redis容器的


## MySQL集群
1. 创建MySQL master 容器 

```bash
docker run -p 3307:3306 --name mysql-master -v /mydata/mysql-master/log:/var/log/mysql -v /mydata/mysql-master/data:/var/lib/mysql -v /mydata/mysql-master/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```

2. 创建/修改 `my.cnf` 配置文件 

```conf
[mysqld]

## 设置server_id，同一局域网中需要唯一
server_id=101 

## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  

## 开启二进制日志功能
log-bin=mall-mysql-bin  

## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  

## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  

## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

3. 修改完成配置后重启master实例 `docker restart mysql-master`
4. 进入master容器并在实例内创建数据同步用户

```bash

docker exec -it mysql-master bash

mysql  -uroot -proot

mysql> CREATE USER 'slave'@'%' identified by '123456';

mysql> grant replication slave,replication client on *.* to 'slave'@'%';
```

5.创建从服务器容器实例3308  

```bash
docker run -p 3308:3306 --name mysql-slave -v /mydata/mysql-slave/log:/var/log/mysql -v /mydata/mysql-slave/data:/var/lib/mysql -v /mydata/mysql-slave/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=root  -d mysql:5.7
```

5. 进入`/mydata/mysql-slave/conf`目录下新建`my.cnf` 

```conf
[mysqld]

## 设置server_id，同一局域网中需要唯一
server_id=102

## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  

## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin  

## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  

## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  

## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062  

## relay_log配置中继日志
relay_log=mall-mysql-relay-bin  

## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1  

## slave设置为只读（具有super权限的用户除外）
read_only=1
```

6. 重启myslave实例 `docker restart mysql-slave`
7. 在`master`中查看主从同步状态`show master status`

```bash
mysql> show master status;
+-----------------------+----------+--------------+------------------+-------------------+
| File                  | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------------+----------+--------------+------------------+-------------------+
| mall-mysql-bin.000001 |      617 |              | mysql            |                   |
+-----------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

8. 进入slave 容器配置主从复制 

- `master_host`：主数据库的IP地址；
- `master_port`：主数据库的运行端口；
- `master_user`：在主数据库创建的用于同步数据的用户账号；
- `master_password`：在主数据库创建的用于同步数据的用户密码；
- `master_log_file`:指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
- `master_log_pos`：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
- `master_connect_retry`：连接失败重试的时间间隔，单位为秒。

```bash
change master to master_host='宿主机ip', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;


change master to master_host='192.168.0.103', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;

```

9. 在从数据库中查看主从同步的状态 `show slave stauts \G;`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockermysql主从同步.png)

10. 在从数据库中开启同步 `start slave;` 继续`show slave stauts \G;`查看是否成功

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockermysql主从同步1.png)

11. 此时在主机创建数据库和表,即可在从库中查询到  


## redis 分布式存储案例 

### 分区方案说明 

- 哈希取余分区

> 用户每次读写操作都是根据公式：`hash(key) % N`个机器台数   
> 优点: 简单粗暴，直接有效，只需要预估好数据规划好节点，例如3台、8台、10台，就能保证一段时间的数据支撑。使用Hash算法让固定的一部分请求落到同一台服务器上    
> 缺点: 扩容或者缩容的情况下会导致数据变动,映射关系需要重新计算;机器宕机也会影响计算关系 

- 一致性哈希算法分区  

一致性哈希算法可以解决分布式缓存数据变动和映射问题    

1. 算法构建一致性哈希环
2. 服务器IP节点映射
3. key落到服务器的落键规则  

- 哈希槽分区  

哈希槽实质就是一个数组，数组[0,2^14 -1]形成hash slot空间。


### Redis 集群配置 
**3主3从的Redis集群配置**

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockerredis创建命令解析.png)

```bash
# 连续创建6个名字 redis-node-1 到redis-node-6
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockerredis1.png)

进入容器`redis-node-1`构建集群  

```bash
redis-cli --cluster create 192.168.0.103:6381 192.168.0.103:6382 192.168.0.103:6383 192.168.0.103:6384 192.168.0.103:6385 192.168.0.103:6386 --cluster-replicas 1
```

> `--cluster-replicas 1` 表示为每个master创建一个slave节点,一主一从


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockerredis创建集群分组.png)

> 分配的三个哈希槽 `0-5460` `5461-10922` `10923-16383`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis槽分区.png)


以6381作为切入点,查看节点状态 `cluster info` `cluster nodes`  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/clusterinfo信息查看集群信息.png)


### 主从容错切换迁移案例

我们在6381redis服务中执行

```bash
127.0.0.1:6381> keys *
(empty array)
127.0.0.1:6381> set k1 v1
(error) MOVED 12706 192.168.0.103:6383
127.0.0.1:6381> 
```

发现赋值操作有`(error) MOVED 12706 192.168.0.103:6383`提示,`set k1 v1` 的操作通过哈希计算槽位不在`6381Redis`所在的槽位范围,无法存入键值    
此时我们需要使用`redis-cli -p 6381 -c` 重新连接redis命令行,参数`-c`，优化路由 

```bash
root@localhost:/data# redis-cli -p 6381 -c
127.0.0.1:6381> set k1 v1
-> Redirected to slot [12706] located at 192.168.0.103:6383
OK
192.168.0.103:6383> set k2 v2
-> Redirected to slot [449] located at 192.168.0.103:6381
OK
192.168.0.103:6381>
```

优化路由后,若存入键值不在当前槽范围内会自动重定向到目标槽位所属的redis服务   


**集群检查**:`redis-cli --cluster check 192.168.0.103:6381`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis集群信息查看.png)


> 当其中一个`master`节点宕机后，其对应的`slave`节点自动升级为`master`节点,若再启动`master`,那么原来的`master`自动变为`slave`节点.



### 主从扩容
创建两个redis容器命名为7,8

```bash
docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6387
docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6388
```

进入容器7内部,将其作为master节点加入集群   
`docker exec -it redis-node-7 bash`    
`redis-cli --cluster add-node 192.168.0.103:6387 192.168.0.103:6381`   

> 将新增的6387作为master节点加入集群  
> redis-cli --cluster add-node 自己实际IP地址:6387 自己实际IP地址:6381    
> 6387 就是将要作为master新增节点     
> 6381 就是原来集群节点里面的领路人，相当于6387拜拜6381的码头从而找到组织加入集群   

再使用命令`redis-cli --cluster check 192.168.0.103:6386` 查看集群信息

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis集群主从扩容.png)


重新分配槽号,即重新哈希,分配哈希槽   
`redis-cli --cluster reshard 192.168.0.103:6386`  

重新分配了槽位是 原来的三个 槽位组,每个组截取一部分分给新的节点 ,所以新节点的槽位是不连续的三个槽位段    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis槽位的重新分配.png)

为新的master节点分配slave节点  


命令：`redis-cli --cluster add-node ip:新slave端口 ip:新master端口 --cluster-slave --cluster-master-id 新主机节点ID`
 
`redis-cli --cluster add-node 192.168.0.103:6388 192.168.0.103:6387 --cluster-slave --cluster-master-id 132e7d6cd953fe602d8f529fbab0a31ce06b4b01` 

最后的id是master 的id   

### 主从缩容

首先删除从机slave 

命令：`redis-cli --cluster del-node ip:从机端口 从机6388节点ID`   

`redis-cli --cluster del-node 192.168.0.103:6388 4aba367adc5310effb6edef685338877d3b53e42`

将master 6387槽号清空,统一分配给6386 master节点   

`redis-cli --cluster reshard 192.168.0.103:6386`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis集群缩容操作.png)

操作完成后 原来6387 的槽位全部进入到6386 中

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redis集群缩容操作-缩容结果.png)

命令删除6387 `redis-cli --cluster del-node 192.168.0.103:6387 132e7d6cd953fe602d8f529fbab0a31ce06b4b01`

完成!

## Nginx  
拉取容器镜像，创建挂载目录

```bash
docker search nginx

docker pull nginx

# 创建挂载目录
mkdir -p /home/nginx/conf
mkdir -p /home/nginx/log
mkdir -p /home/nginx/html
```

先生成一个nginx容器，方便复制配置文件进行后续文件映射，后面再删除这个默认容器

```bash
# 生成容器
docker run --name nginx -p 9001:80 -d nginx

# 复制配置文件
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /home/nginx/conf/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /home/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /home/nginx/


# 直接执行docker rm nginx或者以容器id方式关闭容器
# 找到nginx对应的容器id
docker ps -a
# 关闭该容器
docker stop nginx
# 删除该容器
docker rm nginx
 
# 删除正在运行的nginx容器
docker rm -f nginx

```

最终运行命令 注意端口映射，若有多个端口映射需要多个`-p`参数

```bash
docker run \
-p 9002:80 \
--name nginx \
-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/html:/usr/share/nginx/html \
-d nginx:latest

docker run -p 8083:80 --name nginx8083 -v /home/nginx8083/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx8083/conf/conf.d:/etc/nginx/conf.d -v /home/nginx8083/log:/var/log/nginx -v /home/nginx8083/html:/usr/share/nginx/html -d nginx:latest
```

多端口案例 

```bash
docker run -p 9002:80 -p 9003:3306 --name nginx -v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx/conf/conf.d:/etc/nginx/conf.d -v /home/nginx/log:/var/log/nginx -v /home/nginx/html:/usr/share/nginx/html -d nginx:latest
```

```bash
# 重启容器
docker restart nginx  

# 配置更新
docker exec nginx8081 nginx -s reload
#配置测试
docker exec nginx8081 nginx -t
```

# DockerFile 

`DockerFile` 是用来构建Docker镜像的文本文件,是由一条条构建镜像所需的指令和参数构成的脚本  

## 构建步骤 
- 编写`DockerFile`文件
- `docker build`命令构建镜像
- `docker run` 依据镜像构建实例


## DockerFile 基础
- 每条`保留字指令`都**必须为大写字母**,并且**后面至少跟随一个参数**
- 指令从上到下顺序执行
- 每条指令都会创建一个新的`镜像层`并对镜像进行提交

## 执行流程
- docker 从基础镜像运行一个容器
- 执行一条指令对容器做出修改
- 执行类似`docker commit` 的操作提交一个镜像层
- docker 再基于刚提交的镜像运行一个新容器
- 执行dockerfile中的下一条指令,直至所有指令都执行完成

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockerfile说明.png)


## 参数说明 
- `FROM` 基础镜像,当前镜像是基于那个镜像,指定一个已经存在的镜像作为模板
- `MAINTAINER` 镜像维护者的姓名和邮箱地址
- `RUN`容器构建时需要运行的命令
  - shell格式`RUN <命令行命令>` 例 `RUN yum -y install vim`
  - exec格式 `RUN ["可执行文件","参数1","参数2"]`
  - `RUN ["./test.php","dev","offline"]` 等价于`./test.php dev offline`
  - `RUN`是在docker构建时运行
- `EXPOSE` 当前容器对外暴露的端口
- `WORKDIR` 指定在创建容器后,终端默认登录进来的目录
- `USER`指定镜像以那个用户去执行,默认root
- `ENV` 构建镜像过程中设置环境变量
  - 例:`ENV MY_PATH /usr/mytest` 
  - 后续指令即可使用 该环境变量 `WORKDIR $MY_PATH` 默认登录地址设置
- `ADD`将宿主机目录下的文件拷贝进镜像,且会自动处理URL和解压tar包
- `COPY` 类似于`ADD` `COPY["src","dest"]`
- `VOLUME` 容器数据卷,用于数据保存 
- `CMD` 指定容器启动后要做的事,格式和`RUN`类似
  - shell格式
  - exec格式

> Dockerfile中可以有多个`CMD`指令,但是只有最后一个生效,CMD指令会被`docker run`之后的参数置换    
> 例 `docker run -it -p 8080:8080 tomcat` 启动Tomcat     
> 若此时改为 `docker run -it -p 8080:8080 tomcat /bin/bash` ,那么Tomcat镜像中的Dockerfile最后的`CMD["catalina.sh","run"]` 就会被`/bin/bash`替换,导致直接进入容器内,而且Tomcat并未启动   

- `ENTRYPOINT` 类似`CMD`但是不会被`docker run`后面的指令覆盖
  - 而且这些`CMD`命令行参数会被当做参数送给`ENTRYPOINT`指令指定的程序
  -` ENTRYPOINT ["<executeable>","<param1>","<param2>",...]`

例1: Docker命令`docker run nginx:test`

```dockerfile
FROM nginx
ENTRYPOINT ["nginx","-c"]
CMD ["/etc/nginx/nginx.conf"]
```

上面的命令等同于 `nginx -c /etc/nginx/nginx.conf`

例2: `docker run nginx:test -c /etc/nginx/new.conf`,这条命令会将CMD的参数覆盖,最终执行`nginx -c /etc/nginx/new.conf`

## 自定义Centos环境 

[JDK镜像地址](https://mirrors.yangxingzhen.com/jdk/)

从原始的centos 具备`vim` `ifconfig` `jdk8`  

### 编写Dockerfile

1. 服务器上创建文件夹 `/myfile`
2. 创建`Dockerfile`文件

```dockerfile
FROM centos:7
MAINTAINER zzyy<zzyybs@126.com>
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
#安装vim编辑器
RUN yum -y install vim
#安装ifconfig命令查看网络IP
RUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

3. 构建 `docker build -t 新镜像名:TAG .` 例:`docker build -t centsojava8:1.5 .`
   1. 注意最后的`.`是指使用当前目录的 Dockerfile 创建镜像 ,也可以指定`docker build -f /path/to/a/Dockerfile .`
4. `docker run -it` 启动容器查看环境是否正确 

### 虚悬镜像 

```dockerfile
FROM ubuntu
CMD echo 'actionis success'
```

上面的dockerfile在构建后会产生一个虚悬镜像,或者在构建/删除镜像时出现错误也会产生虚悬镜像

> 虚线镜像的 `REPOSITORY` 和 `TAG` 都是`<none>` 俗称 dangling image

查看虚悬镜像 `docker image ls -f dangling=true`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/虚悬镜像.png)

删除所有的虚悬镜像 `docker image prune`


# Docker微服务 

1. 将微服务打为Jar包，并将Jar包文件上传到
2. 编写Dockerfile

```dockerfile
# 基础镜像使用java
FROM java:8

# 作者
MAINTAINER riiicc

# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
VOLUME /tmp

# 将jar包添加到容器中并更名为zzyy_docker.jar
ADD ric-docker-0.0.1-SNAPSHOT.jar ric-docker.jar

# 运行jar包

RUN bash -c 'touch /ric-docker.jar'

ENTRYPOINT ["java","-jar","/ric-docker.jar"]

#暴露6001端口作为微服务
EXPOSE 6001
```

3. 构建镜像 `docker build -t ric_docker:1.0 .`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/docker微服务打包.png)

4. 启动容器`docker run -d -p 6001:6001 f42df85f5c28`即可访问 


# docker 网络

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/docker虚拟网桥.png)

查看docker网络模式 `docker network ls`

```bash
[root@localhost mydocker]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
7b38adf1bbee   bridge    bridge    local
1a0c0748dbbf   host      host      local
696f97f1ac65   none      null      local
```


## 网络命令

命令 | 说明 |
---------|----------|
  `connect`   |  Connect a container to a network|
  `create`   |   Create a network|
  `disconnect` | Disconnect a container from a network|
  `inspect`  |   Display detailed information on one or more networks|
  `ls`     |     List networks|
  `prune`    |   Remove all unused networks|
  `rm`     |     Remove one or more networks|


## 作用
可以解决容器间的联网通信和端口映射    
在容器IP变动时可以通过**服务名直接进行网络通信**不受影响,类似微服务调用    


## 网络模式 
通过`docker network ls`可以看到三种网络模式,实际有**四种模式**

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/docker网络模式.png)


模式名称 | 使用 |
---------|----------|
 bridge 模式 | `--network bridge` | 
 host 模式 | `--network host` | 
 none 模式 | `--network none` | 
 container 模式 | `--network container:NAME/IMAGEID` | 


### bridge 网桥模式
1. 整个宿主机的网桥模式都是 `docker0` 类似一个交换机有一对接口,每个接口叫 `veth`,在本地主机和容器内分别创建一个虚拟接口,并让他们彼此联通   
2. 每个容器实例内部也有一块网卡,eth0   
3. `docker0` 上面的每一个`veth`匹配某个容器实例内部的`eth0`,两两配对,一一匹配   

最终,宿主机所有容器都将连接到这个内部网络上,几个容器在同一个网络下,会从这个网关下各自拿到分配的IP,此时各容器间的网络是互通的

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/bridge模式说明.png)


### host 主机模式
直接使用宿主机的IP地址与外界进行通信,容器将不会虚拟出自己的网卡而是使用宿主机的IP和端口    
使用命令启动容器 `docker run -d -p 8083:8080 --network host --name tomcat3 billygoo/tomcat8-jdk8`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/host模式启动tomcat容器.png)

此时可以看到`WARNING: Published ports are discarded when using host network mode`   

> docker启动时指定`--network=host或-net=host`，如果还指定了-p映射端口，那这个时候就会有此警告，并且通过-p设置的参数将**不会起到任何作用**，端口号会以主机端口号为主，**重复时则递增**    

此时我们只能采用访问宿主机对应端口来访问tomcat3 [http://192.168.0.103:8080/](http://192.168.0.103:8080/) ,而不是我们指定的端口映射   

在此基础上,我们再次创建一个`tomcat4`容器实例 `docker run -d --network host --name tomcat4 billygoo/tomcat8-jdk8`    
此时tomcat4实例启动端口默认递增为8081 [http://192.168.0.103:8081/](http://192.168.0.103:8081/)


### none模式
采用这个模式,即为禁用网络功能,只有lo标识,表示本地回环   
进入容器内只有一个lo网卡

```bash
root@e304171266ca:/usr/local/tomcat# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

### Container模式
新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等,两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。  

`docker run -it --name alpine1 alpine /bin/sh`    
`docker run -it --network container:alpine1 --name alpine2 alpine /bin/sh`    

容器`alpine2`使用`alpine1` 的网卡,此时若关闭`alpine1`容器,那么`alpine2`也会断网,只剩`lo`网卡


### 自定义模式
自定网络模式维护了主机名和ip的对应关系,可以在一个容器内通过`ping 容器名`连接另一个容器  


# docker compose 容器编排

`Docker-Compose` 是Docker官方的开源项目,负责实现Docker容器集群的快速编排   

例如要实现一个Web微服务项目，除了Web服务容器本身，往往还需要再加上后端的数据库mysql服务容器，redis服务器，注册中心，甚至还包括负载均衡容器等等

> Compose允许用户通过一个单独的`docker-compose.yml`模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建。Docker-Compose 解决了容器与容器之间如何管理编排的问题


## 安装
[说明文档](https://docs.docker.com/compose/compose-file/compose-file-v3/)

[安装文档](https://docs.docker.com/compose/install/)

`curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

使用国内加速   

`curl -L "https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

`chmod +x /usr/local/bin/docker-compose`

查看版本 `docker-compose --version`

## 核心概念 
- 文件`docker-compose.yml`
- 服务service 
  - 一个个应用容器实例,如:微服务,MySQL,redis,nginx
- 工程project
  - 由一组关联的应用容器组成一个完整的业务单元,在`docker-compose.yml`文件中定义 

## 拢共分三步

- 编写Dockerfile定义各个微服务应用并构建出对应的镜像文件
- 使用`docker-compose.yml`定义一个完整的业务单元,安排好整体应用中各个容器服务
- 执行`docker-compose up`命令,来启动整个应用程序,完成一键部署上线  


## 常用命令

```bash
docker-compose -h                           # 查看帮助

docker-compose up                           # 启动所有docker-compose服务

docker-compose up -d                        # 启动所有docker-compose服务并后台运行

docker-compose down                         # 停止并删除容器、网络、卷、镜像。

docker-compose exec  yml里面的服务id                 # 进入容器实例内部  docker-compose exec docker-compose.yml文件中写的服务id /bin/bash

docker-compose ps                      # 展示当前docker-compose编排过的运行的所有容器

docker-compose top                     # 展示当前docker-compose编排过的容器进程

docker-compose logs  yml里面的服务id     # 查看容器输出日志

docker-compose config     # 检查配置

docker-compose config -q  # 检查配置，有问题才有输出

docker-compose restart   # 重启服务

docker-compose start     # 启动服务

docker-compose stop      # 停止服务
```


## 编排微服务

1. 创建项目并打包 
2. 创建Dockerfile,然后执行构建命令 `docker build -t ric_docker:1.5 .`

```dockerfile
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER riiicc
# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为zzyy_docker.jar
ADD ric-docker-0.0.1-SNAPSHOT.jar ric_docker.jar
# 运行jar包
RUN bash -c 'touch /ric_docker.jar'
ENTRYPOINT ["java","-jar","/ric_docker.jar"]
#暴露6001端口作为微服务
EXPOSE 6001

```

3. 创建MySQL容器

```bash
docker run -p 3306:3306 --name mysql57 --privileged=true -v /zzyyuse/mysql/conf:/etc/mysql/conf.d -v /zzyyuse/mysql/logs:/logs -v /zzyyuse/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

# 进入 
docker exec -it mysql57 /bin/bash  
mysql -uroot -p
create database db2021;
use db2021;

CREATE TABLE `t_user` (
  `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(50) NOT NULL DEFAULT '' COMMENT '用户名',
  `password` VARCHAR(50) NOT NULL DEFAULT '' COMMENT '密码',
  `sex` TINYINT(4) NOT NULL DEFAULT '0' COMMENT '性别 0=女 1=男 ',
  `deleted` TINYINT(4) UNSIGNED NOT NULL DEFAULT '0' COMMENT '删除标志，默认0不删除，1删除',
  `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

4. 创建Redis容器 (实际操作可以对其进行配置)

```bash
docker run  -p 6379:6379 --name redis608 --privileged=true -v /app/redis/redis.conf:/etc/redis/redis.conf -v /app/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf
```

5. 启动微服务` docker run -d -p 6001:6001 ric_docker:1.5`


上述操作的问题:   
- 操作顺序固定,要先启动`redis` 和`mysql` 才能启动微服务
- 需要多次执行多个命令
- 容器间启停或宕机有可能导致IP变化导致映射不对应


## 使用Docker Compose 

```yaml
version: "3"
services:
  microService:
    image: ric_docker:1.6
    container_name: ms01
    ports:
      - "6001:6001"
    volumes:
      - /app/microService:/data
    networks: 
      - atguigu_net 
    depends_on: 
      - redis
      - mysql
  redis:
    image: redis:6.0.8
    ports:
      - "6379:6379"
    volumes:
      - /app/redis/redis.conf:/etc/redis/redis.conf
      - /app/redis/data:/data
    networks: 
      - atguigu_net
    command: redis-server /etc/redis/redis.conf
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: '123456'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'no'
      MYSQL_DATABASE: 'db2021'
      MYSQL_USER: 'zzyy'
      MYSQL_PASSWORD: 'zzyy123'
    ports:
       - "3306:3306"
    volumes:
       - /app/mysql/db:/var/lib/mysql
       - /app/mysql/conf/my.cnf:/etc/my.cnf
       - /app/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - atguigu_net
    command: --default-authentication-plugin=mysql_native_password #解决外部无法访问
networks: 
   atguigu_net: 
```

修改配置文件为

```properties
server.port=6001

spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://mysql:3306/db2021?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.druid.test-while-idle=false

spring.redis.database=0
spring.redis.host=redis
spring.redis.port=6379
spring.redis.password=123456
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-wait=-1ms
spring.redis.lettuce.pool.max-idle=8
spring.redis.lettuce.pool.min-idle=0

mybatis-plus.mapper-locations=classpath:mapping/*.xml
spring.swagger2.enabled=true
```

package 微服务并上传到服务器

执行命令`docker-compose up -d` 或 `docker-compose up`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockercompose启动.png)

启动成功后我们发现容器服务已经启动,而且docker network也多出了自定义网络模式

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockercompose启动成功.png)


关停所有服务 `docker-compose stop`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/dockercompose关停.png)



# Portainer 
Portainer 是一款轻量级的应用，它提供了图形化界面，用于方便地管理Docker环境，包括单机环境和集群环境。    
以图形化形式显示docker各项配置,容器信息等      

[官网](https://www.portainer.io/)

```bash
# 安装
docker run -d -p 8000:8000 -p 9000:9000 --name portainer     --restart=always     -v /var/run/docker.sock:/var/run/docker.sock     -v portainer_data:/data     portainer/portainer
```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/portainer.png)


> `--restart=always` 说明: 总是跟随容器启动


# 容器监控 CAdvisor + InfluxDB + Granfana
默认可以通过`docker stats` 命令查看docker中资源占用等信息    
`docker stats`统计结果只能是当前宿主机的全部容器，数据资料是实时的，没有地方存储、没有健康指标过线预警等功能

`CAdvisor`监控收集, `InfluxDB` 存储数据, `Granfana` 展示图表  

## CAdvisor
`CAdvisor`是一个容器资源监控工具,包括容器的内存,网络IO,磁盘IO等.同时提供了一个web页面查看容器的实时运行状态.CAdvisor 默认存储2分钟的数据,而且只针对单物理机,CAdvisor提供了很多数据集成接口,支持InfluxDB,Redis,Kafka,Elasticsearch等    

## InfluxDB
`InfluxDB` 是一个用go语言编写的开源分布式时序,时间和指标数据库


## Granfana
`Granfana` 是一个开源的数据监控分析可视化平台,支持多种数据源配置,支持图标权限控制和报警  


## 构建

```yaml
version: '3.1'
volumes:
  grafana_data: {}
services:
 influxdb:
  image: tutum/influxdb:0.9
  restart: always
  environment:
    - PRE_CREATE_DB=cadvisor # 先创建数据库
  ports:
    - "8083:8083"
    - "8086:8086"
  volumes:
    - ./data/influxdb:/data
 cadvisor:
  image: google/cadvisor
  links:
    - influxdb:influxsrv
  command: -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influxsrv:8086
  restart: always
  ports:
    - "8080:8080"
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
 grafana:
  user: "104"
  image: grafana/grafana
  user: "104"
  restart: always
  links:
    - influxdb:influxsrv
  ports:
    - "3000:3000"
  volumes:
    - grafana_data:/var/lib/grafana
  environment:
    - HTTP_USER=admin
    - HTTP_PASS=admin
    - INFLUXDB_HOST=influxsrv
    - INFLUXDB_PORT=8086
    - INFLUXDB_NAME=cadvisor
    - INFLUXDB_USER=root
    - INFLUXDB_PASS=root
```

# 最佳实践

## SpringBoot打包镜像 
Spring Boot 预装了自己的用于构建 Docker 镜像的插件，我们无需进行任何更改，因为它就在 pom.xml 中的 `spring-boot-starter-parent` 

`mvn spring-boot:build-image`

# 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV1gr4y1U7CY)
