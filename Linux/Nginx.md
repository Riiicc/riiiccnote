# Nginx 



```bash
docker run -p 8081:80 --name nginx8081 -v /home/nginx8081/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx8081/conf/conf.d:/etc/nginx/conf.d -v /home/nginx8081/log:/var/log/nginx -v /home/nginx8081/html:/usr/share/nginx/html -d nginx:latest


docker run -p 8082:80 --name nginx8082 -v /home/nginx8082/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx8082/conf/conf.d:/etc/nginx/conf.d -v /home/nginx8082/log:/var/log/nginx -v /home/nginx8082/html:/usr/share/nginx/html -d nginx:latest

docker run -p 8083:80 --name nginx8083 -v /home/nginx8083/conf/nginx.conf:/etc/nginx/nginx.conf -v /home/nginx8083/conf/conf.d:/etc/nginx/conf.d -v /home/nginx8083/log:/var/log/nginx -v /home/nginx8083/html:/usr/share/nginx/html -d nginx:latest


```

# 介绍和安装  

## 常用版本 

- Nginx开源版 /Nginx plus商用版 
  - https://www.nginx.com/
- Openresty 
  - nginx + Lua
  - http://openresty.org/cn/
- Tengine淘宝 
  - nginx + C
  - http://tengine.taobao.org/

## 文件说明

- `conf`文件夹主要存放nginx配置文件，包括主配置文件，次配置
- `html`存放静态文件的默认目录html，css等
- `sbin`nginx主程序执行文件
- `logs`存放 访问日志`access.log` 错误日志`error.log` 程序pid`nginx.pid`

## 进程说明
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nginx进程模型.png)

1. 首先创建 Masetr process 主进程 开始读取并校验配置文件
2. 校验无误开启多个子进程接受响应请求，nginx启动后会存在两个进程
3. 主进程主要为了协调配置几个子进程  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nginx主进程和工作进程.png)


# Nginx配置与应用场景

## 默认最小配置解析

```conf
# 工作进程个数，启动时默认开启 1 个工作进程
worker_processes  1;

events {
    #单个工作进程可接受 1024个连接 
    worker_connections  1024;
}

http {
    # 引入 http mime类型  引入 mime.types文件
    include       mime.types;
    # 默认类型 mime.types 中不包含的类型 ，
    default_type  application/octet-stream;
    # 数据 零拷贝
    sendfile        on;
    # 保持连接 超时时间
    keepalive_timeout  65;

# 一个server 代表一个主机
    server {
        #当前主机的监听端口
        listen       80;
        #当前主机名，可以配置域名，主机名(host文件中的)
        server_name  localhost;

        # URI 域名之后的资源路径
        location / {
            #当前的请求根目录 在nginx安装目录下
            root   html;
            #默认页面
            index  index.html index.htm;
        }
        # 错误页面  服务器内部错误时 四个错误码都会转向到内部地址 /50x.html上  
        error_page   500 502 503 504  /50x.html;
        # 从root下找错误页面
        location = /50x.html {
            root   html;
        }
    }

}
```

- `mime.types`文件内包含了默认允许的mime类型解析，可以以解析（在浏览器中打开）大部分后缀文件，如pdf，MP3等
- `default_type` mime.types 中没有的采用默认格式进行解析
- `sendfile` 是否开启数据零拷贝，零拷贝是基于Linux内核的，减少数据复制次数，提高性能

## 虚拟主机和域名解析

**虚拟主机**：原本一个服务器只能应对一个站点，通过虚拟主机技术可以虚拟化成多个站点同事对外提供服务，通过 `server_name`进行配置

servername匹配规则   

```conf
# 完整匹配 
server_name vod.mmban.com www1.mmban.com;

#通配符匹配
server_name *.mmban.com

#通配符结束匹配
server_name vod.*;

# 正则匹配  
server_name ~^[0-9]+\.mmban\.com$;

```

适用于：  
- 多用户二级域名`api.baidu.com` `www.baidu.com` 
- 短网址
- httpdns

## 反向代理 




















# 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV1yS4y1N76R)