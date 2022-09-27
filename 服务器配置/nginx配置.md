# Nginx 


## nginx 安装

### 1.下载
```
https://nginx.org/download/
```
#### 快捷安装

```
# 依赖
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel

wget https://nginx.org/download/nginx-1.12.2.tar.gz

tar -zxf nginx-1.12.2.tar.gz

cd nginx-1.12.2

./configure --prefix=/usr/local/nginx

make && make install

```

### 2.启动
```bash
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

/usr/local/nginx/sbin/nginx -s reload

# windows 杀进程
taskkill /f /t /im nginx.exe
```

## 配置项