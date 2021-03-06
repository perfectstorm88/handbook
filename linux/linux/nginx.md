# 安装部署和环境

## 源文件编译安装
其实源文件安装很简单，参照官方文档，写得非常详细，可以不必看那么细，只关注下面的执行语句：
http://nginx.org/en/docs/configure.html

```shell
# 下载文件
wget http://nginx.org/download/nginx-1.9.14.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.zip

tar zxvf nginx-1.9.14.tar.gz
unzip pcre-8.38.zip

cd ngix*
./configure  \
    --sbin-path=/usr/local/nginx/nginx \
    --conf-path=/usr/local/nginx/nginx.conf \
    --pid-path=/usr/local/nginx/nginx.pid \
    --with-pcre=../pcre-8.38    
make

curl localhost #进行测试
```
在源文件目录执行
./configure  \
    --sbin-path=/usr/local/nginx/nginx \
    --conf-path=/usr/local/nginx/nginx.conf \
    --pid-path=/usr/local/nginx/nginx.pid \
    --with-pcre=../pcre/8.38    

    --without-http_rewrite_module
    --with-http_ssl_module \
说明1：如果要支持路径的正则表达式解析，需要PCRE(Perl Compatible Regular Expressions)库，
with-pcre是pcre的源码目录,可以直接从官网上下载http://pcre.org/
说明2: 执行./configure 成功后，执行make 默认是编译，执行make install才是安装。
## 源码修改样例：时间格式
可以参见http://blog.csdn.net/chlaws/article/details/8650218

这篇文章的有个错误，比如“打开nginx_http_log_module.c”，应该是“ngx_http_log_module.c”


## Mac安装和环境

[Installing Nginx in Mac OS X Maverick With Homebrew](https://coderwall.com/p/dgwwuq/installing-nginx-in-mac-os-x-maverick-with-homebrew)

/usr/local/etc/nginx/nginx.conf

## 3 Ubuntu环境说明

配置文件：/etc/nginx 

# 用法
## 多个服务器分文件配置
nginx.conf是主配置文件
包含： include /etc/nginx/sites-enabled/*;
```
root@iZ23hixi2u3Z:/etc/nginx/sites-enabled# cat jfjun
server {
    listen 80; 
    server_name www.jfjun.com;
    location / { 
        proxy_pass http://www.jfjun.com:6668/;
        proxy_set_header Host $host:80;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Via "nginx";
    }   
}
root@iZ23hixi2u3Z:/etc/nginx/sites-enabled# cat  jfjun.com
server {
    listen 80; 
    server_name jfjun.com;
    location / { 
        proxy_pass http://jfjun.com:6668/;
        proxy_set_header Host $host:80;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Via "nginx";
    }   
}
```

日志：/var/log/nginx

## Nginx静态文件
默认放置目录：/usr/share/nginx/www
这个目录配置在，/etc/nginx/sites-enabled的default文件中

## 多目录配置
参见：http://nginx.org/en/docs/beginners_guide.html 
的Serving Static Content章节
```
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```
If there are several matching location blocks nginx selects the one with the longest prefix. The location block above provides the shortest prefix, of length one, and so only if all other location blocks fail to provide a match, this （‘/’）block will be used
如果有多个location，先配置最长的前缀，如果都找不到，再到‘/’根目录
## 把一个目录指向另一个服务器
[Passing a Request to a Proxied Server](https://www.nginx.com/resources/admin-guide/reverse-proxy/)
```
location /some/path/ {
    proxy_pass http://www.example.com/link/;
}
```

the address of the proxied server is followed by a URI, /link/. If the URI is specified along with the address, it replaces the part of the request URI that matches the location parameter. For example, here the request with the /some/path/page.html URI will be proxied to http://www.example.com/link/page.html. If the address is specified without a URI, or it is not possible to determine the part of URI to be replaced, the full request URI is passed (possibly, modified).
代理服务器的地址如果后面带有URI，将会替换请求地址。如/some/path/page.html 被代理为
http://www.example.com/link/page.html

## Nigin的location
[location](http://nginx.org/en/docs/http/ngx_http_core_module.html?&_ga=1.197268911.1815848908.1446455851#location)
https://www.nginx.com/resources/admin-guide/nginx-web-server/

如何配置 NGINX Plus作为一个web服务器，

To make the configuration easier to maintain, we recommend that you split it into a set of feature-specific files stored in the /etc/nginx/conf.d directory and use the include directive in the main nginx.conf file to reference the contents of the feature-specific files.
为了便于维护，把一组特性拆分到不同文件中。通过include进行引用
```
include conf.d/http;
include conf.d/stream;
include conf.d/exchange-enhanced;
```

When NGINX Plus processes a request, it first selects the virtual server that will serve the request.
首选选择虚拟服务器，
A virtual server is defined by a server directive in the http context, for example:
http上下文，一个虚拟服务就是定义一个server指令
```
http {
    server {
        # Server configuration
    }
}
```

## Nigin的负载均衡upstream
```
upstream jfjun {
    server localhost:8000;
    server localhost:8001;
}
server {
    listen 80;
    server_name 192.168.2.108;
    location / {
        proxy_pass http://jfjun;
        proxy_set_header  X-Real-IP  $remote_addr;
    }
}
```


阅读知识：[HTTP 请求头中的 X-Forwarded-For](https://imququ.com/post/x-forwarded-for-header-in-http.html)
 Nginx 配置中的这两行起作用了，为请求额外增加了两个自定义头：
```
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```
Nginx 会在 X-Forwarded-For 后追加client的 IP；并用client的 IP 覆盖 X-Real-IP 请求头。这说明，有了 Nginx 的加工，X-Forwarded-For 最后一节以及 X-Real-IP 整个内容无法构造，可以用于获取用户 IP

X-Forwarded-For 是一个 HTTP 扩展头部。HTTP/1.1（RFC 2616）协议并没有对它的定义，它最开始是由 Squid 这个缓存代理软件引入，用来表示 HTTP 请求端真实 IP。如今它已经成为事实上的标准，被各大 HTTP 代理、负载均衡等转发服务广泛使用，并被写入 RFC 7239（Forwarded HTTP Extension）标准之中。
X-Forwarded-For 请求头格式非常简单，就这样：
```
X-Forwarded-For: client, proxy1, proxy2
```
## Nginx做密码登录
```
server {
    listen 15002;
    server_name 114.55.229.224;
    location ~ .* {
        proxy_pass http://127.0.0.1:5002;
        proxy_set_header Host $host;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        auth_basic "Restricted Content";
        auth_basic_user_file /etc/nginx/.httpasswd;
    }
}
```


/etc/nginx/.httpasswd 是通过htpasswd创建出来的,如
```
htpasswd -c xxxx abc abcd
```
## Nginx控制http超时时间

- proxy_read_timeout 是等待后端返回页面的时间，默认60s
- **keepalive_timeout**

可以控制某个api的超时
```
     location /api/rm/invoice/uploadInvoiceDetail {
         proxy_pass http://rm;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_read_timeout 180s;
     }
```
# 好工具
## ngxtop
https://github.com/lebinh/ngxtop
# 问题
## 上传失败
nginx有大小限制
配置 client_max_body_size 20m;
如
```
server {
    listen 8080;
    #server_name kj.jfjun.com;
    client_max_body_size 20m;
    location / {
        root   /root/project/jfjun-fp1/public;
        index  index.html index.htm;
    }
```
## 502错误
上游服务器挂掉
## 504错误
响应超时
