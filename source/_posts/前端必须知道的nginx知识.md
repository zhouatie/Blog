---
title: 前端必须知道的nginx知识
date: 2019-09-01 01:35:12
categories: 前端
tags: ['nginx']
thumbnail: https://s1.ax1x.com/2020/03/30/GeMNLj.png
---

前端开发者进阶之路不得不学 nginx。

<!-- more -->

## 本文内容涉及

- nginx 命令
- 配置文件、配置解释
- 变量
- 日志
- 跨域
- 代理请求
- location 拦截详解
- gzip
- 防盗链
- 反向代理、正向代理
- 负载均衡
- 缓存
- rewrite

## 正文

### 命令

- nginx -T 查看当前 nginx 最终的配置
- nginx -t 检查配置文件是否有语法错误
- nginx -s reload 向主进程发送信号，重新加载配置文件
- nginx -s stop 快速关闭
- nginx -s quit 等待工作进程处理完成后关闭

### 配置文件

```shell
user  nginx; # 定义 Nginx 运行的用户
worker_processes  1; # 设置工作进程的数量

error_log  /var/log/nginx/error.log warn; #nginx 错误日志
pid        /var/run/nginx.pid; # nginx.pid存放的是nginx的master进程的进程号

# events模块中包含nginx中所有处理连接的设置
events {
    #工作进程的最大连接数量 理论上每台nginx服务器的最大连接数为worker_processes*worker_connections worker_processes为我们再main中开启的进程数
    worker_connections  1024;
}

# 提供http服务相关的一些配置参数
http {
    # include 是个主模块指令，可以将配置文件拆分并引用，可以减少主配置文件的复杂度, 这里是加载一些文件类型
    include       /etc/nginx/mime.types;
    # default_type 属于HTTP核心模块指令，这里设定默认类型为二进制流，也就是当文件类型未定义时使用这种方式
    default_type  application/octet-stream;

    # 定义日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # 定义本虚拟主机的访问日志
    access_log  /var/log/nginx/access.log  main;
    sendfile on; #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改 成off
    sendfile        on;
    tcp_nopush     on;

    keepalive_timeout  65;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k; #最小压缩文件大小
    gzip_buffers 4 16k; #压缩缓冲区
    gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2; #压缩等级

    limit_zone crawler $binary_remote_addr 10m; #开启限制IP连接数的时候需要使用

    server {
        listen       80; # 表示http监听的端口
        server_name  localhost; # 域名可以有多个，用空格隔开  当出现多个server监听的端口一样时，可通过server_name 做区分
        # 有些指令可以支持正则表达式
        location / { # 用于匹配URI，这里也可以用正则的方式匹配 ~ 表示大小写敏感  ~*表示大小写不敏感
            root   html; # 指定了静态文件的根目录
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html; # 表示当在500 502 503 504 错误码的时候重定向到 /50x.html
        location = /50x.html {
            root   html;
        }
    }
    include /etc/nginx/conf.d/*.conf; # 用于加载其他配置文件进来
}
```

### 变量

| 字段 | 说明 |
| - | - |
|`$arg_PARAMETER` | 这个变量值为：`GET` 请求中变量名 PARAMETER 参数的值。|
|`$args` | 这个变量等于 `GET` 请求中的参数。例如，`foo=123&bar=blahblah;`这个变量只可以被修改
|`$binary_remote_addr` | 二进制码形式的客户端地址。
|`$body_bytes_sent` | 传送页面的字节数
|`$content_length` | 请求头中的 `Content-length` 字段。
|`$content_type` | 请求头中的 `Content-Type` 字段。
|`$cookie_COOKIE` | `cookie COOKIE` 的值。
|`$document_root` | 当前请求在 `root` 指令中指定的值。
|`$document_uri` | 与 `$uri` 相同。
|`$host` | 请求中的主机头`(Host)`字段，如果请求中的主机头不可用或者空，则为处理请求的 `server` 名称(处理请求的 `server` 的 `server_name` 指令的值)。值为小写，不包含端口。
|`$hostname` | 机器名使用 `gethostname` 系统调用的值
|`$http_HEADER` | `HTTP`请求头中的内容，HEADER 为 HTTP 请求中的内容转为小写，-变为\_(破折号变为下划线)，例如：`$http_user_agent`(`Uaer-Agent` 的值), `$http_referer`...;
|`$sent_http_HEADER` | `HTTP`响应头中的内容，`HEADER`为`HTTP`响应中的内容转为小写，-变为\_(破折号变为下划线)， 例如：`$sent_http_cache_control, $sent_http_content_type`...;|
|`$is_args` | 如果`$args` 设置，值为"?"，否则为""。
|`$limit_rate` | 这个变量可以限制连接速率。
|`$nginx_version` | 当前运行的 `nginx` 版本号。
|`$query_string` | 与`$args` 相同。
|`$remote_addr` | 客户端的 IP 地址。
|`$remote_port` | 客户端的端口。
|`$remote_user` | 已经经过 `Auth Basic Module` 验证的用户名。
|`$request_filename` | 当前连接请求的文件路径，由 `root` 或 `alias` 指令与 `URI` 请求生成。
|`$request_body` | 这个变量（0.7.58+）包含请求的主要信息。在使用 `proxy_pass` 或 `fastcgi_pass` 指令的 `location` 中比较有意义。
|`$request_body_file` | 客户端请求主体信息的临时文件名。
|`$request_completion` | 如果请求成功，设为"OK"；如果请求未完成或者不是一系列请求中最后一部分则设为空。
|`$request_method` | 这个变量是客户端请求的动作，通常为 GET 或 POST。包括 0.8.20 及之前的版本中，这个变量总为 main request 中的动作，如果当前请求是一个子请求，并不使用这个当前请求的动作。
|`$request_uri` | 这个变量等于包含一些客户端请求参数的原始 URI，它无法修改，请查看\$uri 更改或重写 URI。
|`$scheme` | 所用的协议，比如`http`或者是`https`，比如`rewrite ^(.+)$ $scheme://example.com$1 redirect`;
|`$server_addr` | 服务器地址，在完成一次系统调用后可以确定这个值，如果要绕开系统调用，则必须在 `listen` 中指定地址并且使用 `bind` 参数。
|`$server_name` | 服务器名称。
|`$server_port` | 请求到达服务器的端口号。
|`$server_protocol` | 请求使用的协议，通常是 `HTTP/1.0` 或 `HTTP/1.1`。

### 日志

#### error_log

表示错误日志。`error_log /var/log/nginx/error.log warn;` 。错误日志记录了访问出错的信息。有利于我们排查错误。

#### access_log

表示访问日志。需要指定日志格式 `log_format`。通过访问日志我们可以得到用户的 IP 地址、浏览器的信息，请求的处理时间等信息

#### log_format

日志格式可以结合上面的变量，来指定访问日志的输出格式。

如：

```shell
log_format main '$host $document_uri $server_addr $remote_addr $remote_port';
server {
  listen 5002;
  access_log /usr/local/etc/nginx/servers/access.log main;
  location / {
    root /usr/local/etc/nginx/servers;
    index index.html;
  }
}
```

定义了个日志格式 main

然后在将`access_log`绑定日志`main`

输出：

```shell
localhost /index.html 127.0.0.1 127.0.0.1 61517
```

可以看到，access_log 按照我的 main 日志格式输出了。

### 跨域

相信作为前端开发者，你肯定知道 CORS。

- Access-Control-Allow-Origin <origin> | \*;
  其中，origin 参数的值指定了允许访问该资源的外域 URI。对于不需要携带身份凭证的请求，服务器可以指定该字段的值为通配符，表示允许来自所有域的请求。

- Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header

在跨域访问时，XMLHttpRequest 对象的 getResponseHeader()方法只能拿到一些最基本的响应头，Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma，如果要访问其他头，则需要服务器设置本响应头。

- Access-Control-Max-Age
  Access-Control-Max-Age 头指定了 preflight 请求的结果能够被缓存多久;Access-Control-Max-Age: 86400

- Access-Control-Allow-Credentials

指定了当浏览器的 credentials 设置为 true 时是否允许浏览器读取 response 的内容。当用在对 preflight 预检测请求的响应中时，它指定了实际的请求是否可以使用 credentials。请注意：简单 GET 请求不会被预检；如果对此类请求的响应中不包含该字段，这个响应将被忽略掉，并且浏览器也不会将相应内容返回给网页

话不多少，下面就直接贴代码：

```javascript
// index.js
import axios from 'axios';

axios.get('http://127.0.0.1:5556/api/abc').then(res => {
	console.log(res, 'res');
	document.write(res);
});
```

```shell
server {
    listen 5556;
    location /api {
        proxy_pass http://localhost:5000;
        add_header Access-Control-Allow-Origin *;
    }
}
```

将 5556 端口的请求路径 /api 开头的 转发到 5000 端口的 node 服务上。添加上`Access-Control-Allow-Origin *`就可解决跨域问题。

### 代理请求

代理请求可以看下上面的跨域。把 5556 端口的请求转发到 5000 端口的服务上。

### location 拦截详解

#### 修饰符

- `=`: 精确匹配路径
- `~`: 表示用该符号后面的正则去匹配路径，区分大小写
- `~*`: 表示用该符号后面的正则去匹配路径，不区分大小写
- `^~`: 表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找。

```shell
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /api/ {
    [ configuration C ]
}

location ^~ /static/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

请求/精准匹配 A，不再往下查找。

请求/index.html 匹配 B。首先查找匹配的前缀字符，找到最长匹配是配置 B，接着又按照顺序查找匹配的正则。结果没有找到，因此使用先前标记的最长匹配，即配置 B。

请求/api/list 匹配 C。首先找到最长匹配 C，由于后面没有匹配的正则，所以使用最长匹配 C。

请求/user/1.jpg 匹配 E。首先进行前缀字符的查找，找到最长匹配项 C，继续进行正则查找，找到匹配项 E。因此使用 E。

请求/static/img.jpg 匹配 D。首先进行前缀字符的查找，找到最长匹配 D。但是，特殊的是它使用了^~修饰符，不再进行接下来的正则的匹配查找，因此使用 D。这里，如果没有前面的修饰符，其实最终的匹配是 E。

请求/router/pageA 匹配 B。因为 B 表示任何以/开头的 URL 都匹配。在上面的配置中，只有 B 能满足，所以匹配 B。

### gzip

#### 配置说明

```shell
# gzip 默认off 默认关闭gzip
gzip             on;
# gzip_min_length 默认0
# 作用域: http, server, location
# 设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
# 默认值是0，不管页面多大都压缩。
# 建议设置成大于1k的字节数，小于1k可能会越压越大。 即: gzip_min_length 1024
gzip_min_length  1k;
# gzip_comp_level 默认 1 范围 1 ~ 9
# 作用域: http, server, location
# gzip压缩比，1 压缩比最小处理速度最快，9 压缩比最大但处理最慢（传输快但比较消耗cpu）。
gzip_comp_level  6;
# 默认值: gzip_types text/html
# 作用域: http, server, location
# 匹配MIME类型进行压缩，（无论是否指定）"text/html"类型总是会被压缩的。
# 注意：如果作为http server来使用，主配置文件中要包含文件类型配置文件
gzip_types       text/plain application/x-javascript text/css application/xml application/javascript application/json;
```

#### 代码演示

```shell
server {
  listen 9002;
  #gzip on;
  location / {
    root /usr/local/etc/nginx/servers/;
    index gzip.html;
  }
}
```

我们先记录下未开启 gzip 时加载的文件大小
![without-gzip](https://s1.ax1x.com/2020/05/05/YivuOs.png)

size 显示的是 1.3kb

然后我们将 gzip 注释去掉的结果如下

![with-gzip](https://s1.ax1x.com/2020/05/05/YivMmn.png)

可以看到只有 300B 了，当然你还可以根据其他配置，比如来控制压缩等级来控制输出的大小。我们前端项目打包的时候可以开启 gzip，这样 nginx 就不用在服务器上进行 gzip 压缩了。

### 防盗链

```shell
location ~ .*\.(jpg|png|gif)$ {
  valid_referers none blocked 47.104.184.134;
  if ($invalid_referer) {
    return 403;
  }
  root /data/images;
}
```

valid_referers none | blocked | server_names | string ....;
none 检测 Referer 头域不存在的请求
blocked 检测 Referer 头域的值被防火墙或者代理服务器删除或伪装的情况。
这种情况下，该头域的值不以“http://”或者“https：//”开头
server_names 设置一个或多个 URL,检测 Referer 头域的值是否是这些 URL 中的某个。
从 nginx 0.5.33 以后支持使用通配符“\*”。

`valid_referers` 用于支持访问该资源的 referers

\$invalid_referer 这个变量为 true 表示不符合上面定义的规则。就 return 403

### 反向代理、正向代理

#### 正向代理

```shell
location / {
  proxy_pass http://$http_host$request_uri;
}
```

正向代理你可以理解为代理客户端，比如 VPN。因为国内无法访问国外的网站，所以通过将请求转发到 VPN 服务器，VPN 将你的请求原封不动的转发到国外网址。正向代理，客户端知道服务端，服务端不知道客户端。

#### 反向代理

比如说下面将会讲到的负载均衡。所有请求统一走到一个 nginx 服务上，由这个 nginx 服务讲请求分配到多台服务器上。

### 负载均衡

#### 代码演示

```shell
.
├── 9004.html
├── 9005.html
├── 9006.html
└── upstream.conf
```

```shell
server {
	listen 9004;
	location / {
		root /usr/local/etc/nginx/servers/;
		index 9004.html;
	}
}
server {
        listen 9005;
        location / {
                root /usr/local/etc/nginx/servers/;
                index 9005.html;
        }
}
server {
        listen 9006;
        location / {
                root /usr/local/etc/nginx/servers/;
                index 9006.html;
        }
}
upstream atie {
	server localhost:9004;
    server localhost:9005;
    server localhost:9006;
}

server {
	listen 9003;
	location / {
		proxy_pass http://atie;
	}

}
```

通过上面代码可以看到我通过访问 9003 端口，均衡到其他端口。

打开浏览器访问 localhost:9003 可以看到页面会分别加载 9004 ~ 9006.html 页面

#### 配置介绍

| 状态         | 描述                                  |
| ------------ | ------------------------------------- |
| down         | 不参与负载均衡                        |
| backup       | 备份的服务器                          |
| max_fails    | 允许请求失败的次数                    |
| fail_timeout | 经过 max_fails 失败后，服务暂停的时间 |
| max_conns    | 限制最大的接收的连接数                |
| weight       | 权重比                                |

```shell
upstream atie {
	server localhost:9004 down; # 这里如果加了down 表示负载均衡的时候就不会转到这个服务
    server localhost:9005 backup; # 表示备份 只有当服务挂了，才会走到这个备份服务上
    server localhost:9006 max_fails=1 fail_timeout=10s; # 超过最大次数后，在fail_timeout时间内，新的请求将不会分配给这台机器。
}
```

```shell
upstream atie {
	server localhost:9004 weight=1;
    server localhost:9005 weight=2;
    server localhost:9006 weight=3;
}
```

当你访问 6 次时，一次走到 9004 2 次走到 9005 3 次走到 9006

### 缓存

通过添加缓存请求头设置过期时间等

```shell
location ~ .*\.(gif|jpg|png|css|js)(.*) {
    expires 90d; # 设置有效90天
}
```

### rewrite

rewrite 功能就是，使用 nginx 提供的全局变量或自己设置的变量，结合正则表达式和标志位实现 url 重写以及重定向。rewrite 只能放在 server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用

```shell
location / {
    rewrite /rewrite/(.*) http://www.$1.com;
    return 200 "ok";
}
# 在浏览器中输入 127.0.0.1:8080/rewrite/google
# 则临时重定向到 www.google.com
# 后面的 return 指令将没有机会执行了
```
