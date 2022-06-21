## redis安装

### 1.下载
```

yum -y install gcc pcre-devel zlib-devel openssl openssl-devel

# 4.0版本
wget http://download.redis.io/releases/redis-4.0.14.tar.gz

# 最新stable
wget http://download.redis.io/redis-stable.tar.gz

tar -zxf redis-4.0.14.tar.gz

cd redis-4.0.14

make && make PREFIX=/usr/local/redis install


```
### 启动
```
修改redis.conf daemonize yes

./redis-server ./redis.conf

# 停

```
