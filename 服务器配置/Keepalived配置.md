# Keepalived 

官网地址  https://www.keepalived.org/download.html  
官方仓库  https://github.com/acassen/keepalived

## 安装(有网络)

```bash
mkdir /usr/local/keepalived

tar -zxf keepalived-2.2.7.tar.gz

./configure --prefix=/usr/local/keepalived --sysconf=/etc

# 报错 安装环境
# no acceptable C compiler found in $PATH
yum -y install gcc-c++

# 报错 安装环境
# !!! OpenSSL is not properly installed on your system. !!!
yum -y install openssl openssl-devel 

# 再次执行
./configure --prefix=/usr/local/keepalived --sysconf=/etc  

# 查看是否成功 返回0 即成功
echo $? 

make && make install 

#查看服务状态
systemctl status keepalived.service
# 配置文件位置
cd /etc/keepalived/
# 复制模拟配置
cp keepalived.conf.sample  keepalived.conf

# 若不通过下面的复制，则会报错 Can't open PID file /run/keepalived.pid (yet?) after start: No such file or directory

#cp /usr/local/keepalived/etc/sysconfig/keepalived  /etc/sysconfig/keepalived
#cp /usr/local/keepalived/sbin/keepalived /usr/sbin/keepalived
#cp /usr/local/keepalived-2.2.7/keepalived/etc/init.d/keepalived  /etc/init.d/keepalived
#cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf

# https://blog.csdn.net/qq_42825214/article/details/105314603

# 启动 keepalived
systemctl start keepalived

# 修改配置后重启
systemctl restart keepalived

# 停止
systemctl stop keepalived

# 开机启动
systemctl enable keepalived


# 关闭防火墙：
systemctl stop firewalld

#　　禁用防火墙：
systemctl disable firewalld
```


## 安装 (无网络)
1. 依赖文件分别为 `keepalived-2.2.7.tar.gz` 和 `openssl-1.1.1s.tar.gz` 
2. 安装`zlib-devel` 采用 `zlib-devel-1.2.7-18.el7.x86_64.rpm` rpm包

```bash
rpm -ivh zlib-devel-1.2.7-18.el7.x86_64.rpm
```

3. 安装 `openssl`,将文件`openssl-1.1.1s.tar.gz`放在`/usr/local`,依次执行下面命令  

```bash
# openssl安装过程
tar -zxf openssl-1.1.1s.tar.gz

cd openssl-1.1.1s/

# 配置
./config --prefix=/usr/local/openssl shared zlib

make depend

make && make install

# 等待安装完成  需要等待比较长 。。。
```

4. 进行配置覆盖原来的openssl版本,依次或批量执行下面的命令  

```bash
echo "/usr/local/lib64/" >> /etc/ld.so.conf
ldconfig
mv /usr/bin/openssl /usr/bin/openssl.old
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
ldconfig -v

```
5. 此时通过下面的命令查看版本  

```bash
# 查看版本是否为1.1.1s
openssl version 

```

6. 解压安装  keepalived,将` keepalived-2.2.7.tar.gz` 放在`/usr/local`,依次执行下面命令  

```bash
tar -zxf keepalived-2.2.7.tar.gz  

cd keepalived-2.2.7

# 指定openssl编译
LDFLAGS="$LDFAGS -L /usr/local/openssl/lib"  ./configure  --prefix=/usr/local/keepalived --sysconf=/etc

make && make install 

#继续等待安装完成  。。。
```

7. 安装完成后，keepalived的安装位置位于 `/etc/keepalived`
8. 把配置文件`keepalived.conf`放入`/etc/keepalived/`中


## 日志

```bash
tail -f /var/log/messages
```

查看日志，可以看到启动过程中是否有报错(红框位置)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/keepalived日志标识.jpg)


## 配置参考

### MASTER主机

```conf
! Configuration File for keepalived

global_defs {
   router_id centos5
   script_user root
}
vrrp_script chk_nginx {		# 定义一个检测脚本，在global_defs之外配置
  script "/etc/keepalived/check_nginx.sh"	# 自己写的监测脚本
  interval 2	# 每2s监测一次
  weight 10		# 该参数用于指定当监测失效时，该设备的优先级会减少的值，该值为负表示减少
  fall 2        # 尝试两次都成功才成功
  rise 2        # 尝试两次都失败才失败
}
vrrp_instance VI_1 {
    state MASTER
    interface enp0s3
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.200
    }
}
```

### BACKUP从机

```conf
! Configuration File for keepalived

global_defs {
   router_id centos6
   script_user root
}
vrrp_script chk_nginx {		# 定义一个检测脚本，在global_defs之外配置
  script "/etc/keepalived/check_nginx.sh"	# 自己写的监测脚本
  interval 2	# 每2s监测一次
  weight 10		# 该参数用于指定当监测失效时，该设备的优先级会减少的值，该值为负表示减少
  fall 2        # 尝试两次都成功才成功
  rise 2        # 尝试两次都失败才失败
}
vrrp_instance VI_2 {
    state BUCKUP
    interface enp0s3
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.200
    }
}
```

### vrrp_script说明



## nginx探活脚本

```bash
#! /bin/bash


nginx_server=`ps -C nginx --no-header | wc -l`

if [ $nginx_server -gt 0 ];then
    exit 0
else
systemctl stop keepalived
fi

```

## 最终实现的参考连接

https://blog.csdn.net/yuzhenhaocugb/article/details/120208514

https://blog.csdn.net/D1179869625/article/details/126198495

https://zhuanlan.zhihu.com/p/578139761






