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
  - 文档 http://nginx.org/en/docs/
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

## 常用命令

```shell
./nginx -c /usr/local/nginx/nginx.conf

./nginx -t 

./nginx -s reload

./nginx -s stop

./nginx -s quit
# ngnix 编译报错 查看报错信息
systemctl status nginx -l
```

## 默认最小配置解析

```nginx
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

```nginx
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

### 正向代理和反向代理
**正向代理**：为客户端服务，代理的是客户端，代客户端进行请求    


**反向代理**：为服务端服务，代理的是服务端，代服务端进行响应接受请求

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nginx反向代理.png)

### 反向代理实例
通过`location`块中的`proxy_pass`进行反向代理站点`http://www.baidu.com`

```nginx
location / {
    proxy_pass http://www.baidu.com;
}
```

## 负载均衡

```nginx
   upstream backend {
    server 172.24.49.38:8000;
    server 172.24.49.39:8000;
    }

    server {
        listen       8080;
        server_name  localhost;


      location / {
    	  proxy_pass http://backend;
      }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
```

nginx的负载均衡策略有六种：  
- 轮询，默认配置
- `weight`权重，通过设置权重按比例访问，权重越高，访问次数越多

```nginx
   upstream backend {
    server 172.24.49.38:8000 weight=5;
    server 172.24.49.39:8000 weight=10;
    }
```

- `ip_hash`根据IP分配，固定的客户端请求访问固定的某个服务器，可以解决session不能跨服务器问题 
- `least_conn` 最少连接，将请求转发给连接数较少的服务器
- `url_hash` 需要第三方组件，按照俩的hash结果来分配请求，使得每个url定向到同一个后端服务器，配合缓存,适合访问固定资源
- `fair` 需要安装第三方组件，按照服务器的响应时间分配，相应端的优先分配

```nginx
   upstream backend {
    ip_hash;
    #least_conn;
    #url_hash;
    server 172.24.49.38:8000 weight=5;
    server 172.24.49.39:8000 weight=10;
    }
```


如果需要临时下线某个服务器可以进行配置`down`  
设置备份机，其他服务器不可用时，会接管服务`backup`     
```nginx
   upstream backend {
    server 172.24.49.38:8000 weight=5 down;
    server 172.24.49.39:8000 weight=10 backup;
    }
```

?> 最佳实践是通过lua脚本开发实现自定义规则的负载均衡

## 动静分离
原服务模型如下  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nginx动静分离示例.png)

动静分离，就是把static中的静态文件通过nginx的`location`直接映射，而不通过访问tomcat

```nginx
worker_processes  1;
 
events {
    worker_connections  1024;
}
 
http {
 
   server {
       listen       10000;
       server_name  localhost;
      
      #拦截后台请求
      location / {
        proxy_pass http://localhost:8888;
        proxy_set_header X-Real-IP $remote_addr;
      }
      # 单个映射
      location /css{
         root /home/static/css;
      }
 
      # 正则 拦截静态资源 映射目录
      location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
        root /Users/dalaoyang/Downloads/static;
       }
 
    }
 
}
```

## URL_Rewirte伪静态
通过配置对原url进行伪装，如`www.xxx.com/index.jsp?pageNum=2`伪装成`www.xxx.com/2.html` 

```nginx
server {
	listen 82;
	listen [::]:82;

	#root /var/www/web/index;
	#index index.html;
     
	server_name localhost;
	location / {
		rewrite ^/2.html$ /index.jsp?pageNum=2 break;
		proxy_pass http://192.168.xxx.xxx:8080;
		#try_files $uri $uri/ =404;
	}

	location ~*/(js|img|css) {
		root html;
		index index.html index.htm;
	}
}

```

`rewrite ^/2.html$ /index.jsp?pageNum=2 break;`    
`rewrite <regex> <replacement> [flag]`    

`^/2.html$`是正则匹配，可以换成别的   
`/index.jsp?pageNum=2` 替代内容  
`flag` 支持的flag标记:    
- last 本条匹配后继续向下匹配新的location URI  
- break 本条规则匹配完成即终止，不再匹配后面的任何规则  
- redirect 返回302临时重定向，浏览器地址栏会显示跳转后的URL地址  
- permanent 返回301永久重定向，（和上面无差，地址都会变成真实地址）  

## 伪静态+负载均衡

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nginxrulrewrite负载均衡.png)

```nginx
server {
	listen 82;
	listen [::]:82;

	#root /var/www/web/index;
	#index index.html;
    upstream totalurl {
    	server 192.168.xxx.xxx:8080 weight=8 break;
    	server 192.168.yyy.yyy:8080 weight=8 break;
    }
	server_name localhost;
	location / {
		rewrite ^/2.html$ /index.jsp?pageNum=2 break;
		proxy_pass http://totalurl;
		#try_files $uri $uri/ =404;
	}

	location ~*/(js|img|css) {
		root html;
		index index.html index.htm;
	}
}
```

## 防盗链
在访问静态页面，会请求页面中引用的静态文件`js` `图片`等，可以在其中的`Request Headers`看到静态文件多出一个`Referer`属性值就是请求静态页面的地址，代表这个静态文件的来源   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nginx防盗链referer.png)

被盗链时，`Referer`属性值就是盗链的地址，这样根据接收到的请求头中`Referer`的值来判定是否是盗链请求  

`valid_referers none | blocked | server_names | strings ....;` 

- none 检测referer头域不存在的情况 
  - `valid_referers none 192.168.0.1;` 允许不带referer或者带192.168.0.1的referer
- blocked 检测 Referer 头域的值被防火墙或者代理服务器删除或伪装的情况 
- server_names 设置一个或多个 URL ，检测 Referer 头域的值是否是这些 URL 中的某一个

```nginx
# 静态文件部分
location ~*/(js|img|css) {
    # 设置允许请求的地址
    valid_referers 192.168.0.1;
    
    # 判断是否盗链
    if($valid_referers){
        # 可以返回状态码或者其他页面等
        return 403;
    }
	root html;
	index index.html index.htm;
}
```

盗链返回指定页面

```nginx
location ~*/(js|img|css) {
    valid_referers 192.168.0.1;
    
    if($valid_referers){
        return 401;
        # return 401.html
    }
	root html;
	index index.html index.htm;
}

# 配置location
error_page 401 /401.html
location = /401.html{
    root html;
}
```

返回错误页面无法直观的感受提示，针对盗链图片可以直接返回一个错误图片提醒，通过`urlrewrite`来实现  

```nginx
location ~*/(js|img|css) {
    valid_referers 192.168.0.1;
    
    if($valid_referers){
        # 通过urlrewrite
        rewirte ^/ /img/error.png break;
    }
	root html;
	index index.html index.htm;
}
```


# Http和Https

p38-p51略过  


# 扩容

## iphash维持会话的特点及配置
iphash通过对用户IP进行hash后取余，根据得到的值来访问负载服务器   
例如：2台服务器采用iphash，`hash(ip)%2`得出的结果只有两种，0和1对应两台服务器

使用iphash的缺点：  
- 很容易造成流量倾斜，尤其局域网内，或者大量相同ip的用户访问，最终请求同一台服务器  
- 后端服务器宕机，会导致会话信息消失，需要重新认证


## 使用其他hash方案
- `hash  $cookie_jsessionid;` 这里的jsessionid 是浏览器总cookie的key，可以根据实际情况更换

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ngin_hashcookie示意图.png)

- `hash  $request_uri;`通过uri的hash来保证每个uri访问一个固定的服务器
- `使用lua逻辑定向分发`
- `Redis + SpringSession`

```nginx
    upstream totalurl {
        ip_hash;
        # hash  $cookie_jsessionid;
        # hash  $request_uri;
    	server 192.168.xxx.xxx:8080 weight=8 break;
    	server 192.168.yyy.yyy:8080 weight=8 break;
    }
```

## 使用sticky模块完成对Nginx的负载均衡

nginx 官方sticky配置说明   
http://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky

`tengine`中有`session_sticky模块`我们通过第三方的方式安装在开源版本中

sticky是第三方模块，需要重新编译Nginx,他可以对Nginx这种静态文件服务器使用基于cookie的负载均衡

### 下载模块

**项目官网**

https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/src/master/

另外一个版本

https://github.com/bymaximus/nginx-sticky-module-ng

**下载**

https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/get/1.2.6.zip


### 重新编译Nginx

依赖`openssl-devel`

**进到源码目录重新编译**，这里的`--add-module`就是添加第三方模块，`--with-module`是加载内部模块 

```shell
./configure --prefix=/usr/local/nginx --add-module=/root/nginx-goodies-nginx-sticky-module-ng-c78b7dd79d0d
```

**执行make**

**如遇报错修改源码**

打开 `ngx_http_sticky_misc.c`文件

在12行添加

```c
#include <openssl/sha.h>
#include <openssl/md5.h>
```



**备份之前的程序**

```
mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
```



**把编译好的Nginx程序替换到原来的目录里，或者使用`make install`(本质也是将编译好的文件，复制到对应位置，但是会覆盖配置文件！)**

```shell
cp objs/nginx /usr/local/nginx/sbin/
```

**升级检测**

```
make upgrade
```

检查程序中是否包含新模块

```
nginx -V
```

**配置方法**

```nginx
upstream httpget {
# sticky;
sticky name=route expires=6h;

server 192.168.44.102;
server 192.168.44.103;
}
```

`sticky`也是通过浏览器中设置一个`route`key的cooike 来维持会话，保证固定客户端访问固定服务器    
- `name=route`指定cookie名称
- `expires=6h`设置cookie的有效期，不配置就是永久 


## Keepalive   
HTTP是短连接，client向server发送一个request，得到response后，连接就关闭。而通常一个网页可能有多种请求类型，图片，静态资源，请求等，只有所有资源加载完毕后才能用网站完整功能，**如果每请求一个资源就创建一个链接，代价太大**，这样就需要`Keep-alive`   

基于此背景，我们希望连接能够在短时间内得到复用，加载同一个网页内容时，尽量复用连接,这就是Keep-alive  
- `HTTP 1.0` 中默认是关闭的，需要在http头加入"Connection: Keep-Alive"，启用Keep-Alive；
- `HTTP 1.1` 中默认启用Keep-Alive，如果加入"Connection: close "，关闭Keep-Alive。


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/keepalive请求示例.png)

**适用情况：**  
明显的预知用户会在当前连接上有下一步操作   
复用连接，有效减少握手次数，尤其是https建立一次连接开销会更大

**不适用情况：**   
访问内联资源一般用缓存，不需要keepalive  
长时间的tcp连接容易导致系统资源无效占用


### nginx中的keep-alive
文档地址 http://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream  


```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;


    sendfile        on;


    keepalive_timeout  65 65; #超过这个时间 没有活动，会让keepalive失效 
    keepalive_time 1h; # 一个tcp连接总时长，超过之后 强制失效
  
    send_timeout 60;# 默认60s  此处有坑！！ 系统中 若有耗时操作，超过 send_timeout 强制断开连接。 注意：准备过程中，不是传输过程


    keepalive_requests 1000;  #一个tcp复用中 可以并发接收的请求个数
}
```

`keepalive_timeout  65;`表示连接保持65秒，超出这个时间就会重新建立连接  
`keepalive_timeout  0;`关闭keepalive,nginx响应中不会启用

**配置项说明：**   
- `keepalive_timeout`连接保留时间，超时失效
- `keepalive_disable`不对某些浏览器建立长连接,默认`msie6`IE6
- `keepalive_requests`默认1000，一个tcp复用中，可以并发接收的请求个数
- `send_timeout` 默认60s，两次向客户操作之间的间隔，如果大于这个时间则强制关闭连接
- `keepalive_time`一个tcp连接总时长，超过之后强制失效,默认1h
- `kepalive` 设置每个工作进程保留到上游服务器的最大空闲保持连接数，超出会将最近最少使用的连接关闭。


## 反向代理内存与文件缓冲流程
P70 详细讲解


## UpStream工作流程
proxy_pass 向上游服务器请求数据共有6个阶段

- 初始化
- 与上游服务器建立连接
- 向上游服务器发送请求
- 处理响应头
- 处理响应体
- 结束 


- `proxy_set_header `设置header
- `proxy_connect_timeout`与上游服务器(真实服务)连接超时时间,即后端服务器处理请求的时间、超时快速失败
- `proxy_send_timeout`定义nginx向后端服务发送请求的间隔时间(不是耗时)。默认60秒，超过这个时间会关闭连接
- `proxy_read_timeout`后端服务给nginx响应的时间，规定时间内后端服务没有给nginx响应，连接会被关闭，nginx返回504 Gateway Time-out。默认60秒

```nginx
upstream http_backend {
    server 127.0.0.1:8080;
}

server {

    location /http/ {
        proxy_pass http://http_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_connect_timeout 60;
        proxy_send_timeout 60;
        proxy_read_timeout 60
    }
}
```

### 缓冲区
接收来自服务端(上游服务器)的请求限制  

- `proxy_buffer_size 4k` header缓冲区大小 
- `proxy_requset_buffering on|off` 是否完全读到请求体之后再向上游服务器发送请求
- `proxy_buffering  on|off` 是否缓冲上游服务器数据
- `proxy_buffers 32 64k;`缓冲区大小 32个 64k大小内存缓冲块
- `proxy_max_temp_file_size`临时文件的最大大小(默认1G),缓冲区内存不够就写到磁盘里  
- `proxy_temp_file_write_size 8k`当启用从代理服务器到临时文件的响应的缓冲时，限制一次写入临时文件的数据的大小。默认情况下，大小由proxy_buffer_size和proxy_buffers指令设置的两个缓冲区限制。
- `proxy_max_temp_file_size 1024m;`临时文件最大值
- `proxy_temp_path 1 2` 缓冲临时文件路径，后面的1 2表示文件层级，防止单一目录下文件过多，查找繁琐

> ```
> proxy_temp_path /spool/nginx/proxy_temp 1 2;
> ```

生成的临时文件夹示例

> ```
> /spool/nginx/proxy_temp/7/45/00000123457
> ```

```nginx
server {

    location /http/ {
        proxy_pass http://http_backend;
        proxy_requset_buffering on;
        proxy_buffering on;
        proxy_buffer_size 64k;
        proxy_buffers 32 128k;
        proxy_busy_buffers_size 8k;
        proxy_max_temp_file_size 1024m;
    }
}

```

### 对客户端的限制
可配置位置  
- `http`
- `server`
- `location`



- `client_body_buffer_size` 对客户端请求中的body缓冲区大小,默认32位8k 64位16k如果请求体大于配置，则写入临时文件
- `client_header_buffer_size`设置读取客户端请求体的缓冲区大小。 如果请求体大于缓冲区，则将整个请求体或仅将其部分写入临时文件。 默认32位8K。 64位平台16K。  
- `client_max_body_size 1000M;`默认1m，如果一个请求的大小超过配置的值，会返回413 (request Entity Too Large)错误给客户端,将size设置为0将禁用对客户端请求正文大小的检查。  
- `client_body_timeout`指定客户端与服务端建立连接后发送 request body 的超时时间。如果客户端在指定时间内没有发送任何内容，Nginx 返回 HTTP 408（Request Timed Out）
- `client_header_timeout`客户端向服务端发送一个完整的 request header 的超时时间。如果客户端在指定时间内没有发送一个完整的 request header，Nginx 返回 HTTP 408（Request Timed Out）。
- `client_body_temp_path path [level1 [level2 [level3]]]`在磁盘上客户端的body临时缓冲区位置,同样是层级设置
- `client_body_in_file_only on;`把body写入磁盘文件，请求结束也不会删除
- `client_body_in_single_buffer`尽量缓冲body的时候在内存中使用连续单一缓冲区，在二次开发时使用`$request_body`读取数据时性能会有所提高
- `client_header_buffer_size` 设置读取客户端请求头的缓冲区大小，如果一个请求行或者一个请求头字段不能放入这个缓冲区，那么就会使用large_client_header_buffers
- `large_client_header_buffers`默认8k


## 获取客户端真实IP
### X-Real-IP
额外模块，不推荐使用

### setHeader
> X-Forwarded-For（XFF）是用来识别通过HTTP代理或负载均衡方式连接到Web服务器的客户端最原始的IP地址的HTTP请求头字段。

```nginx
proxy_set_header X-Forwarded-For $remote_addr;
```

## Gzip压缩
文档地址http://nginx.org/en/docs/http/ngx_http_gzip_module.html

Nginx开启Gzip压缩功能， 可以使网站的`css、js 、xml、html` 文件在传输时进行压缩，提高访问速度, 进而优化Nginx性能!在Nginx配置文件中可以配置Gzip的使用，相关指令可以在http区域server区域、location区域配置。Nginx可以通过`ngx_http_gzip_module`模块、`ngx_http_gzip_static_module`模块和`ngx_http_gunzip_module`模块对这些指令进行解析和处理   


作用域 `http, server, location`

**配置项**：  
- `gzip on;`开关，默认关闭
- `gzip_buffers 32 4k|16 8k`缓冲区大小
- `gzip_comp_level 1;`压缩等级 1-9，数字越大压缩比越高
- `gzip_http_version 1.1;`使用gzip的最小版本
- `gzip_min_length`设置将被gzip压缩的响应的最小长度,大于这个长度就会被压缩长度仅由“Content-Length”响应报头字段确定。
- `gzip_proxied` 多选,
  - off 为不做限制作为反向代理时，针对上游服务器返回的头信息进行压缩
  - expired - 启用压缩，如果header头中包含 "Expires" 头信息
  - no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
  - no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
  - private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
  - no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
  - no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
  - auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
  - any - 无条件启用压缩
- `gzip_vary on;`增加一个header，适配老的浏览器，头多信息`Vary: Accept-Encoding`
- `gzip_types`哪些mime类型的文件进行压缩
- `gzip_disable`禁止某些浏览器使用gzip

### 完整实例

```nginx
  gzip on;
  gzip_buffers 16 8k;
  gzip_comp_level 6;
  gzip_http_version 1.1;
  gzip_min_length 256;
  gzip_proxied any;
  gzip_vary on;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";
```

```http
HTTP/1.1 200
Server: nginx/1.21.6
Date: Wed, 18 May 2022 17:42:35 GMT
Content-Type: text/html;charset=utf-8
Content-Length: 7832
Connection: keep-alive
Keep-Alive: timeout=65
```

### 动态压缩和静态压缩  
启用Gzip的动态压缩后，无法使用`sendfile on`通过零拷贝高效传输数据，所以需要静态压缩来提高效率  

> nginx模块中分别有`ngx_http_gzip_module`和`ngx_http_gzip_static_module`两个模块，分别对应动态压缩和静态压缩



#### http_gzip_static_module
重新编译nginx

```
./configure --with-http_gzip_static_module
```
  
配置实例`gzip_static on | off | always;`    
- on就是开启静态压缩，同时检查客户端能否支持，若不支持就无效，相当于关闭
- off关闭静态压缩
- always 永久开启，并不检查客户端是否支持都使用gzip静态压缩

> 当在配置了always时，可以使用另一个模块`ngx_http_gunzip_module`来解压已经压缩的静态资源，再传输给不支持静态压缩的客户端  

### ngx_http_gunzip_module
安装时使用`--with-http_gunzip_module `进行配置

- `gunzip on | off;`为缺少 gzip 支持的客户端启用或禁用 gzip 响应的解压缩。。
- `gunzip_buffers number size;`默认`gunzip_buffers 32 4k|16 8k;`

## Brotli压缩
插件地址 https://github.com/google/ngx_brotli 
算法安装包 https://github.com/google/brotli

- 首先下载连个安装包，解压后将算法包根目录所有内容拷贝到插件包中`dep/brotli`文件夹下  
- 采用nginx模块化编译

```shell
./configure --with-compat --add-dynamic-module=/root/ngx_brotli-1.0.0rc --prefix=/usr/local/nginx/
```  

- 命令 `make modules` 会在objs 文件夹下生成下面两个文件，以及编译完成的nginx主程文件
- 将`ngx_http_brotli_filter_module.so` `ngx_http_brotli_static_module.so`拷贝到`/usr/local/nginx/modules/`
- 复制nginx主程序
- 配置文件中添加

```nginx
load_module "/usr/local/nginx/modules/ngx_http_brotli_filter_module.so";
load_module "/usr/local/nginx/modules/ngx_http_brotli_static_module.so";

http{
     server {
        
        location / {
            brotli on;
            brotli_static on;
	        brotli_comp_level 6;
	        brotli_buffers 16 8k;
	        brotli_min_length 20;
	        brotli_types text/plain text/css text/javascript application/javascript text/xml application/xml application/xml+rss application/json image/jpeg image/gif image/png;
            root   html;
            index  index.html index.htm;
        }
    }
}
```

## 合并客户端请求

### Concat模块
Tengine发布的Concat模块通过合并几个css或js请求来减少请求数，提高请求效率  

git地址 https://github.com/alibaba/nginx-http-concat


```nginx
location /static/js/ {
    concat on;
    concat_max_files 30;
}
```

## 资源静态化方案  

集数P85  
- 合并文件输出
- 集群文件同步`rsync`    

**前端：**采用节约服务器端的计算资源，请求合并   
**后端：**采用SSI模块


### SSI合并服务器端文件 
SSI是一种基于服务端的网页制作技术，nginx上的模板引擎

官方文档
http://nginx.org/en/docs/http/ngx_http_ssi_module.html

#### 配置指令
SSI在`nginx.conf`的配置项说明，具体参考官方文档  

- `ssi_min_file_chunk`向磁盘存储并使用sendfile发送，文件大小最小值
- `ssi_last_modified` 是否保留lastmodified
- `ssi_silent_errors on | off;` 是否显示逻辑错误，配置语法不正确时，页面是否有错误提醒
- `ssi_value_length`限制脚本参数最大长度
- `ssi_types` 默认text/html;如果需要其他mime类型 需要设置

#### 配置代码
配置通用格式`!--# command parameter1=value1 parameter2=value2 ... -->`   
**详细见官方文档，实现方式参考模板引擎**


```html
静态文件直接引用
<!--# include file="footer.html" -->

可以指向location，而不一定是具体文件
<!--# include virtual="/remote/body.php?argument=value" -->

```

### 静态资源同步方案rsync  
p89-p96 

https://www.samba.org/ftp/rsync/rsync.html

`remote synchronize`是一个远程数据同步工具，可通过 LAN/WAN 快速同步多台主机之间的文件。也可以使用 rsync 同步本地硬盘中的不同目录。   
`rsync` 是用于替代 rcp 的一个工具，rsync 使用所谓的 rsync算法 进行数据同步，这种算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快。

rsync 基于inotify 开发

Rsync有三种模式：   
- 本地模式（类似于cp命令）
- 远程模式（类似于scp命令）
- 守护进程（socket进程：是rsync的重要功能）

#### rsync 常用选项



| 选项     | 含义                                                         |
| :------- | :----------------------------------------------------------- |
| -a       | 包含-rtplgoD                                                 |
| -r       | 同步目录时要加上，类似cp时的-r选项                           |
| -v       | 同步时显示一些信息，让我们知道同步的过程                     |
| -l       | 保留软连接                                                   |
| -L       | 加上该选项后，同步软链接时会把源文件给同步                   |
| -p       | 保持文件的权限属性                                           |
| -o       | 保持文件的属主                                               |
| -g       | 保持文件的属组                                               |
| -D       | 保持设备文件信息                                             |
| -t       | 保持文件的时间属性                                           |
| –delete  | 删除DEST中SRC没有的文件                                      |
| –exclude | 过滤指定文件，如–exclude “logs”会把文件名包含logs的文件或者目录过滤掉，不同步 |
| -P       | 显示同步过程，比如速率，比-v更加详细                         |
| -u       | 加上该选项后，如果DEST中的文件比SRC新，则不同步              |
| -z       | 传输时压缩                                                   |

#### 安装
两端安装

```
yum install -y rsync
```

创建密码文件文件`/etc/rsync.password` 内容：

```
hello:123
```

修改权限

```
chmod 600 /etc/rsync.password
```

修改配置

```
auth users = sgg
secrets file = /etc/rsyncd.pwd
```

开机启动

在`/etc/rc.local`文件中添加

```
rsync --daemon
```

修改权限

```
echo "sgg:111" >> /etc/rsyncd.passwd
```
#### 查看远程目录

```
rsync --list-only 192.168.44.104::www/
```

#### 拉取数据到指定目录

```
rsync -avz rsync://192.168.44.104:873/www

rsync -avz 192.168.44.104::www/ /root/w
```

##### 使用SSH方式

```
rsync -avzP /usr/local/nginx/html/ root@192.168.44.105:/www/
```

#### 客户端免密

客户端只放密码

```
echo "111" >> /etc/rsyncd.passwd
```

此时在客户端已经可以配合脚本实现定时同步了



#### 如何实现推送？

修改配置

```
rsync -avz --password-file=/etc/rsyncd.passwd.client /usr/local/nginx/html/ rsync://sgg@192.168.44.105:/www
```

```--delete 删除目标目录比源目录多余文件```

#### 实时推送

推送端安装inotify

依赖

```
yum install -y automake
```



```
wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz
./configure --prefix=/usr/local/inotify
make && make install
```



监控目录

```
/usr/local/inotify/bin/inotifywait -mrq --timefmt '%Y-%m-%d %H:%M:%S' --format '%T %w%f %e' -e close_write,modify,delete,create,attrib,move //usr/local/nginx/html/

```



#### 简单自动化脚本

```shell
#!/bin/bash

/usr/local/inotify/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f %e' -e close_write,modify,delete,create,attrib,move //usr/local/nginx/html/ | while read file
do
       
        rsync -az --delete --password-file=/etc/rsyncd.passwd.client /usr/local/nginx/html/ sgg@192.168.44.102::ftp/
done

```


## 多级缓存   
- 静态资源缓存
- 浏览器缓存
- CDN缓存
- 正向代理缓存
- 反向代理缓存
- Nginx内存缓存
- 外置内存缓存
- 上游服务器应用缓存


### 浏览器缓存  
`memorycache`字面理解是从内存中，其实也是字面的含义，这个资源是直接从内存中拿到的，**不会请求服务器**一般已经加载过该资源且缓存在了内存当中，当关闭该页面时，此资源就被内存释放掉了，再次重新打开相同页面时不会出现from memory cache的情况

`diskcache`是从磁盘当中取出的，也是在已经在之前的某个时间加载过该资源，**不会请求服务器**但是此资源不会随着该页面的关闭而释放掉，因为是存在硬盘当中的，下次打开仍会from disk cache

> 内存缓存`memorcache`适用于更新频率较高的资源(存储时间短)，例如，图片，图标等  
> 磁盘缓存`diskcache`适用于更新频率较低的资源(存储时间长)，例如，js，css  

?> 电脑中的清理软件会影响缓存保存，可能导致重复获取资源(资源未过期就被清理)  

#### 强制缓存和协商缓存  
示例ResponseHeader  

```
Accept-Ranges: bytes
Cache-Control: max-age=315360000
Content-Length: 12802
Content-Type: image/png
Date: Tue, 17 Jan 2023 02:34:54 GMT
Etag: "3202-5a533d00d4900"
Expires: Fri, 14 Jan 2033 02:34:54 GMT
Last-Modified: Sat, 09 May 2020 09:33:56 GMT
Server: Apache
```


**协商缓存**主要由 `ETag` 和 `Last-Modified` 两个字段来实现，浏览器发起请求时`RequestHeader`中`If-Modified-Since`和`If-None-Match`分别对应`ResponseHeader`中的`Last-Modified`和`ETag`，两两对应。  
**协商缓存在Nginx中是默认启用的**   

- `Last-Modified` 通常是文件最后更新的日期时间戳
- `ETag` 是一个用于映射web资源的映射token(hash)，这个token应该满足唯一对应到一个web服务器上的静态资源,用于校验文件变化

```nginx
...
location{
    #关闭协商缓存  
    etag off;
    
    # 覆盖responseheader 强制赋空值
    add_header Last-Modified "";
    # 或者使用关闭If-Modified-Since 和上面的add_header二选一即可
    if_modified_since off;

    ...
}
```



**强制缓存**的思想是，在浏览器内置数据库中缓存每次请求中"可以被缓存"（受到一些关键字的管控）的静态资源如 `image, css, js` 文件，当第二次请求被缓存过的资源时候，会通过校验两个字段 `Expires` 和 `Cache-Control` 的`max-age`字段（注意，Expires 是 http1.0 的产物， Cache-Control 则是 http1.1 的产物。 两者同时存在， 或者只存在其中之一， 都可以触发强制缓存)   


- `Cache-Control: max-age`请求缓存时长(秒)
- `Expires`是指缓存有效时间(返回的是服务器的时间可能存在时差，**或导致缓存永不生效**)

首先校验`Cache-Control`通过则直接使用缓存，不通过则校验`Expires`  

```nginx
...
location{
    etag off;
    add_header Last-Modified "";
    # 设置过期时间300s/m//h/d
    expires 300s;
    ...
}
```

相应的可以在`ResponseHeader`中看到`Expires`中写的过期时间，按照（服务器+300s）时间推算

> **最佳实践** 强制缓存+协商缓存，首次打开页面采用强制缓存，强制缓存过期，就访问协商缓存信息，若协商缓存未命中，进行请求数据更新四个属性字段信息Expires、max-age、Etag、Last-Modified


#### 注意事项  
- 多集群负载时`last-modified`必须保持一致
- 一些场景需要禁用浏览器缓存，可以采用动态url，添加额外hash字符串，版本号，数字等来防止缓存
- 对于js、css缓存很久的资源，可以通过添加版本号的方式更新内容
- 不需要强一致性的数据，可以缓存几秒
- 异步加载的接口数据，可以使用`ETag`来校验,后端返回通过**切面**手动添加
- 在服务器添加Server头，有利于排查错误
- 分为手机APP和Client以及是否遵循http协议
- 在没有联网的状态下可以展示数据
- 流量消耗过多
- 提前下发避免秒杀时同时下发数据造成流量短时间暴增
- 兜底数据在服务器崩溃和网络不可用的时候展示
- 临时缓存退出即清理
- 固定缓存展示框架这种，可能很长时间不会更新，可用随客户端下发
- **首页**有的时候可以看做是框架 应该禁用缓存，以保证加载的资源都是最新的
- 父子连接 页面跳转时有一部分内容不需要重新加载，可用从父菜单带过来
- 预加载 某些逻辑可用**判定用户接下来的操作**，那么可用异步加载那些资源
- **骨架屏**加载过程异步加载先展示框架，然后异步加载内容，避免主线程阻塞


### 代理缓存

#### nginx正向代理实现

```nginx
location{

    proxy_pass $scheme://$host$request_uri;
    resolver 8.8.8.8;
}
```

`$scheme`协议转发，`$host` ip地址，`$request_uri`请求路径  

#### 代理https请求

需要第三方模块

https://github.com/chobits/ngx_http_proxy_connect_module

配置

```nginx
 server {
     listen                         3128;

     # dns resolver used by forward proxying
     resolver                       8.8.8.8;

     # forward proxy for CONNECT request
     proxy_connect;
     proxy_connect_allow            443 563;
     proxy_connect_connect_timeout  10s;
     proxy_connect_read_timeout     10s;
     proxy_connect_send_timeout     10s;

     # forward proxy for non-CONNECT request
     location / {
         proxy_pass http://$host;
         proxy_set_header Host $host;
     }
 }
```



### 代理缓存 proxy 

官网文档  
http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache

#### 配置

```nginx
http{
...
# 配置缓存信息    路径目录      文件目录层级 缓存名称:内存可用大小  缓存文件保存时长1天                  
proxy_cache_path /ngx_tmp levels=1:2 keys_zone=test_cache:100m inactive=1d max_size=10g ;

server{
    ...
    location{
        # upstream_cache_status 缓存是否命中的标识
        # responseheaders 中可以看Nginx-Cache HIT即为命中缓存,反之为MISS  
        add_header  Nginx-Cache "$upstream_cache_status";

        # 缓存key 对应keys_zone 缓存名称
        proxy_cache test_cache;
        # 缓存有效期 一般小于inactive 
        proxy_cache_valid 168h;

        root html;
    }
}

}
```

`proxy_cache_path`配置项说明    
- `/ngx_tmp`是路径目录
- `levels=1:2` 目录层级数及目录名称位数，取mdb5后几位  
  - 案例：`a15d21adf452df`根据规则生成的目录文件为`/f/2d/a15d21adf45`
- `keys_zone=test_cache:100m`缓存名称为test_cache
  - `100m`允许内存占用保存缓存hash方便查找的最大空间
- `inactive=1d` 缓存文件保存时长一般大于`proxy_cache_valid`缓存有效期
- `max_size=10g`缓存静态文件最大占用磁盘空间
- `proxy_cache_use_stale` 默认off在什么时候可以使用过期缓存

可选`error` | `timeout` | `invalid_header` | `updating` | `http_500` | `http_502` | `http_503` | `http_504` | `http_403` | `http_404` | `http_429` | `off`

- `proxy_cache_background_update`默认off运行开启子请求更新过期的内容。同时会把过期的内容返回给客户端

- `proxy_no_cache`  `proxy_cache_bypass`指定什么时候不使用缓存而直接请求上游服务器

```
proxy_no_cache $cookie_nocache $arg_nocache$arg_comment;
proxy_no_cache $http_pragma    $http_authorization;
```

如果这些变量如果存在的话不为空或者不等于0，则不使用缓存

- `proxy_cache_convert_head` 默认 on是否把head请求转换成get请求后再发送给上游服务器 以便缓存body里的内容如果关闭 需要在 `cache key` 中添加 $request_method 以便区分缓存内容

- `proxy_cache_lock` 默认off缓存更新锁
- `proxy_cache_lock_age`默认5s缓存锁超时时间




### 缓存清理 

#### purger插件
**purger**   
需要第三方模块支持  
https://github.com/FRiCKLE/ngx_cache_purge

配置

```nginx

        location ~ /purge(/.*) {
            #$1 代表请求后的uri
            proxy_cache_purge  test_cache  $1;
        }
        自定义cachekey
         proxy_cache_key $uri;
```

通过访问`localhost/purge/`来进行缓存清理  

`proxy_cache_key` 默认`$scheme$proxy_host$request_uri`缓存的key，即根据请求地址生成key,根据插件文档可以配置多种组合       

`proxy_cache_revalidate` 如果缓存过期了，向上游服务器发送“If-Modified-Since” and “If-None-Match来验证是否改变，如果没有就不需要重新下载资源了

`proxy_cache_valid`可以针对不容http状态码设置缓存过期时间不设置状态码会默认200, 301, 302

```nginx

proxy_cache_valid 200 302 10m;
proxy_cache_valid 301      1h;
proxy_cache_valid any      1m;

```

any指其他任意状态码

### 断点续传缓存 range
观看视频时拖动/点击滚动条，就可以使用range来实现     

文档地址 http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_max_range_offset

在反向代理服务器中需向后传递header

```nginx
proxy_set_header Range $http_range;
```

proxy_cache_key中增加range

`proxy_cache_max_range_offset` range最大值，超过之后不做缓存，默认情况下 不需要对单文件较大的资源做缓存

`proxy_cache_methods`默认`head` get

`proxy_cache_min_uses` 默认1被请求多少次之后才做缓存

`proxy_cache_path` path指定存储目录

`use_temp_path`默认创建缓存文件时，先向缓冲区创建临时文件，再移动到缓存目录

### Nginx内存缓存  
索引内存缓存理解为上面的`keys_zone=test_cache:100m`缓存名称为test_cache `100m`允许内存占用保存缓存hash方便查找的最大空间    
还有就是workerprocess进程的共享缓存


### 应用缓存与多级缓存整体结构
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nginx的应用缓存和多级缓存整体结构.png)


### strace追踪内核对sendfile缓存调优

```shell
yum -y install strace 

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/strace尝试.png)

### sendfile 优化
在`sendfile on`的情况下**open_file_cache**配置文件缓存调优  

```nginx
location{
    open_file_cache max=500 inactive=60s
    open_file_cache_min_uses 1; 
    open_file_cache_valid 60s; 
    open_file_cache_errors on
}
```

- `max`缓存最大数量，超过数量后会使用`LRU`淘汰
  - `LRU(Least Recently Used)`是一种内存淘汰算法，逐渐淘汰掉使用频率最低的数据
- `inactive` 指定时间内未被访问过的缓存将被删除,自动过期
- `pen_file_cache_min_uses`被访问到多少次后会开始缓存
- `open_file_cache_valid`间隔多长时间去检查文件是否有变化
- `open_file_cache_errors`对错误信息是否缓存

### 外置内存缓存

#### error_page和匿名location

```nginx
http{
    ...
    server {
        ...
        # 指定具体跳转地址
        error_page 404 http://www.itcast.cn;
        # 指定重定向地址
        error_page   500 502 503 504  /50x.html;
        # 修改跳转响应
        error_page 404 =200 /50x.html;
        # 所有的404响应 通过匿名location 跳转location
        error_page 404 @jump_to_error;
	    location @jump_to_error {
		    default_type text/plain;
		    return 404 'Not Found Page...';
	    }

        location = /50x.html {
            root   html;
        }

    }
}
```

#### nginx + memcached
安装memcahced  

`yum -y install memcached`

默认配置文件在`/etc/sysconfig/memcached`

查看状态

```shell
memcached-tool 127.0.0.1:11211  stats
```

nginx模块文档 http://nginx.org/en/docs/http/ngx_http_memcached_module.html

```nginx
server {
    location / {
        set            $memcached_key "$uri?$args";
        # 连接memcached 
        memcached_pass localhost:11211;
        error_page     404 502 504 = @fallback;
    }

    location @fallback {
        proxy_pass     http://backend;
    }
}
```
- `$memcached_key "$uri?$args";`以uri和参数作为key保存到memcached中
  - `http://ss.com/index.html?id=100`的uri+args为`/index.html?id=100`
  - 将该资源保存后，再次访问命中缓存直接取出memcached中的数据  
  - 未命中直接进入` @fallback`中执行访问上游服务器 

#### nginx + redis
redis2-nginx-module是一个支持 Redis 2.0 协议的 Nginx upstream 模块，它可以让 Nginx 以非阻塞方式直接防问远方的 Redis 服务，同时支持 TCP 协议和 Unix Domain Socket 模式，并且可以启用强大的 Redis 连接池功能。

https://www.nginx.com/resources/wiki/modules/redis2/

https://github.com/openresty/redis2-nginx-module

> 此插件是openresty的redis插件模块，同时可以用于nginx

1. 获取插件安装包`https://github.com/openresty/redis2-nginx-module/archive/refs/tags/v0.15.tar.gz` 
2. 将redis插件编译入nginx `./configure --prefix=/usr/local/nginx/ --add-module=/root/redis2-nginx-module-0.15`

配置文件  

```nginx
# 测试接口
location = /foo {
    default_type text/html;
    # 带密码认证，无密码不需要认证
    redis2_query auth 123123;

    # 设置value变量值为first
    set $value 'first';
    # 设置 key为one的value(first)
    redis2_query set one $value;
    # redis连接信息
    redis2_pass 192.168.199.161:6379;

 }

# 赋值保存
 location = /get {

    default_type text/html;

    redis2_pass 192.168.199.161:6379;

    redis2_query auth 123123;
    # 转义url 防止乱码
    set_unescape_uri $key $arg_key;  #this requires ngx_set_misc

    redis2_query get $key;

}


# 获取 GET /set?key=one&val=first value

location = /set {

    default_type text/html;
    redis2_pass 192.168.199.161:6379;
    redis2_query auth 123123;
    # 获取key值(one)
    set_unescape_uri $key $arg_key;  # this requires ngx_set_misc
    # 获取val值(first value)
    set_unescape_uri $val $arg_val;  # this requires ngx_set_misc
    # 赋值 $key 
    redis2_query set $key $val;

 }


```

**其他操作详见GitHub文档**

# Stream模块
从1.9.0开始，NGINX增加了stream模块用来实现**四层协议**的转发、代理和负载均衡。与著名的四层LB软件lvs相比，stream 模块(开源版)无论从功能还是性能上，都有一定的差距，实现也相对简单。

> TCP/IP通常被认为是一个四层协议系统

文档地址 http://nginx.org/en/docs/stream/ngx_stream_core_module.html

在编译nginx需要手动开启模块，`./configure --prefix=/usr/local/nginx/ --with-stream`


## Stream模块为MySQL集群透明代理  
nginx可以使用Stream模块简易替换MyCat作为集群代理，同时可以通过 weight 保证保证访问比例实现数据灰度发布  

```nginx
stream{
    upstream mysqlurl{
        server 192.168.0.1:3306
        server 192.168.0.2:3306
    }
    server {

        listen 3306;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass mysqlurl;
    }
}

http{
    ...
}
```

# Nginx限流

## req模块(漏桶算法)
文档 http://nginx.org/en/docs/http/ngx_http_limit_req_module.html

`ngx_http_limit_req_module`模块用于限制请求速率，特别是来自单个IP地址的请求的处理速率。限制是使用**漏斗(漏桶)算法**完成的。

> 漏桶算法就是接收大量请求，但是是以固定速度流出，水桶满后多余请求被丢弃

```nginx
http {
   
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

    ...

    server {

        ...

        location /search/ {
            limit_req zone=one burst=5 nodelay;
        }
    }
}
```

**limit_req_zone**  
- `$binary_remote_addr`表示通过remote_addr这个标识来做限制，“binary_”的目的是缩写内存占用量，是限制同一客户端ip地址
- `zone=one:10m`表示生成一个大小为10M，名字为one的内存区域，用来存储访问的频次信息。
- `rate=1r/s`表示允许**相同标识的客户端**的访问频次，这里限制的是每秒1次，还可以有比如30r/m的。

**limit_req**   
- `zone=one`设置使用那个配置区域来做限制，上面的zone对应
- `burst=5`超量请求缓冲，设置一个大小为5的缓冲区，大量请求访问超出的访问可以先放到这个缓冲区
- `nodelay`如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回503(快速失败)，如果没有设置，则所有请求会等待排队

## 令牌桶算法
令牌桶算法，请求去令牌桶中申请令牌，得到令牌才能进行下一步访问，只需控制令牌产生的速率，即可限流请求，同时一个请求可以携带多个令牌来标识其特性（鸡毛信）


## Nginx的带宽限制

```nginx
http {
    ...
    server {
        ...
        location / {
            # 以1kb/s的速度传输数据
            limit_rate 1k;
            # 在请求了 10k 数据后开始限速 
            limit_rate_after 10k;
        }
    }
}
```

## 并发数限制  
计数器算法

```nginx
http {
   
    limit_conn_zone $binary_remote_addr zone=xianzhi:10m;
    ...
    server {
        ...
        location / {
            # 限制并发连接数为1 
            limit_conn zone=xianzhi 1;
        }
    }
}
```



# Nginx日志 

日志文档  http://nginx.org/en/docs/http/ngx_http_log_module.html

## empty_gif
empty_gif 是一个很不错的nginx 模块，可以方便的生成1*1 像素的图片（很适合数据分析，记录用户画像）  
也可以用作快速返回空白图片节约服务器资源   

文档 http://nginx.org/en/docs/http/ngx_http_empty_gif_module.html

```nginx
location / {
    empty_gif;
}

```


## access.log 
access_log用来定义日志级别，日志位置。语法如下：
日志级别： `debug > info > notice > warn > error > crit > alert > emerg`

`access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];` 

- `buffer=32k`日志缓存，累计到32kb就写入日志文件  
- `flush=10s`每隔10秒写入日志，强行写入，防止缓存不满无法保存日志  
- `gzip=9`开启gzip压缩的比例，压缩后的日志文件不可直接读，此时的文件名为log结尾但是是gzip格式，所以需要直接修改格式用gzip解压缩 


```
http{
    # 积攒到32k就写入日志文件 
    access_log /spool/logs/nginx-access.log test buffer=32k gzip=9 flush=10s;

    log_format test '$remote_addr - $remote_user [$time_local] '
                       '"$request" $status $bytes_sent '
                       '"$http_referer" "$http_user_agent" "$gzip_ratio"';
}
```

**json格式的log配置**  
使用json格式文档可以直接导入相关数据库，方便日志的迁移

```nginx
http{
     log_format jsonlog escape=json '{ "timestamp": "$time_iso8601", '
                     '"remote_addr": "$remote_addr", '
                     '"body_bytes_sent": $body_bytes_sent, '
                     '"request_time": $request_time, '
                     '"response_status": $status, '
                     '"request": "$request", '
                     '"request_method": "$request_method", '
                     '"host": "$host",'
                     '"content":"$content",'
                     '"upstream_cache_status": "$upstream_cache_status",'
                     '"upstream_addr": "$upstream_addr",'
                     '"http_x_forwarded_for": "$http_x_forwarded_for",'
                     '"http_referrer": "$http_referer", '
                     '"http_user_agent": "$http_user_agent" }';
}
```

## error.log

文档  http://nginx.org/en/docs/ngx_core_module.html#error_log

```nginx
error_log /var/log/nginx-error.log info;
```
# Nginx上游服务器健康检查
- 重试机制
- 状态检查

## upstream被动重试机制

http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream

- `max_fails`最大失败次数0为标记一直可用，不检查健康状态
- `fail_timeout`失败时间当fail_timeout时间内失败了max_fails次，标记服务不可用,fail_timeout时间后会再次尝试激活服务

- `proxy_next_upstream error|timeout|...`发生那些错误尝试下一个上游服务
- `proxy_next_upstream_timeout 15s`超时时长，判定是否超时，重试最大超时时间
- `proxy_next_upstream_tries 5` 重试次数，包括第一次
  - proxy_next_upstream_timeout时间内允许proxy_next_upstream_tries次重试

```nginx
htp{
    upstream forntend {
        # 10s内失败两次 标记为不可用，10s后再次尝试
        server 192.168.0.1 max_fails=2 fail_timeout=10s;
        server 192.168.0.1 max_fails=2 fail_timeout=10s;
    }
}
```

## 主动健康检查

tengine版

https://github.com/yaoweibin/nginx_upstream_check_module

nginx商业版

http://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html


### Tengine的主动健康检查模块
1. 下载GitHub上的模块
2. 下载仓库中支持补丁`.patch`文件的nginx版本,并解压，进入到解压目录
3. 安装patch`yum -y install patch`
4. 执行`patch -p1 < ../nginx_upstream_check_module-master/check_1.20.1+.patch` 
5. 下载nginx插件,并解压 https://github.com/yaoweibin/nginx_upstream_check_module/releases/tag/v0.4.0 
6. 进行编译`./configure --prefix=/usr/local/nginx --add-module=/root/nginx_upstream_check_module`


```nginx
http{
    upstream backend {
    
    server 192.168.44.104:8080;
    server 192.168.44.105:8080;

    check interval=3000 rise=2 fall=5 timeout=1000 type=http;
       check_http_send "HEAD / HTTP/1.0\r\n\r\n";
       check_http_expect_alive http_2xx http_3xx;
    }
    location /status {
        check_status;
        access_log off;
    }
    location / {
        proxy_pass http://backend;
    	root   html;
    }
}

```

- 访问`/status`可以查看上游服务器状态

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nginx的tengine模块主动健康检查.png)

- `interval=3000`每隔3000ms检查一次
- `rise=2` 两次成功标记为正常(上线)
- `fall=5` 5次失败标记为异常(下线)
- `type=http`采用http协议

# GEOIP
官方网站 https://www.maxmind.com/en/home 


GeoIP基本上是一个付费数据库，能够将IP地址映射到各自的位置，包括国家/地区，ISP和城市。简言之就是尽量保证访问的服务是离客户端最近的服务   
GeoLite2是GeoIP2的免费版本，与GeoIP2数据库相比准确性较差。GeoLite2数据库每周更新国家、城市和自治系统编号信息，更新时间为每周二。


## 安装依赖

https://github.com/maxmind/libmaxminddb

下载后执行编译安装之后

```shell
$ echo /usr/local/lib  >> /etc/ld.so.conf.d/local.conf 
$ ldconfig
```

## Nginx模块

https://github.com/leev/ngx_http_geoip2_module

更完整的配置可参考官方文档

http://nginx.org/en/docs/http/ngx_http_geoip_module.html#geoip_proxy



## Nginx配置

```nginx
    geoip2 /root/GeoLite2-ASN_20220524/GeoLite2-ASN.mmdb {
        $geoip2_country_code country iso_code;
    }
    add_header country $geoip2_country_code;
```

# Nginx 二次开发

## Lua Nginx Openresty

### Lua
Lua 是一种轻量、小巧的脚本语言，用标准 C 语言编写，其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

> Luagit 是lua的开发环境，类比JDK

官网：http://www.lua.org/

- 教程
  - https://www.runoob.com/lua/lua-environment.html

- IDEA EmmyLua插件
  - https://github.com/EmmyLua/IntelliJ-EmmyLua
  - https://emmylua.github.io/zh_CN/

- LDT 基于eclipse
  - https://www.eclipse.org/ldt/


### Openresty

#### 下载安装

http://openresty.org/cn/download.html

最小版本基于nginx1.21

`./configure`

然后在进入 `openresty-VERSION/ `目录, 然后输入以下命令配置:

 `./configure`

默认`--prefix=/usr/local/openresty` 程序会被安装到`/usr/local/openresty`目录。

依赖 `gcc openssl-devel pcre-devel zlib-devel`

安装：`yum install gcc openssl-devel pcre-devel zlib-devel postgresql-devel`

 

您可以指定各种选项，比如

```
./configure --prefix=/opt/openresty \

            --with-luajit \

            --without-http_redis2_module \

            --with-http_iconv_module \

            --with-http_postgres_module
```

试着使用 `./configure --help` 查看更多的选项。

`make && make install`


#### 启动运行

```shell
/usr/local/openresty/nginx/sbin/nginx -c /usr/local/openresty/nginx/conf/nginx.conf

```

**启动，重启等命令和nginx相同**

#### lua脚本

```nginx
# 在Nginx.conf 中写入
   location /lua {

        default_type text/html;
        # 执行lua代码
        content_by_lua '
           ngx.say("<p>Hello, World!</p>")
         ';
      }
```

访问`/lua` 可以看到页面上的Hello, World!


```nginx
# 引入外置lua脚本文件，代码解耦
server {
    listen       80;
    server_name  localhost;

    location /lua1 {
        default_type text/html;
        content_by_lua_file lua/hello.lua;
    }
}
```

Lua文件位置 `/usr/local/openresty/nginx/lua/hello.lua`   
每次改变脚本都需要重启生效，可以添加配置`lua_code_cache off;`

- `set_by_lua`修改nginx变量
- `rewrite_by_lua`修改uri
- `access_by_lua`访问控制
- `header_filter_by_lua`修改响应头
- `boy_filter_by_lua`修改响应体
- `log_by_lua`日志



#### 获取系统变量及参数

```lua
local uri_args = ngx.req.get_uri_args()  

for k, v in pairs(uri_args) do  

    if type(v) == "table" then  

        ngx.say(k, " : ", table.concat(v, ", "), "<br/>")  

    else  

        ngx.say(k, ": ", v, "<br/>")  

    end  
end
```

针对url`http://localhost/lua1?kill=1&fk=12`的结果为

```
kill: 1
fk: 12
```
   


#### 获取Nginx请求头信息

```lua
local headers = ngx.req.get_headers()                         

ngx.say("Host : ", headers["Host"], "<br/>")  

ngx.say("user-agent : ", headers["user-agent"], "<br/>")  

ngx.say("user-agent : ", headers.user_agent, "<br/>")

for k,v in pairs(headers) do  

    if type(v) == "table" then  

        ngx.say(k, " : ", table.concat(v, ","), "<br/>")  

    else  

        ngx.say(k, " : ", v, "<br/>")  

    end  

end  
```

可以得到请求头信息

```
Host : 121.89.244.48
user-agent : Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36
user-agent : Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36
accept-encoding : gzip, deflate
upgrade-insecure-requests : 1
accept-language : zh-CN,zh;q=0.9
host : 121.89.244.48
cache-control : max-age=0
accept : text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
user-agent : Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36
```

#### 获取post请求参数

```lua
ngx.req.read_body()  

ngx.say("post args begin", "<br/>")  

local post_args = ngx.req.get_post_args()  

for k, v in pairs(post_args) do  

    if type(v) == "table" then  

        ngx.say(k, " : ", table.concat(v, ", "), "<br/>")  

    else  

        ngx.say(k, ": ", v, "<br/>")  

    end  
end
```

#### http协议版本

```lua
ngx.say("ngx.req.http_version : ", ngx.req.http_version(), "<br/>")
```

#### 请求方法

```lua
ngx.say("ngx.req.get_method : ", ngx.req.get_method(), "<br/>")  
```

#### 原始的请求头内容  

```lua
ngx.say("ngx.req.raw_header : ",  ngx.req.raw_header(), "<br/>")  
```

#### body内容体  

```lua
ngx.say("ngx.req.get_body_data() : ", ngx.req.get_body_data(), "<br/>")
```


### lua实现缓存

简单的使用`lua_shared_dict`设置一块共享内存区域,可以被各个worker共享写在http模块中，并且保证原子性
`lua_shared_dict shared_data 1m;`申请1m内存作为缓存

```nginx
http{
    ...
    lua_shared_dict shared_data 1m;

}
```

```lua
local shared_data = ngx.shared.shared_data  
local i = shared_data:get("i")  

if not i then  

    i = 1  

    shared_data:set("i", i)  

    ngx.say("lazy set i ", i, "<br/>")  
end  
 

i = shared_data:incr("i", 1)  

ngx.say("i=", i, "<br/>")
```

`lua-resty-lrucache`Lua 实现的一个简单的 LRU 缓存，适合在 Lua 空间里直接缓存较为复杂的 Lua 数据结构：它相比 ngx_lua 共享内存字典可以省去较昂贵的序列化操作，相比 memcached 这样的外部服务又能省去较昂贵的 socket 操作

https://github.com/openresty/lua-resty-lrucache

引用lua文件

```nginx
location / {
    content_by_lua_block {
        require("my/cache").go()
    }
}
```

> 关于`my/cache`文件位置的说明，首先配置这个location后进行访问，查看error.log可以找到默认位置有下面几个文件夹,可以选择任意文件夹存放lua脚本

```
2023/01/31 17:33:33 [error] 3902#0: *21 lua entry thread aborted: runtime error: content_by_lua(nginx.conf:62):2: module 'my/cache' not found:
        no field package.preload['my/cache']
        no file '/usr/local/openresty/site/lualib/my/cache.ljbc'
        no file '/usr/local/openresty/site/lualib/my/cache/init.ljbc'
        no file '/usr/local/openresty/lualib/my/cache.ljbc'
        no file '/usr/local/openresty/lualib/my/cache/init.ljbc'
        no file '/usr/local/openresty/site/lualib/my/cache.lua'
        no file '/usr/local/openresty/site/lualib/my/cache/init.lua'
        no file '/usr/local/openresty/lualib/my/cache.lua'
        no file '/usr/local/openresty/lualib/my/cache/init.lua'
        no file './my/cache.lua'
        no file '/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/my/cache.lua'
        no file '/usr/local/share/lua/5.1/my/cache.lua'
        no file '/usr/local/share/lua/5.1/my/cache/init.lua'
        no file '/usr/local/openresty/luajit/share/lua/5.1/my/cache.lua'
        no file '/usr/local/openresty/luajit/share/lua/5.1/my/cache/init.lua'
        no file '/usr/local/openresty/site/lualib/my/cache.so'
        no file '/usr/local/openresty/lualib/my/cache.so'
        no file './my/cache.so'
        no file '/usr/local/lib/lua/5.1/my/cache.so'
        no file '/usr/local/openresty/luajit/lib/lua/5.1/my/cache.so'
        no file '/usr/local/lib/lua/5.1/loadall.so'
stack traceback
```

> 使用配置项`lua_package_path`指定文件路径（内容来自官方GitHub仓库）

```nginx
http {
        # only if not using an official OpenResty release
        lua_package_path "/path/to/lua-resty-lrucache/lib/?.lua;;";
        ...
    }
```

```lua
local _M = {}

lrucache = require "resty.lrucache"

c, err = lrucache.new(200)  -- allow up to 200 items in the cache
ngx.say("count=init")

if not c then
    error("failed to create the cache: " .. (err or "unknown"))
end

function _M.go()

count = c:get("count")

c:set("count",100)
ngx.say("count=", count, " --<br/>")

if not count then  
    c:set("count",1)

    ngx.say("lazy set count ", c:get("count"), "<br/>")  

else
c:set("count",count+1)
 
ngx.say("count=", count, "<br/>")
end

end
return _M
```



### lua-resty-redis访问redis

https://github.com/openresty/lua-resty-redis

#### 常用方法

```lua
local res, err = red:get("key")

local res, err = red:lrange("nokey", 0, 1)

ngx.say("res:",cjson.encode(res))
```



#### 创建连接

```lua
red, err = redis:new()

ok, err = red:connect(host, port, options_table?)
```

#### timeout

```lua
red:set_timeout(time)
```

#### keepalive

```lua
red:set_keepalive(max_idle_timeout, pool_size)
```

#### close

```
ok, err = red:close()
```

 

#### pipeline

```nginx
red:init_pipeline()

results, err = red:commit_pipeline()
```

#### 认证

```nginx
    local res, err = red:auth("foobared")

    if not res then

        ngx.say("failed to authenticate: ", err)

        return
end
```

```lua
  local redis = require "resty.redis"
                local red = redis:new()

                red:set_timeouts(1000, 1000, 1000) -- 1 sec

  local ok, err = red:connect("127.0.0.1", 6379)
 if not ok then
                    ngx.say("failed to connect: ", err)
                    return
                end

                ok, err = red:set("dog", "an animal")
                if not ok then
                    ngx.say("failed to set dog: ", err)
                    return
                end

                ngx.say("set result: ", ok)

                local res, err = red:get("dog")
                if not res then
                    ngx.say("failed to get dog: ", err)
                    return
                end

                if res == ngx.null then
                    ngx.say("dog not found.")
                    return
                end


              ngx.say("dog: ", res)
```
#### redis-cluster支持

https://github.com/steve0511/resty-redis-cluster


### lua-resty-mysql

 https://github.com/openresty/lua-resty-mysql

 ```lua
 local mysql = require "resty.mysql"
                 local db, err = mysql:new()
                 if not db then
                     ngx.say("failed to instantiate mysql: ", err)
                     return
                 end
 
                 db:set_timeout(1000) -- 1 sec
 
 
                 local ok, err, errcode, sqlstate = db:connect{
                     host = "192.168.44.211",
                     port = 3306,
                     database = "zhangmen",
                     user = "root",
                     password = "111111",
                     charset = "utf8",
                     max_packet_size = 1024 * 1024,
                 }
 
 
                 ngx.say("connected to mysql.<br>")
 
 
 
                 local res, err, errcode, sqlstate = db:query("drop table if exists cats")
                 if not res then
                     ngx.say("bad result: ", err, ": ", errcode, ": ", sqlstate, ".")
                     return
                 end
 
 
                 res, err, errcode, sqlstate =
                     db:query("create table cats "
                              .. "(id serial primary key, "
                              .. "name varchar(5))")
                 if not res then
                     ngx.say("bad result: ", err, ": ", errcode, ": ", sqlstate, ".")
                     return
                 end
 
                 ngx.say("table cats created.")
 
 
 
                 res, err, errcode, sqlstate =
                     db:query("select * from t_emp")
                 if not res then
                     ngx.say("bad result: ", err, ": ", errcode, ": ", sqlstate, ".")
                     return
                 end
 
                 local cjson = require "cjson"
                 ngx.say("result: ", cjson.encode(res))
 
 
                 local ok, err = db:set_keepalive(10000, 100)
                 if not ok then
                     ngx.say("failed to set keepalive: ", err)
                     return
                 end
  
 
 ```


## Lua 开源项目

### WAF

https://github.com/unixhot/waf

https://github.com/loveshell/ngx_lua_waf

- 防止 SQL 注入，本地包含，部分溢出，fuzzing 测试，XSS/SSRF 等 Web 攻击
- 防止 Apache Bench 之类压力测试工具的攻击
- 屏蔽常见的扫描黑客工具，扫描器
- 屏蔽图片附件类目录执行权限、防止 webshell 上传
- 支持 IP 白名单和黑名单功能，直接将黑名单的 IP 访问拒绝
- 支持 URL 白名单，将不需要过滤的 URL 进行定义
- 支持 User-Agent 的过滤、支持 CC 攻击防护、限制单个 URL 指定时间的访问次数
- 支持支持 Cookie 过滤，URL 与 URL 参数过滤
- 支持日志记录，将所有拒绝的操作，记录到日志中去

### Kong 基于Openresty的流量网关

https://konghq.com/

https://github.com/kong/kong

Kong 基于 OpenResty，是一个云原生、快速、可扩展、分布式的微服务抽象层（Microservice Abstraction Layer），也叫 API 网关（API Gateway），在 Service Mesh 里也叫 API 中间件（API Middleware）。

Kong 开源于 2015 年，核心价值在于高性能和扩展性。从全球 5000 强的组织统计数据来看，Kong 是现在依然在维护的，在生产环境使用最广泛的 API 网关。

Kong 宣称自己是世界上最流行的开源微服务 API 网关（The World’s Most Popular Open Source Microservice API Gateway）。

核心优势：

- 可扩展：可以方便的通过添加节点水平扩展，这意味着可以在很低的延迟下支持很大的系统负载。

- 模块化：可以通过添加新的插件来扩展 Kong 的能力，这些插件可以通过 RESTful Admin API 来安装和配置。

- 在任何基础架构上运行：Kong 可以在任何地方都能运行，比如在云或混合环境中部署 Kong，单个或全球的数据中心。

###  APISIX

### ABTestingGateway

https://github.com/CNSRE/ABTestingGateway

ABTestingGateway 是一个可以动态设置分流策略的网关，关注与灰度发布相关领域，基于 Nginx 和 ngx-lua 开发，使用 Redis 作为分流策略数据库，可以实现动态调度功能。

ABTestingGateway 是新浪微博内部的动态路由系统 dygateway 的一部分，目前已经开源。在以往的基于 Nginx 实现的灰度系统中，分流逻辑往往通过 rewrite 阶段的 if 和 rewrite 指令等实现，优点是性能较高，缺点是功能受限、容易出错，以及转发规则固定，只能静态分流。ABTestingGateway 则采用 ngx-lua，通过启用 lua-shared-dict 和 lua-resty-lock 作为系统缓存和缓存锁，系统获得了较为接近原生 Nginx 转发的性能。

- 支持多种分流方式，目前包括 iprange、uidrange、uid 尾数和指定uid分流

- 支持多级分流，动态设置分流策略，即时生效，无需重启

- 可扩展性，提供了开发框架，开发者可以灵活添加新的分流方式，实现二次开发

- 高性能，压测数据接近原生 Nginx 转发

- 灰度系统配置写在 Nginx 配置文件中，方便管理员配置

- 适用于多种场景：灰度发布、AB 测试和负载均衡等




































# 参考资料
> - [尚硅谷视频](https://www.bilibili.com/video/BV1yS4y1N76R)
> - [强制缓存和协商缓存](https://blog.csdn.net/m0_63657524/article/details/125966386)
> - [nginx限流](https://www.cnblogs.com/biglittleant/p/8979915.html)
> - [nginx日志](https://www.cnblogs.com/biglittleant/p/8979856.html)