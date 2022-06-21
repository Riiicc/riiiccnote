## MySQL5.7
#### 查看已安装的MySQL版本，并卸载
执行命令
rpm -qa|grep -i mysql  
yum remove +对应的包

#### 第一步：获取mysql YUM源

进入mysql官网获取RPM包下载地址(目前官网需要登录才能下载,群里有备份或者去别的服务器找备份)
```
https://dev.mysql.com/downloads/repo/yum/
```
#### 第二步：下载和安装mysql源

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
#### 第三步：在线安装Mysql
```
yum -y install mysql-community-server
或者
yum -y install mysql-server
```
**下载的东西比较多 要稍微等会**

#### 第四步：启动Mysql服务

systemctl start mysqld

#### 第五步：设置开机启动
```
systemctl enable mysqld
```
守护
```
systemctl daemon-reload
```
#### 第六步：修改root本地登录密码

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

#### 第七步：设置允许远程登录

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

#### 第八步：配置默认编码为utf8

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
#### 常见语句
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

#### mysql修改端口，添加用户，用户权限等
