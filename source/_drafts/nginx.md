

```
location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css)
```
* `~`代表匹配时区分大小写
* `.*`代表任意字符都可以出现零次或多次，即资源名不限制
* `\.`代表匹配后缀分隔符.
* `(html|...|css)`代表匹配括号里所有静态资源类型

简单一句话概述：该配置表示匹配以`.html~.css`为后缀的所有资源请求。

## 资源压缩
Nginx 提供了三个支持资源压缩的模块`ngx_http_gzip_module、ngx_http_gzip_static_module、ngx_http_gunzip_module`，其中`ngx_http_gzip_module`属于内置模块，代表着可以直接使用该模块下的一些压缩指令，后续的资源压缩操作都基于该模块。

| 参数项 | 释义 | 参数值 |
| :--: | :--: | :--: |
| gzip | 开启或关闭压缩机制 | on/off |
| gzip_types | 根据文件类型选择性开启压缩机制 | text/plain、text/css、... |
| gzip_comp_level | 设置压缩级别，越高资源消耗越大 | 1~9 |
| gzip_vary | 设置响应头部是否携带Vary: Accept-Encoding | 
| gzip_buffers | 设置处理压缩请求的缓冲区数量和大小 | 数量大小，如16 8k |
| gzip_disable | 针对不同客户端的请求来设置是否开启压缩 | 如`.*Chrome.*` |
| gzip_http_version | 指定压缩响应所需要的最低HTTP请求版本 | 如1.1 |
| gzip_min_length | 设置触发压缩的文件最低大小 | 如512k |
| gzip_proxied | 对于后端服务器的响应结果是否开启压缩 | off、expired、... |

```
http{
  # 开启压缩机制
  gzip on;
  # 指定会被压缩的文件类型(也可自己配置其他类型)
  gzip_types text/plain application/javascript text/css application/xml text/javascript image/jpeg image/gif image/png;
  # 设置压缩级别，越高资源消耗越大，但压缩效果越好
  gzip_comp_level 5;
  # 在头部中添加Vary: Accept-Encoding（建议开启）
  gzip_vary on;
  # 处理压缩请求的缓冲区数量和大小
  gzip_buffers 16 8k;
  # 对于不支持压缩功能的客户端请求不开启压缩机制
  gzip_disable "MSIE [1-6]\."; # 低版本的IE浏览器不支持压缩
  # 设置压缩响应所支持的HTTP最低版本
  gzip_http_version 1.1;
  # 设置触发压缩的最小阈值
  gzip_min_length 2k;
  # 关闭对后端服务器的响应结果进行压缩
  gzip_proxied off;
}
```
在上述的压缩配置中，最后一个gzip_proxied选项，可以根据系统的实际情况决定，总共存在多种选项：
off：关闭Nginx对后台服务器的响应结果进行压缩。
expired：如果响应头中包含Expires信息，则开启压缩。
no-cache：如果响应头中包含Cache-Control:no-cache信息，则开启压缩。
no-store：如果响应头中包含Cache-Control:no-store信息，则开启压缩。
private：如果响应头中包含Cache-Control:private信息，则开启压缩。
no_last_modified：如果响应头中不包含Last-Modified信息，则开启压缩。
no_etag：如果响应头中不包含ETag信息，则开启压缩。
auth：如果响应头中包含Authorization信息，则开启压缩。
any：无条件对后端的响应结果开启压缩机制。

## 缓冲区
缓冲区的配置项：
* proxy_buffering：是否启用缓冲机制，默认为on关闭状态。
* client_body_buffer_size：设置缓冲客户端请求数据的内存大小。
* proxy_buffers：为每个请求/连接设置缓冲区的数量和大小，默认4 4k/8k。
* proxy_buffer_size：设置用于存储响应头的缓冲区大小。
* proxy_busy_buffers_size：在后端数据没有完全接收完成时，Nginx可以将busy状态的缓冲返回给客户端，该参数用来设置busy状态的buffer具体有多大，默认为proxy_buffer_size*2。
* proxy_temp_path：当内存缓冲区存满时，可以将数据临时存放到磁盘，该参数是设置存储缓冲数据的目录。
* path是临时目录的路径。语法：proxy_temp_path path; path是临时目录的路径
* proxy_temp_file_write_size：设置每次写数据到临时文件的大小限制。
* proxy_max_temp_file_size：设置临时的缓冲目录中允许存储的最大容量。
* 非缓冲参数项：
 * proxy_connect_timeout：设置与后端服务器建立连接时的超时时间。
 * proxy_read_timeout：设置从后端服务器读取响应数据的超时时间。
 * proxy_send_timeout：设置向后端服务器传输请求数据的超时时间。

```
http{  
  proxy_connect_timeout 10;  
  proxy_read_timeout 120;  
  proxy_send_timeout 10;  
  proxy_buffering on;  
  client_body_buffer_size 512k;  
  proxy_buffers 4 64k;  
  proxy_buffer_size 16k;  
  proxy_busy_buffers_size 128k;  
  proxy_temp_file_write_size 128k;  
  proxy_temp_path /soft/nginx/temp_buffer;  
}
```
上述的缓冲区参数，是基于每个请求分配的空间，而并不是所有请求的共享空间。
## 缓存机制
对于性能优化而言，缓存是一种能够大幅度提升性能的方案，因此几乎可以在各处都能看见缓存，如客户端缓存、代理缓存、服务器缓存等等，Nginx的缓存则属于代理缓存的一种。对于整个系统而言，加入缓存带来的优势额外明显：

减少了再次向后端或文件服务器请求资源的带宽消耗。
降低了下游服务器的访问压力，提升系统整体吞吐。
缩短了响应时间，提升了加载速度，打开页面的速度更快。
那么在Nginx中，又该如何配置代理缓存呢？先来看看缓存相关的配置项：

「proxy_cache_path」：代理缓存的路径。
```
proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
```
参数项的含义：
path：缓存的路径地址。
levels：缓存存储的层次结构，最多允许三层目录。
use_temp_path：是否使用临时目录。
keys_zone：指定一个共享内存空间来存储热点Key(1M可存储8000个Key)。
inactive：设置缓存多长时间未被访问后删除（默认是十分钟）。
max_size：允许缓存的最大存储空间，超出后会基于LRU算法移除缓存，Nginx会创建一个Cache manager的进程移除数据，也可以通过purge方式。
manager_files：manager进程每次移除缓存文件数量的上限。
manager_sleep：manager进程每次移除缓存文件的时间上限。
manager_threshold：manager进程每次移除缓存后的间隔时间。
loader_files：重启Nginx载入缓存时，每次加载的个数，默认100。
loader_sleep：每次载入时，允许的最大时间上限，默认200ms。
loader_threshold：一次载入后，停顿的时间间隔，默认50ms。
purger：是否开启purge方式移除数据。
purger_files：每次移除缓存文件时的数量。
purger_sleep：每次移除时，允许消耗的最大时间。
purger_threshold：每次移除完成后，停顿的间隔时间。

「proxy_cache」：开启或关闭代理缓存，开启时需要指定一个共享内存区域。

语法：

proxy_cache zone | off;
zone为内存区域的名称，即上面中keys_zone设置的名称。

「proxy_cache_key」：定义如何生成缓存的键。

语法：

proxy_cache_key string;
string为生成Key的规则，如$scheme$proxy_host$request_uri。

「proxy_cache_valid」：缓存生效的状态码与过期时间。

语法：

proxy_cache_valid [code ...] time;
code为状态码，time为有效时间，可以根据状态码设置不同的缓存时间。

例如：proxy_cache_valid 200 302 30m;

「proxy_cache_min_uses」：设置资源被请求多少次后被缓存。

语法：

proxy_cache_min_uses number;
number为次数，默认为1。

「proxy_cache_use_stale」：当后端出现异常时，是否允许Nginx返回缓存作为响应。

语法：

proxy_cache_use_stale error;
error为错误类型，可配置timeout|invalid_header|updating|http_500...。

「proxy_cache_lock」：对于相同的请求，是否开启锁机制，只允许一个请求发往后端。

语法：

proxy_cache_lock on | off;
「proxy_cache_lock_timeout」：配置锁超时机制，超出规定时间后会释放请求。

proxy_cache_lock_timeout time;
「proxy_cache_methods」：设置对于那些HTTP方法开启缓存。

语法：

proxy_cache_methods method;
method为请求方法类型，如GET、HEAD等。

「proxy_no_cache」：定义不存储缓存的条件，符合时不会保存。

语法：

proxy_no_cache string...;
string为条件，例如$cookie_nocache $arg_nocache $arg_comment;

「proxy_cache_bypass」：定义不读取缓存的条件，符合时不会从缓存中读取。

语法：

proxy_cache_bypass string...;
和上面proxy_no_cache的配置方法类似。

「add_header」：往响应头中添加字段信息。

语法：

add_header fieldName fieldValue;
「$upstream_cache_status」：记录了缓存是否命中的信息，存在多种情况：

MISS：请求未命中缓存。
HIT：请求命中缓存。
EXPIRED：请求命中缓存但缓存已过期。
STALE：请求命中了陈旧缓存。
REVALIDDATED：Nginx验证陈旧缓存依然有效。
UPDATING：命中的缓存内容陈旧，但正在更新缓存。
BYPASS：响应结果是从原始服务器获取的。

```
http{  
  # 设置缓存的目录，并且内存中缓存区名为hot_cache，大小为128m，  
  # 三天未被访问过的缓存自动清楚，磁盘中缓存的最大容量为2GB。
  proxy_cache_path /soft/nginx/cache levels=1:2 keys_zone=hot_cache:128m inactive=3d max_size=2g;  
    
  server{  
    location / {  
      # 使用名为nginx_cache的缓存空间  
      proxy_cache hot_cache;  
      # 对于200、206、304、301、302状态码的数据缓存1天  
      proxy_cache_valid 200 206 304 301 302 1d;  
      # 对于其他状态的数据缓存30分钟  
      proxy_cache_valid any 30m;  
      # 定义生成缓存键的规则（请求的url+参数作为key）  
      proxy_cache_key $host$uri$is_args$args;  
      # 资源至少被重复访问三次后再加入缓存  
      proxy_cache_min_uses 3;  
      # 出现重复请求时，只让一个去后端读数据，其他的从缓存中读取  
      proxy_cache_lock on;  
      # 上面的锁超时时间为3s，超过3s未获取数据，其他请求直接去后端  
      proxy_cache_lock_timeout 3s;  
      # 对于请求参数或cookie中声明了不缓存的数据，不再加入缓存  
      proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;  
      # 在响应头中添加一个缓存是否命中的状态（便于调试）  
      add_header Cache-status $upstream_cache_status;  
    }  
  }  
}
```
## 防盗链设计

## 大文件传输配置
| 配置项 | 释义 |
| :--: | :--: |
| client_max_body_size |  设置请求体允许的最大体积 |
| client_header_timeout | 等待客户端发送一个请求头的超时时间 | 
| client body_timeout | 设置读取请求体的超时时间 |
| proxy_read_timeout | 设置请求被后端服务器读取时，Nginx等待的最长时间 |
| proxy_send_timeout | 设置后端向Nginx返回响应时的超时时间 |

上述配置仅是作为代理层需要配置的，因为最终客户端传输文件还是直接与后端进行交互，这里只是把作为网关层的Nginx配置调高一点，调到能够“容纳大文件”传输的程度。当然，Nginx中也可以作为文件服务器使用，但需要用到一个专门的第三方模块nginx-upload-module，如果项目中文件上传的作用处不多，那么建议可以通过Nginx搭建，毕竟可以节省一台文件服务器资源。但如若文件上传/下载较为频繁，那么还是建议额外搭建文件服务器，并将上传/下载功能交由后端处理。

## 配置SLL证书
SSL证书配置过程：
1. 先去CA机构或从云控制台中申请对应的SSL证书，审核通过后下载 Nginx 版本的证书。
2. 下载数字证书后，完整的文件总共有三个：`.crt、.key、.pem`：
 * `.crt`：数字证书文件，`.crt`是`.pem`的拓展文件，因此有些人下载后可能没有。
 * `.key`：服务器的私钥文件，及非对称加密的私钥，用于解密公钥传输的数据。
 * `.pem`：Base64-encoded编码格式的源证书文本文件，可自行根需求修改拓展名。
3. 在 Nginx 目录下新建`certificate`目录，并将下载好的证书/私钥等文件上传至该目录。
4. 最后修改一下`nginx.conf`文件即可，如下：
```
# ----------HTTPS配置-----------  
server {  
    # 监听HTTPS默认的443端口  
    listen 443;  
    # 配置自己项目的域名  
    server_name www.xxx.com;  
    # 打开SSL加密传输  
    ssl on;  
    # 输入域名后，首页文件所在的目录  
    root html;  
    # 配置首页的文件名  
    index index.html index.htm index.jsp index.ftl;  
    # 配置自己下载的数字证书  
    ssl_certificate  certificate/xxx.pem;  
    # 配置自己下载的服务器私钥  
    ssl_certificate_key certificate/xxx.key;  
    # 停止通信时，加密会话的有效期，在该时间段内不需要重新交换密钥  
    ssl_session_timeout 5m;  
    # TLS握手时，服务器采用的密码套件  
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  
    # 服务器支持的TLS版本  
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;  
    # 开启由服务器决定采用的密码套件  
    ssl_prefer_server_ciphers on;  
  
    location / {  
        ....  
    }  
}  
  
# ---------HTTP请求转HTTPS-------------  
server {  
    # 监听HTTP默认的80端口  
    listen 80;  
    # 如果80端口出现访问该域名的请求  
    server_name www.xxx.com;  
    # 将请求改写为HTTPS（这里写你配置了HTTPS的域名）  
    rewrite ^(.*)$ https://www.xxx.com;  
}
```
隐藏 Nginx 版本信息
```nginx
http {
  server_tokens  off;
}
```
PC端和移动端使用不同的项目文件映射
```nginx
server {
  ......
  location / {
    root /home/static/pc;
    if ($http_user_agent ~* '(mobile|android|iphone|ipad|phone)') {
      root /home/static/mobile;
    }
    index index.html;
  }
}
```
一个web服务，配置多个项目 (location 匹配路由区别)
```
server {
  listen                80;
  server_name           _;
  
  # 主应用
  location / {
    root          html/main;
    index               index.html;
    try_files           $uri $uri/ /index.html;
  }
  
  # 子应用一
  location ^~ /store/ {
    proxy_pass          http://localhost:8001;
    proxy_redirect      off;
    proxy_set_header    Host $host;
    proxy_set_header    X-Real-IP $remote_addr;
    proxy_set_header    X-Forwarded-For
    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
  }
  
  # 子应用二
  location ^~ /school/ {
    proxy_pass          http://localhost:8002;
    proxy_redirect      off;
    proxy_set_header    Host $host;
    proxy_set_header    X-Real-IP $remote_addr;
    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
  }
  
  # 静态资源读取不到问题处理
  rewrite ^/api/profile/(.*)$ /(替换成正确路径的文件的上一层目录)/$1 last;
}

# 子应用一服务
server {
  listen                8001;
  server_name           _;
  location / {
    root          html/store;
    index               index.html;
    try_files           $uri $uri/ /index.html;
  }
  
  location ^~ /store/ {
    alias               html/store/;
    index               index.html index.htm;
    try_files           $uri /store/index.html;
  }
  
  # 接口代理
  location  /api {
    proxy_pass          http://localhost:8089;
  }
}

# 子应用二服务
server {
  listen                8002;
  server_name           _;
  location / {
    root          html/school;
    index               index.html;
    try_files           $uri $uri/ /index.html;
  }
  
  location ^~ /school/ {
    alias               html/school/;
    index               index.html index.htm;
    try_files           $uri /school/index.html;
  }
  
  # 接口代理
  location  /api {
    proxy_pass          http://localhost:10010;
  }
}

```
配置负载均衡
```
upstream my_upstream {
  server                http://localhost:9001;
  server                http://localhost:9002;
  server                http://localhost:9003;
}

server {
  listen                9000;
  server_name           test.com;

  location / {
    proxy_pass          my_upstream;
    proxy_set_header    Host $proxy_host;
    proxy_set_header    X-Real-IP $remote_addr;
    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```
