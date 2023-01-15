# Nginx 

# nginx 安装
## 下载  

```
https://nginx.org/download/
```
### 快捷安装

```
# 依赖
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel

wget https://nginx.org/download/nginx-1.12.2.tar.gz

tar -zxf nginx-1.12.2.tar.gz

cd nginx-1.12.2

./configure --prefix=/usr/local/nginx 

# 不采用默认配置 加入stream 模块
./configure --prefix=/usr/local/nginx --with-stream

make && make install

```

### 启动
```bash
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

/usr/local/nginx/sbin/nginx -s reload

/usr/local/nginx/sbin/nginx -s stop

# 配置文件测试
/usr/local/nginx/sbin/nginx -t

# windows 杀进程
taskkill /f /t /im nginx.exe


nginx #启动
nginx -s stop #快速停止
nginx -s quit #优雅关闭，在退出前完成已经接受的连接请求
nginx -s reload #重新加载配置
```

## 配置项