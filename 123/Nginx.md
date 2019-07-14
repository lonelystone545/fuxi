

[TOC]



nginx是开源的、可靠的、高性能的http中间件服务，代理服务

# 高性能的原因

1. IO多路复用epoll（多个描述符的IO操作都能在一个线程内并发交替地顺序完成，这就是IO多路复用，复用指的是复用同一个线程）

   IO多路复用实现方式有select、poll和epoll

   select缺点：能够监视文件描述符地数量存在最大限制（最大1024）；线性扫描效率低下；

   epoll：当FD就绪后，采用系统回调函数的方式将FD放入就绪列表中，效率更高；无最大连接数限制；

2. 轻量级

   功能模块少，代码模块化

3. CPU亲和

   是一种把CPU核心和Nginx工作进程绑定方式，把Nginx的每个worker进行固定在一个cpu上执行，减少切换cpu的cache miss，获得更好的性能。

4. sendfile工作机制--处理静态文件

   对于原来的http server来说，在请求一个文件时，需要经过操作系统的内核空间和用户空间，最终到达socket，响应给用户；而对于静态文件来说，是不需要经过用户空间的处理，直接在内核空间进行传输即可；sendfile采用的就是这种 零拷贝 的机制，直接在内核空间传输文件，到达socket，响应给用户。

# 安装目录详解

| 路径                                                         | 类型           | 作用                                                         |
| ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
| /etc/logrotate.d/nginx                                       | 配置文件       | nginx日志轮转，使用logrotate服务进行日志切割                 |
| /etc/nginx<br />/etx/nginx/nginx.conf<br />/etc/nginx/conf.d<br />/etc/nginx/conf.d/default.conf | 目录、配置文件 | 主要配置，nginx启动时读取nginx.conf文件，在配置没有变更时，会读取default.conf文件 |
|                                                              |                | cgi相关配置（主要是用于php）                                 |
| /etc/nginx/mime.types                                        | 配置文件       | 设置http协议的content-type与扩展名对应关系                   |
| /usr/lib64/nginx/modules<br />/etc/nginx/modules             |                | nginx模块目录                                                |
| /usr/sbin/nginx<br />/usr/sbin/nginx-debug                   |                | nginx命令                                                    |
| /var/cache/nginx                                             |                | nginx缓存目录                                                |
| /var/log/nginx                                               |                | nginx日志目录                                                |

# nginx常用命令

1. nginx -V 查询版本，编译的参数

   ![1550976928007](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550976928007.png)

   ![1550976946112](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550976946112.png)

   ![1550976972030](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550976972030.png)

   ![1550977003151](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550977003151.png)

   ![1550977019773](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550977019773.png)

   2. start nginx

   3. nginx reload 重载配置文件并启动nginx

   4. nginx -s stop 停止nginx

   5. nginx -tc nginx.conf 检查语法是否正确

      

# Nginx配置详解

   ## 日志详解

   1. error.log 记录nginx处理http请求的错误，以及nginx本身运行的错误数据

   2. access.log 记录每次http请求的状态，分析用户的行为和交互；

      依赖log_format实现的，nginx会记录很多请求的信息，这些信息相当于nginx的变量，log_format将这些变量组合起来记录到日志中。

      ![1550979306819](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550979306819.png)

      nginx变量：

      ​	http请求的变量： -arg_PARAMETER、 http_HEADER、sent_http_HEADER

      ​	内置变量：参考[官网配置](http://nginx.org/en/docs/syslog.html)

      ​	自定义变量：- 自己定义的

## 配置

1. server配置{ }

listen 80: 监听80端口

server_name www.baidu.com: 配置域名，通过域名访问

```xm
location / {
root:/usr/share/nginx/html   访问的路径
index:index.html index.htm 路径下具体的资源}
error_page   500 502 503 504  /50x.html; 当返回这些错误码时，进行重定向至静态页50x.html
location = /50x.html {
root   html;
}
```

# Nginx模块

## Nginx官方模块

通过nginx -V可以查看编译时关联的模块,--with-

### --with-http_stub_status_module  

  显示Nginx处理客户端连接的状态，用于监控Nginx当前连接的信息；

语法：stub_status
 上下文：server/location

 ​         case：

```xml
    localtion /myStatus {
		stub_status;
	}
```

### --with-http_random_inde_module

在目录中随机选择一个文件作为主页

语法：random_index on | off; 默认是 random_index off;

 上下文：location

 case:

```xml
localtion / {
     root:/usr/share/nginx/html;
     random_index on;
 }
```

### --with-http_sub_module

替换nginx服务端返回给客户端的内容

应用场景：当nginx作为http服务器时，有多个虚拟主机时，需要对每台主机返回的内容进行替换；对返回的敏感内容进行替换，简捷迅速；临时增加通用的js或者css文件；

语法：sub_filter string replacement; 将string替换为replacement

 	   sub_filter_last_modified on|off; 是否开启缓存，默认是off

 	   sub_filter_once on|off;是否匹配所有html代码中的第一个,默认on

 	   .....具体看官方文档

 上下文：http,server,location

### Nginx的请求限制

连接频率限制 - limit_conn_module
请求频率限制 - limit_req_module
![1550991680743](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550991680743.png)
http协议是建立在tcp协议基础之上，先进行http请求的三次握手，然后传输数据，最后客户端和服务端通过发送fin和ack请求来维持长连接。

http协议版本与连接的关系：

对于http1.0，tcp是不能被复用的，即一次连接对应一次请求；

对于http1.1，是顺序性tcp复用，即一次连接可以顺序性发起多次http请求；

对于http2.0，是多路复用tcp，即一次连接可以多路并行的发起http请求；

==》http请求是建立在一次tcp连接基础之上；一次tcp请求至少产生一次http请求。

1. 连接限制
   

语法: limit_conn_zone key zone=name:size;
上下文：http
（理解：需要在nginx上存储连接的状态从而对连接进行限制，limit_conn_zone是开辟了一个空间用于存储；key是对什么进行限制，比如ip；zone=name空间的名称，size是空间的大小。
语法：limit_conn zone number;
上下文：http,server,location
（理解：需要结合上面进行使用，zone就是上面声明的name，number是连接数限制，同一时间有number个连接）
2. 请求限制
语法：limit_req_zone key zone=name:size  rate=rate;
上下文：http
(理解：rate单位:个/s)
语法：limit_req zone=name`[burst=number][nodelay]`
上下文：http，server，location
(理解：burst)
case:
```xml
            http{
            	limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
            	limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s; #对于同个客户端ip地址的请求限制最大为1秒1个请求，存储空间为1m大小
                sever{
                   	location / {
                        root /opt/app/code;
                        index index.html;
                        limit_req zone=req_zone burst=3 nodelay; #burst=3表示将3个请求放到缓冲区，等到下一秒执行；nodelay表示其他请求会立刻返回503错误，否则其他请求会一直等待；漏桶原理
                        limit_conn conn_zone 1; # 限制对于同一个ip同时间只能一个连接
                   	}
                }
            }
```

(注：1. binary_remote_addr大约占用4个字节，32位，如果直接存储ip地址，不采用二进制存储，那么大约需要4*4=16字节（64位系统下，int占用4字节）,因此binary节省空间    2. 如果未加nodelay，单个ip需要把并发控制在burst内，如果加了nodelay，单个ip不仅需要把并发控制在burst内，而且不超过rate速率限制)
<font color="00ffff">测试方法：ab工具</font>
命令：ab -n 20 -c 20 http://localhost/2.html
-n表示请求数  -c表示并发数（同一时间发出去的请求)

<font color="00ffff"> remote_addr指的是访问nginx的ip地址，如果客户端和nginx之间有一层代理，如nginx或者cdn，那么该ip就是代理层的ip地址，而非用户真实的ip，获取用户ip方法如下：</font>(需要第一层代理就要设置x_forwarder_for，否则后面无法取出用户ip，一般cdn或者nginx都会设置，但是该种方式有风险，会被侵入，代码中需要做好ip校验，防止sql注入等)

```xml
## 这里取得原始用户的IP地址
map $http_x_forwarded_for  $clientRealIp {
"" $remote_addr;
~^(?P<firstAddr>[0-9\.]+),?.*$$firstAddr;
}

## 针对原始用户 IP 地址做限制
limit_req_zone $clientRealIp zone=one:10m  rate=1r/s;
```

### Nginx的访问控制

1. 基于IP的访问控制 -http_access_module 允许部分ip通过

   语法：allow|deny address | CIDR(网段) | unix: | all;

   上下文：http,server,location,limit_except

   case:

   ```xml
   localtion ~ ^/admin.html {
   	root /opt/app/code;
   	index index.html;
   	deny 222.128.189.17;
   	allow all;
   }
   ```

   

2. 基于用户的信任登录 -http_auth_basic_module

![1550998000894](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550998000894.png)

可解决的办法：通过http_x_forwarded_for

![1550998191296](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550998191296.png)

http_x_forwarded_for = clientIP,proxy(1)IP,proxy(2)IP...

![1550998289186](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550998289186.png)

(x_forwarded_for可能会被改写。。。在http头部中自定义一个变量存储ip地址，既能避免被改写，又能准确拿到客户端地址)

语法：auth_basic string | off; 默认auth_basic off;

上下文：http，server，location，limit_except

语法：auth_basic_user_file file;

上下文：http, server, location, limit_except

(理解：file存储用户名和密码的信息)

case：

1. 生成file文件,使用工具htpasswd进行加密(查看官网)
    htpasswd -c ./auth_conf wy

2. 修改配置

   ```xml
   location {
   	root 
   	index
   	auth_basic  "Auth access" #非off表示开启
   	auth_basic_user_file /etc/nginx/auth_conf
   }
   ```



效果：在访问url时就要求输入用户名和密码

![1550999766720](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550999766720.png)

![1550999803619](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1550999803619.png)

解决方案===========》nginx+lua实现高效验证

或者 Nginx和LDAP打通，利用nginx-auth-ldap模块

### Nginx第三方模块

# 静态资源web服务

![1551000424098](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551000424098.png)

## 静态资源类型

非服务器动态运行生成的文件

| 类型         | 种类         |
| ------------ | ------------ |
| 浏览器端渲染 | html,css,js  |
| 图片         | jpeg,gif,png |
| 视频         | flv,mpeg     |
| 文件         | txt等        |

## 静态资源服务场景-CDN

CDN是内容分发网络，目的是让用户就近取得所需的内容，解决Internet网络拥挤的状态，提高用户访问网站的速度。简单的CDN网络由一台DNS服务器和几台缓存服务器就可以组成。

参考[CDN知识点](https://www.zhihu.com/question/37353035)

![1551013300916](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551013300916.png)

上图是用Nginx作为静态web服务器组成的cdn节点。

## Nginx作为静态资源服务的关键配置

1. 语法：sendfile on | off; 默认是off

上下文：http, server, location, if in location

（引读：-with-file-aio 异步文件读取）

2. 语法：tcp_nopush on | off; 默认off

上下文：http, server, location

作用：sendfile开启的情况下，提高网络包的传输效率（将多个包进行整合为一个包发送给客户端，而不是一个个发送）

3. 语法：tcp_nodelay on | off; 默认on

上下文：http, server, location

作用：keepalive连接下，提高网络包传输的实时性（数据包不等待，实时发送给用户；对于实时性要求高的场景需要打开）（前提是长连接）

4. 压缩

语法：gzip on | off; 默认off

上下文：http, server, location, if in location

作用：压缩传输（减少网络带宽资源占用，减少传输时间）(nginx压缩，浏览器解压)



语法：gzip_comp_level level;默认是1

上下文：http, server, location

作用：压缩比(压缩比越大，文件越小，但是占用更多的服务端资源)



语法：gzip_http_version 1.0 | 1.1; 默认1.1主流

上下文：http, server, location

作用：压缩采用的版本

5. 扩展Nginx压缩模块

http_gzip_static_module - 预读gzip功能(在压缩前，先查看硬盘中有无1.html.gizp文件，如果有，那么直接使用，否则再进行压缩)                     

语法：gzip_static on;

http_gunzip_module - 应用支持gunzip的压缩方式(为了解决部分浏览器不支持gzip功能，采用gunzip压缩)

6. 示例配置：

![1551014870826](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551014870826.png)

# 浏览器缓存

浏览器的缓存是基于http协议的缓存机制（如expires；cache-control）实现的。

流程：

1. 浏览器第一次发起http请求时，首先查看cache目录中是否有该url的缓存文件，没有，则向web服务器发起http请求，让响应，在页面上显示；响应的消息头中包含last-modified(最后修改的时间)、etag(资源唯一标识)、expires(资源在浏览器缓存中的过期时间)，然后浏览器会将文件缓存到cache目录下，并同时保存文件上述信息。

2. 浏览器第二次发起http请求时，发现cache目录中有缓存文件，然后进行校验是否过期，如果没有过期则直接读取缓存文件，不会发起http请求；如果过期了则发送http请求给web服务器，并在请求头中携带相关信息，如if-modified-since(上一次修改时间),if-none-match(上一次请求返回的etag值)，服务器收到请求后首先解析header头部信息，校验，如果该文件从上次时间到限制都没修改过或者etag信息无变化，则服务端直接返回304状态码，告知客户端之间文件无变化，并直接使用缓存文件即可，减少了网络带宽，提升用户体验；如果文件修改了，就会将资源文件返回，并带上新文件的状态信息。

   参考[浏览器缓存请求过程](https://www.cnblogs.com/liumingwang/articles/6681813.html)



校验过期机制

![1551016920260](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551016920260.png)

![1551017089843](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551017089843.png)



Nginx对于静态资源过期时间的配置

原理：通过对返回的response添加cache-control和expires头

语法：expires [modified] time;

​	   expires epoch | max | off;默认是off

上下文：http, server, location, if in location

跨域访问

![1551017643851](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551017643851.png)

浏览器出于安全考虑，禁止在同一个页面访问不同的域名。防止出现CSRF攻击。

nginx解决跨域：向http response返回的消息头中添加access-control-allow-origin信息，浏览器识别到这个信息时，认为服务端允许进行跨域，因此浏览器也就会允许跨域了。

语法： add_header name value [always]; #value是允许哪些站点进行访问,*表示所有站点都允许跨域访问，不安全 name是access-control-allow-origin

上下文：http, server, location, if in location

![1551018434243](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551018434243.png)



![1551018141144](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551018141144.png)

在ajax请求中有对jeson.imooc.com进行访问。

# 防盗链

目的：防止资源被盗用

首要方式：区别哪些请求是非正常的用户请求

## 基于http_refer防盗链配置模块

语法：vaild_referers none | blocked | server_names | string | 正则表达式

(none-无referer信息、blocked-有referer但是被防火墙或者代理去除了,这些值不以http://或者https://开头、server_names-referer来源的头部包含当前的server_names(当前域名)、string或者正则表达式-匹配referer)

(nginx会通过查看referer字段和valid_referers后面的referer列表进行匹配，如果匹配成功，则invalid_referer字段设置为0，否则设置为1)

上下文：server, location

referer：代表网页的来源，即上一页的地址；

防盗链原理：在web服务器层面，根据http协议的referer头信息进行判断，如果来自站外，则统一重写到一个很小的防盗链提醒图片上去。

![1551497711761](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551497711761.png)

![1551496776263](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551496776263.png)

![1551496813166](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551496813166.png)

使用curl命令进行测试,-I 参数只会打印出头部信息,-e参数表示从哪个页面作为源头进行访问;

# 代理

![1551498169768](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551498169768.png)

Nginx可以作为HTTP、POP、ICMP、IMAP、HTTPS、RTMP等代理；

## 正向代理

![1551498248928](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551498248928.png)

正向代理是代理客户端向服务端进行访问。

正向代理类似于一个跳板机，代理访问外部资源；比如翻墙等；

客户端需要知道正向代理服务器的IP地址和端口。

作用：

1. 访问原来无法访问的资源；
2. 可以做缓存，加速访问资源；(做缓存配置)
3. 对客户端进行授权，上网认证等；
4. 能够记录用户的访问记录（上网行为管理），对外隐藏用户信息；

## 反向代理

![1551498683599](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551498683599.png)

反向代理是代理服务端，客户端是无法感知代理的存在，不知道提供服务具体是哪一台服务器。

作用：

1. 保证内网安全，使用反向代理提供的WAF功能，阻止web攻击；（大型网站，通常将反向代理作为公网地址，web服务器是内网）
2. 负载均衡

## 配置

语法：proxy_pass URL;

上下文：location, if in location, limit_except

URL格式：http://localhost:8080/uri/    https://192.168.1.1:80000/uri/  http://unix:/tmp/backend.socket:/uri/（socket形式）

### 反向代理配置

![1551499404849](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551499404849.png)

访问jeson.t.imooc.io/test_proxy.html，会跳转至本地的8080端口进行访问。

### 正向代理配置

这个是最终需要访问的目标服务器配置：

![1551499973440](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551499973440.png)

jeson.t.imocc.io网站只能从116.162.103.228的服务器进行访问，从其他服务器访问时，都会报403无权限访问。

因此，需要在228服务器上进行配置，正向代理。

![1551500321526](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551500321526.png)

http_host即对应的host主机，request_url为对应的请求参数；

配置resolver进行域名解析；比如，浏览器中配置了正向代理服务器的ip和端口，然后在浏览器上输入要访问的网站链接，该链接会被正向代理服务器执行，如果不配置域名解析，那么正向代理服务器如何识别这个访问链接呢？所以必须要配置的！

增加无server_name名的server；<font color="ffff">上述案例的server_name可能需要去掉</font>

# 缓冲区

<font color="ffff">优化</font>

语法：proxy_buffering on | off;默认是on；

上下文：http，server，location

扩展：proxy_buffer_size、proxy_buffers、proxy_busy_buffers_size;

反向代理存在的问题是，大量用户请求时，会加重对服务器性能的冲击影响，通过缓冲区和缓存可以减轻。如果没有缓冲区，数据从server发送到nginx，然后立即传输给client。假设client速度很快，缓冲区可以关闭，从而使数据尽快到达client；有了缓冲区，nginx就可以暂存server的响应，然后按需提供数据给客户端；假设客户端很慢，缓冲区允许nginx关闭到后端的连接，然后它可以以任何可能的速度将数据传输给客户端。

缓冲区能够释放后端服务器以处理更多的请求，减少频繁IO；（对于长轮询需要关闭缓冲区）

工作机制：

1. server端响应数据给nginx，proxy buffer会判断本次响应的数据量大小；
2. 如果buffer足够，那么本次响应数据会直接写入buffer；
3. 如果buffer不够，那么nginx会将部分接收到的数据临时存放在磁盘中，磁盘上的临时文件可以通过procy_temp_path指定，临时文件大小由proxy_max_temp_file_size和proxy_temp_file_write_size指令决定；
4. 一次响应数据接收完毕或者buffer满，nginx会向客户端传输数据。

# 跳转重定向

语法：proxy_redirect default | off  | redirect replacement;

off：不会对301和302请求做任何修改

上下文：http, server, location

nginx作为反向代理服务器时，在接收到上游服务器（服务器端）返回的重定向或者刷新请求（301或者302）时，proxy_redirect可以重设http头部的localtion或者refresh字段。(在后端服务器重定向后的地址，是客户端无法访问的时候，比如客户端只能访问到test1.com的地址，但是后端服务器重定向变成了test2.com，此时可以将test2.com改为test1.com返回给客户端，那么客户端会拿新的url再进行请求)

![1551512247289](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551512247289.png)

将被代理服务器发出的重定向http协议的location改为https协议：proxy_redirect ~^http://([^:]+)(:\d+)?(.*)$ https://$1$2$3;

# 头信息

语法：proxy_set_header filed value;

默认：proxy_set_header Host $proxy_host;

​	   proxy_set_header Connection close;

上下文：http，server，location

扩展：proxy_hide_header、proxy_set_body

# 超时

语法：proxy_connect_timeout time;默认是60s

上下文：http，server，location

nginx到后端服务器的连接超时；

扩展：proxy_read_timeout（建立连接完毕后，nginx发送请求给服务器处理，即这部分的处理时间）、proxy_send_timeout（服务端请求后，发送给客户端的时间）

# 实战配置

```xml
server {
	listen      80;
	server_name  localhost jeson.t.imooc.io;
}
access_log     /var/log/nginx/test_proxy.access.log  main;
location / {
	proxy_pass http://127.0.0.1:8080;
	proxy_redirect  default;
	proxy_set_header  Host  $http_host;
	proxy_set_header  X-Real-IP  $remote_addr;//实际ip

	proxy_connect_timeout 30；	
	proxy_send_timeout  60;
	proxy_read_timeout  60;

	proxy_buffer_size   32k;
	proxy_buffering on;
	proxy_buffers  4    128k;
	proxy_busy_buffers_size   256k;
	proxy_max_temp_file_size  256k;
}
```

# 负载均衡

![1551518569991](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551518569991.png)

GSLB是全局的负载均衡，涉及的地域范围比较广，以国家或者省单为进行全局负载均衡。

通过中心调度节点、边缘调度节点以及应用节点实现了全局的负载均衡。

![1551518714110](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551518714110.png)

![1551518812726](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551518812726.png)

对应于OSI模型的传输层，tcp/ip层协议控制，只需要进行包转发，性能好，不需要复杂的逻辑处理；

![1551518904512](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551518904512.png)

对应于OSI的应用层，如http协议等；可以对头信息改写，安全规则控制等；Nginx是典型的七层的SLB。现在也支持四层了。

## Nginx

![1551519065591](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551519065591.png)

Nginx通过proxy_pass将http请求进行转发至上游服务器upstream_server，根据不同的路由规则实现了负载均衡。

## upstream配置

语法：upstream name {}

上下文：http

![1551519450177](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551519450177.png)

![1551519585699](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551519585699.png)

![1551519632566](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551519632566.png)

## 调度算法

![1551519794122](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551519794122.png)

默认是轮询。

url_hash: hash $request_uri;

支持对uri某个变量进行hash：通过正则表达式提取uri中的某个变量，针对该变量进行自定义key的hash。

# 缓存服务

![1551521022775](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551521022775.png)

客户端缓存--代理缓存--服务端缓存

![1551521091527](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551521091527.png)

## 配置

![1551521236369](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1551521236369.png)

--存放缓存文件的路径

语法：proxy_cache zone | off; 默认off

上下文：http，server，location

语法：proxy_cache_valid [code ...] time;

上下文：http，server，location

--- 缓存过期周期

语法：proxy_cache_key string；

默认：proxy_cache_key $scheme$proxy_hosts$request_uri；(协议-域名-uri)

上下文：http，server，location

--- 缓存的维度

配置案例：

![1552195694130](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552195694130.png)

缓存设置为2级目录结构，缓存keys的空间名称为imooc_cache，大小为10m(一般来说，1m可以存储8000key)，60min内没有被访问过，那么会被清理；关闭临时文件，防止性能损耗；

## 删除缓存

1. rm -rf 删除那整个缓存目录的内容
2. 第三方扩展模块,ngx_cache_purge

## 部分页面不缓存

语法：proxy_no_cache string ...;

上下文：http，server，location

![1552201780402](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552201780402.png)

# 大文件分片

语法：slice size；默认slice 0；

上下文：http，server，location

![1552202048247](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552202048247.png)

原理：客户端第一次发出请求下载很大的文件，nginx收到请求后，首先向后端服务器发送请求获取这个文件的大小，然后按照size进行分片，分割为多个子请求去请求后端，每个请求的响应内容是小的文件，可以单独进行花奴才能，各个子请求响应之间互不影响。

优点：每个子请求收到的数据都会形成一个独立文件，一个请求断了，其他请求不受影响。

缺点：当文件很大或者slice很小的时候，可能会导致文件描述符耗尽等情况。

[参考链接](https://my.oschina.net/u/2539854/blog/1634878?from=timeline)

# 动静分离

通过中间件将动态请求和静态请求分离。

why：对于服务端来说，减少不必要的请求的消耗；静态资源不需要复杂的cpu运算；对于客户端来说，减少请求延时，提高响应。

![1552205369467](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552205369467.png)

html文件：

![1552204588460](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552204588460.png)

浏览器输入网址后，会发送多个http请求获取资源，

![1552204778170](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552204778170.png)

nginx配置：

![1552205029878](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552205029878.png)

# rewrite规则

首先url重写和重定向

## 场景

1. url访问跳转，支持开发设计；

   页面跳转、兼容性设计(网站升级时，保证老的用户仍然能够访问之前的网站)、展示效果(复杂的url进行重写，避免大量长串)等；

2. SEO优化；（使路径符合搜索引擎的搜索规范，提高效率）

3. 维护：后台维护（跳转至维护的页面）、流量转发 

4. 安全：伪静态，将真实的动态页面进行伪装，从而让黑客认为该页面是静态页面。

## 配置语法

语法：rewrite regex replacement [flag]

上下文：server，location，if

rewrite ^(.*)$ /pages/maintain.html break;==对所有页面都重定向至维护页面

## 正则表达式

![1552206263326](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552206263326.png)

![1552206296463](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552206296463.png)

![1552206332804](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552206332804.png)

![1552206386124](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552206386124.png)

## flag

![1552206635988](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552206635988.png)

![1552207189890](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552207189890.png)

效果：访问/break时，提示404 not found；访问/last，正常跳转至/test。

原因：break时，请求会到对应的root目录下找test文件，发现没有这个文件，则报404；而last时，在对url进行替换后，会重新发送一次请求，从而命中了新的location /test/。break只会在当前location下寻找资源，last会发起新的http请求。（客户端是无感知的，对于客户端来说，依然是一次请求）

临时重定向和永久重定向对于客户端来说都是两次http请求，区别是，临时重定向每次都会向服务端发送请求，而永久重定向只有在第一次会请求到服务端，并在客户端保存重定向后的结果，下次再去请求该url时，直接在客户端请求了重定向后的结果，不会经过服务端了。

## 其他

![1552208074397](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552208074397.png)

当访问/course-11-22-33.html时，会自动定位到root目录下的/course/11/22/33.html真实的资源文件。==SEO优化

![1552208232728](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552208232728.png)

如果是chrome浏览器，并且以nginx开头，进行临时重定向。

![1552208275088](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552208275088.png)

如果请求的文件路径不存在，则重定向至新的url。

## 优先级

1. 执行server块的rewrite指令
2. 执行location匹配
3. 执行选定的location的rewrite

## 书写

![1552208489051](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552208489051.png)

推荐以下写法：

![1552208516673](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552208516673.png)

# 进阶模块学习

## secure_link_module模块

（访问限制和防盗链功能，http头部信息可以改写，验证机制不完善）

作用：

1. 限制并允许检查请求的链接的真实性以及保护资源免遭未经授权的访问

2. 限制链接生效周期

   （后端加密返回给前端，类似于数字签名的机制）

语法：secure_link expression;

上下文：http,server,location

语法：secure_link_md5 expression;

上下文：server,location

### 验证机制

![1552209474693](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552209474693.png)

![1552209673004](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552209673004.png)

 由服务端生成md5和expires过期时间，由nginx端进行验证。

![1552209913068](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552209913068.png)

通过secure_link取出md5和expires参数的值，然后nginx端采用和服务端相同的md5加密方式对原始的url进行加密，再比较md5值是否一致即可。（md5是不可逆的），其中，imooc是加密串，只有服务端才能有。

服务端加密：

![1552210147603](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552210147603.png)

## geoip_module模块

基于ip地址匹配MaxMind GeoIP二进制文件，读取IP所在地域信息。

yum install nginx-module-geoip

### 场景

1. 区别国内外的http访问规则
2. 区别国内城市地域的http访问规则

![1552210624802](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552210624802.png)

# HTTPS

## 为什么需要https

http协议不安全，传输数据被中间人盗用，信息泄漏；数据内容劫持、篡改

## HTTPS协议实现

对传输内容进行加密以及身份验证。https = http + ssl

### 对称和非对称加密

![1552216586570](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552216586570.png)

![1552216614798](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552216614798.png)

### HTTPS加密原理

同时用到了对称加密和非对称加密。

![1552216719682](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552216719682.png)

非对称加密在对大数据量进行加解密的适合，性能比较差，速度慢，而对称加密的性能更好。

第三方伪造客户端和服务端，如下：

![1552216974323](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552216974323.png)

因此对称加密和非对称加密是无法阻止第三方拦截的。

HTTPS依靠CA证书解决了该问题。

![1552218908125](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552218908125.png)

CA证书中一般包含了公钥以及证书编号，即数字签名。

### CA证书的生成

这里只讲解自签证书的生成，不介绍公司进行签发的证书（需要付钱）。

1. 确认系统是否安装openssl

   #openssl version

2. nginx作为服务端，需要编译ssl模块

   #nginx -v   (--with-http_ssl_module)

(openssl生成密钥，通过密钥生成证书签名的请求文件csr文件，将密钥和csr文件打包以及对应的签名机构，以及网站名称、公司的信息，让第三方机构进行签名，会返回给对应的ca证书)

生成密钥和CA证书：

1. 生成key密钥

   #mkdir ssl_key

   #cd ssl_key

   #openssl genrsa -idea -out jesonc.key 1024

   ![1552223169494](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552223169494.png)

2. 生成证书签名请求文件csr文件

   #openssl req -new -key jesonc.key -out jesonc.csr

   ![1552223336945](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552223336945.png)

3. 生成证书签名文件ca文件

   将上述两个文件打包发送给签名机构进行权威的签名请求。

   对于个人而言，建立自己的签名证书如下：

   #openssl x509 -req -days 3650 -in jesonc.csr -signkey jesonc.key -out jesonc.crt

   crt文件即为ca文件。

   需要配置https服务：ssl服务，ssl证书文件，ssl证书的密码文件

   nginx端（服务端）配置：(参考网上配置)

   listen 443 default ssl;

   ssl on;

   ssl_certificate ssl/server.crt;  #证书(公钥，发送给客户端)

   ssl_certificate_key ssl/server.key; #私钥

也可以不生成csr文件，直接通过key生成crt文件。

#openssl req -days 36500 -x509 -sha256 -nodes -newkey rsa:2048 -keyout jesonc.key -out jesonc_apple.crt

![1552224217053](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224217053.png)

![1552224274015](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224274015.png)

### 服务优化

1. 激活keepalive长连接（避免多次的连接和认证）

2. 设置ssl session缓存

   ![1552224504674](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224504674.png)

### 抓包

https解决的是在传输链路中对数据进行加密的问题，也就是传输层进行加解密；而我们抓包是在本地机器进行抓包，抓取的是http报文，这个是应用层，此时数据已经被解密了，也就是说抓包中看到的数据也就是client端看到的数据；（wireshark）

对于charles（fidder）是使用了https代理的功能，来完成查看https明文的目睹，也就是ssl中间人攻击。当允许charless作为代理时，首先是需要本人同意的，其次client端需要信任charless的证书，那么，client实际上与charless进行通信，然后charless再与server端进行通信。

### 参考

[https协议原理](http://blog.jobbole.com/110354/)--ca机构会使用不同的公私钥进行签名。

[https://www.cnblogs.com/lovesong/p/5186200.html](https://www.cnblogs.com/lovesong/p/5186200.html)

[http://blog.jobbole.com/110354/](http://blog.jobbole.com/110354/)

1. http://www.ruanyifeng.com/blog/2006/12/notes_on_cryptography.html
2. http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html
3. http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html
4. http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html

# Nginx Lua

lua 是一个简洁、轻量（不需要解释器）、可扩展（依赖c的模块）的脚本语言。

## 优势

充分结合nginx的并发处理epoll优势和lua的轻量级实现简单的功能，应对高并发的场景。

## lua基础语法

yum install lua

![1552224800853](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224800853.png)

![1552224908195](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224908195.png)

![1552224932768](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224932768.png)

![1552224945701](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224945701.png)

![1552224975170](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224975170.png)

![1552224992322](C:\Users\wangyuan1\AppData\Roaming\Typora\typora-user-images\1552224992322.png)





 

















