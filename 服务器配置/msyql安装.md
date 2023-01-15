# MySQL5.7
# 安装
## 查看已安装的MySQL版本，并卸载
执行命令
rpm -qa|grep -i mysql  
yum remove +对应的包

## 第一步：获取mysql YUM源

进入mysql官网获取RPM包下载地址(目前官网需要登录才能下载,群里有备份或者去别的服务器找备份)
```
https://dev.mysql.com/downloads/repo/yum/
```
## 第二步：下载和安装mysql源

先下载 mysql源安装包
```
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm  或者
wget --no-check-certificate https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
-bash: wget: 未找到命令

我们先安装下wget 

yum -y install wget
```

安装mysql源
```
yum -y localinstall mysql57-community-release-el7-11.noarch.rpm
```

## 第三步：在线安装Mysql

```
yum -y install mysql-community-server
或者
yum -y install mysql-server
```
**下载的东西比较多 要稍微等会**

## 第四步：启动Mysql服务

`systemctl start mysqld`

## 第五步：设置开机启动

```
systemctl enable mysqld
```
守护
```
systemctl daemon-reload
```
## 第六步：修改root本地登录密码

mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个临时的默认密码。
```
密码在root@loaclhost : ******* ?LLptYNge4uX

[root@localhost ~]# vi /var/log/mysqld.log

[root@localhost ~]#  mysql -u root -p

Enter password: 

输入临时密码 进入mysql命令行；
855m@h!NN#Fr34vq$3rC
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'CUyB82gYP!ZeztDV';

Query OK, 0 rows affected (0.00 sec)
```
(备注 mysql5.7默认密码策略要求密码必须是大小写字母数字特殊字母的组合，至少8位)

## 第七步：设置允许远程登录

Mysql默认不允许远程登录，我们需要设置下，并且防火墙开放3306端口；
```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'CUyB82gYP!ZeztDV' WITH GRANT OPTION;

Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> exit;

Bye
```
防火墙设置 阿里云免装
```

[root@localhost ~]# firewall-cmd --zone=public --add-port=3306/tcp --permanent

success

[root@localhost ~]# firewall-cmd --reload

success

[root@localhost ~]# 
```

## 第八步：配置默认编码为utf8

修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：
```
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'

[root@localhost ~]# vi /etc/my.cnf
```
编辑保存完 重启mysql服务；
```
[root@localhost ~]# systemctl restart mysqld
[root@localhost ~]# 
查看下编码：
mysql> show variables like '%character%';
```
## 常见语句
Linux登陆测试 
ALTER USER 'root'@'localhost' IDENTIFIED BY 'N3Y!XU^D%pP3%NPEq7hW51USV^%jo&';
更改密码
```
update mysql.user set authentication_string=password('N3Y!XU^D%pP3%NPEq7hW51USV^%jo&') where user='root';

//navicat 命令行执行修改密码  这两个都要执行 针对远程和本地用户访问
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';

ALTER USER 'root'@'%' IDENTIFIED BY 'password';

```

root用户只有一个，但是mysql.user表中plugin使用的并不是mysql_native_password 而是auth_socket，导致用户连接一直使用unix socket空密码进入。修改plugin改回mysql_native_password，然后重新修改root密码并刷新全权限即可。

## mysql修改端口，添加用户，用户权限等


# 主从配置
## 清除原有的主从关系

### 从机slave操作

```bash

stop slave;

# 清除slave信息
reset slave all;

# 查看当前的slave状态 得到Empty set 的提示
mysql> show slave status\G
Empty set (0.00 sec)

```

### 主机Master 操作

```bash
#清除binlog
reset master

```

> reset master 后bin-log从00001开始，`Position`位置也会重置，

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql重置master.png)

### 双主搭建   
条件说明：   
- 主机1 IP `192.168.0.105`
- 主机2 IP `192.168.0.108`

**====Master1配置====**
1. `vim /etc/my.cnf`

```conf
server-id = 1         
log-bin = mysql-bin
#同步写磁盘  
sync_binlog = 1
binlog_checksum = none
binlog_format = mixed
auto-increment-increment = 2     
auto-increment-offset = 1    
slave-skip-errors = all  

```

2. 重启MySQL服务`systemctl restart mysqld`

3. 登录`mysql -uroot -p`  

```sql
#在主机MySQL里执行授权主从复制的命令 并创建slave1
GRANT REPLICATION SLAVE ON *.* TO 'slave1'@'192.168.0.108' IDENTIFIED BY 'CUyB82gYP!ZeztDV'; 

flush privileges;
# 锁表，待同步配置完成在解锁
flush tables with read lock;

```

4. 查看节点信息

`show master status;`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql主主备份截图1.png)

> 记住`File`的值和`Position`的值


**====Master2配置====**   
过程和1类似

```conf
server-id = 2        
log-bin = mysql-bin    
sync_binlog = 1
binlog_checksum = none
binlog_format = mixed
auto-increment-increment = 2     
auto-increment-offset = 2    
slave-skip-errors = all

```

```sql
GRANT REPLICATION SLAVE ON *.* TO 'slave1'@'192.168.0.105' IDENTIFIED BY 'CUyB82gYP!ZeztDV';

flush privileges;

flush tables with read lock;

```

show master status;

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql主主备份截图2.png)



**====Master1配置-2====**
```sql
-- 解锁
unlock tables;

stop slave;
-- 使用2号机的File 和Position

CHANGE MASTER TO MASTER_HOST='192.168.0.108',MASTER_USER='slave1',MASTER_PASSWORD='CUyB82gYP!ZeztDV',MASTER_LOG_FILE='riiicctest2-bin.000001',MASTER_LOG_POS=603;

start slave;

```

执行`show slave status \G;` 如图   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql主主备份截图.png)

**====Master2配置-1====** 
```sql
unlock tables;

stop slave;

CHANGE MASTER TO MASTER_HOST='192.168.0.105',MASTER_USER='slave1',MASTER_PASSWORD='CUyB82gYP!ZeztDV',MASTER_LOG_FILE='riiicctest-bin.000002',MASTER_LOG_POS=603;

start slave;

show slave status \G;
```

以上双主MySQL搭建成功  

### Nginx Stream 实现负载均衡
Nginx IP `192.168.0.103`   

1. 对两个MySQL执行如下命令，允许nginx主机对其进行访问   

```sql
GRANT ALL ON *.* TO 'root'@'192.168.0.103' IDENTIFIED BY 'CUyB82gYP!ZeztDV';

FLUSH PRIVILEGES;
```

1.  使用Nginx 的stream模块进行配置(需要在nginx安装时加入`--with-stream`)   

> stream模块一般用于tcp/UDP数据流的代理和负载均衡，可以通过stream模块代理转发TCP消息

```conf
stream {

    upstream mysql {

    #hash $remote_addr consistent;
    server 192.168.0.108:3306 max_fails=2 fail_timeout=10s;
    server 192.168.0.109:3306 max_fails=2 fail_timeout=10s;
    # 默认连接 连接数最少的
    least_conn;
    }

    server {

        listen 3306;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass mysql;
    }
}

```

此时访问192.168.0.103:3306 进行数据库访问，采用轮询方式
停掉一个 自动切换到另一个  

http://sirian369.com/article/2022/4/22/54.html  

https://www.cnblogs.com/effortsing/p/10073344.html 


### keepalived 实现双机热备

https://www.cnblogs.com/oppo/p/6728337.html








# 参考资料 
> - [双主结构MySQL](https://www.cnblogs.com/effortsing/p/10073344.html)