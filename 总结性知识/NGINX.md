# NGINX

### nginx自带变量

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211027135603.png)

### proxy 代理

```nginx
正向代理中代理的对象是客户端，proxy和client同属一个LAN，对server透明；
反向代理中代理的对象是服务端，proxy和server同属一个LAN，对client透明。

proxy具体配置详解
proxy_pass ：真实服务器的地址，可以是ip也可以是域名和url地址
proxy_redirect ：如果真实服务器使用的是的真实IP:非默认端口。则改成IP：默认端口。
proxy_set_header：重新定义或者添加发往后端服务器的请求头
proxy_set_header X-Real-IP ：启用客户端真实地址（否则日志中显示的是代理在访问网站）
proxy_set_header X-Forwarded-For：记录代理地址

proxy_connect_timeout：:后端服务器连接的超时时间发起三次握手等候响应超时时间
proxy_send_timeout：后端服务器数据回传时间就是在规定时间之内后端服务器必须传完所有的数据
proxy_read_timeout ：nginx接收upstream（上游/真实） server数据超时, 默认60s, 如果连续的60s内没有收到1个字节, 连接关闭。像长连接

proxy_buffering on;开启缓存
proxy_buffer_size：proxy_buffer_size只是响应头的缓冲区
proxy_buffers 4 128k; 内容缓冲区域大小
proxy_busy_buffers_size 256k; 从proxy_buffers划出一部分缓冲区来专门向客户端传送数据的地方
proxy_max_temp_file_size 256k;超大的响应头存储成文件。
```

### upstream  负载均衡

```nginx
upstream testapp { 
      server 10.0.105.199:8081;
      server 10.0.105.202:8081;
    }
 server {
        ....
        location / {         
           proxy_pass  http://testapp;  #请求转向 testapp 定义的服务器列表         
        } 
    
upstream 支持4种负载均衡调度算法:
A、轮询(默认):每个请求按时间顺序逐一分配到不同的后端服务器。
B、ip_hash:每个请求按访问IP的hash结果分配，同一个IP客户端固定访问一个后端服务器。可以保证来自同一ip的请求被打到固定的机器上，可以解决session问题。
C、url_hash:按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器。                            
D、fair:这是比上面两个更加智能的负载均衡算法。按后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持 fair的，如果需要使用这种调度算法，必须下载Nginx的 upstream_fair模块。


```

### 会话保持

```nginx 
1、ip_hash
    ip_hash使用源地址哈希算法，将同一客户端的请求总是发往同一个后端服务器，除非该服务器不可用。
    
 ip_hash语法：   

upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
}    

注意：
ip_hash简单易用，但有如下问题： 
当后端服务器宕机后，session会丢失； 
来自同一局域网的客户端会被转发到同一个后端服务器，可能导致负载失衡；   

2.sticky_cookie_insert
        使用sticky_cookie_insert启用会话亲缘关系，这会导致来自同一客户端的请求被传递到一组服务器的同一台服务器。与ip_hash不同之处在于，它不是基于IP来判断客户端的，而是基于cookie来判断。因此可以避免上述ip_hash中来自同一局域网的客户端和前段代理导致负载失衡的情况。(需要引入第三方模块才能实现sticky)
 下载地址：https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/get/08a395c66e42.zip
  下载后进行编译安装
语法：
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    sticky_cookie_insert srv_id expires=1h domain=3evip.cn path=/;
}
expires：设置浏览器中保持cookie的时间 
domain：定义cookie的域 
path：为cookie定义路径
```

###  动静分离

```nginx
server {
        listen      80;
        server_name     localhost
        #动态资源加载
        location ~ \.(php|jsp)$ {
            proxy_pass http://phpserver;
            proxy_set_header Host $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                }
        #静态资源加载
        location ~ .*\.(html|gif|jpg|png|bmp|swf|css|js)$ {
            proxy_pass http://static;
            proxy_set_header Host $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                }
        }
```

### 防盗链

```nginx
1、nginx 防止网站资源被盗用模块
ngx_http_referer_module

2. 防盗链配置
[root@nginx-server ~]# vim /etc/nginx/nginx.conf
# 日志格式添加"$http_referer"
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

[root@nginx-server html]# vim /etc/nginx/conf.d/nginx.conf
server {
    listen       80;
    server_name  localhost;

    location ~  .*\.(gif|jpg|png|jpeg)$ {
         root  /usr/share/nginx/html;

         valid_referers none blocked *.qf.com 192.168.1.10;
                if ($invalid_referer) {
                   return 403;
                }
        }
}
• none : 允许没有http_refer的请求访问资源；
• blocked : 允许不是http://开头的，不带协议的请求访问资源---被防火墙过滤掉的；
• server_names : 只允许指定ip/域名来的请求访问资源（白名单）；
```

###  地址重写

```nginx
1、什么是Rewrite
        Rewrite对称URL Rewrite，即URL重写，就是把传入Web的请求重定向到其他URL的过程。
应用环境
        server，location

语法：
if (condition) { … }
if 可以支持如下条件判断匹配符号
~ 和	!~				用来判断是否正则匹配 (区分大小写)
~* 和 !~*		    用来判断是否正则匹配 (不区分大小写)
-f 和!-f 		    用来判断是否存在文件
-d 和!-d 		    用来判断是否存在目录
-e 和!-e 		    用来判断是否存在文件或目录
-x 和!-x 		    用来判断文件是否可执行
在匹配过程中可以引用一些Nginx的全局变量
$args				请求中的参数;
$document_root	    针对当前请求的根路径设置值;
$host				请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名;
$limit_rate			对连接速率的限制;
$request_method		请求的方法，比如"GET"、"POST"等;
$remote_addr		客户端地址;
$remote_port		客户端端口号;
$remote_user		客户端用户名，认证用;
$request_filename   当前请求的文件路径名（带网站的主目录/usr/local/nginx/html/images /a.jpg）
$request_uri		当前请求的文件路径名（不带网站的主目录/images/a.jpg）
$query_string		与$args相同;
$scheme				用的协议，比如http或者是https
$server_protocol	请求的协议版本，"HTTP/1.0"或"HTTP/1.1";
$server_addr 		服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费);
$server_name		请求到达的服务器名;
$document_uri 		与$uri一样，URI地址;
$server_port 		请求到达的服务器端口号;
```

