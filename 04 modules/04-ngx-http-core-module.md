## ngx-http-core模块

### HTTP请求结构及命名
因为在翻译中，可能我使用的翻译名称不当，为免引发歧义，现将HTTP请求头的命名以图例表示出来：
<img src="./img/request.png" />

> 语法：absolute_redirect on|off

> 默认：absolute_redirect on;

> 上下文：http, server, location

    1.11.8起

如果禁用它，nginx的跳转将是相对路径跳转。

-----

> 语法：aio on | off threads [=pool]

> 默认：aio off;

> 上下文：http, server, location

    0.8.11起

在FreeBSD或Linux系统上启用或禁用AIO（异步IO）：
```
location /video/ {
    aio on;
    output_buffers 1 64k;
}
```
在FreeBSD下，4.3起可使用AIO，在11.0之前，AIO可以静态链接到内核中：
```
options VFS_AIO
```
或者动态加载到内核模块里来：
```
kldload aio
```
在Linux系统里，自内核版本2.6.22起可使用AIO。同样的，你必须启用`directio`，否则读取将会被阻塞。
```
location /video/ {
    aio on;
    directio 512;
    output_buffers 1 128k;
}
```

> On Linux, directio can only be used for reading blocks that are aligned on 512-byte boundaries (or 4K for XFS). File’s unaligned end is read in blocking mode. The same holds true for byte range requests and for FLV requests not from the beginning of a file: reading of unaligned data at the beginning and end of a file will be blocking.

在Linux系统上，`directio`只能用于读取对齐到512字节的块（在XFS上是4K），未对齐的文件的读取将是阻塞模式的。同样的，对于不是从头读取FLV的请求（分段读取）时也是一样：读取一个未对齐文件数据的开头或结尾时将是阻塞模式的。`译注：这一块好饶舌`

当在Linux系统上同时启用了`sendfile`和`AIO`时，AIO将用于大于或等于由`directio`指令设置的大小的文件，`sendfile`指令用于小于此大小的文件或当`directio`禁用时启用。

```
location /video/ {
    sendfile on;
    aio on;
    directio on;
}
```
最终，文件将会以多线程的方式读取及发送（1.7.11版本起），不阻塞worker进程。

```
location /video/ {
    sendfile on;
    aio threads;
}
```

由指定的线程线来完成读取和发送文件，如果不填写线程池名称，将默认使用`default`的线程池，线程池的名称可以使用变量：
```
aio threads=pool$disk;
```

默认情况下，多线程是禁用的。需要在编绎时使用`--with-threads`参数进行配置。目前，多线程兼容于`epoll`、`kqueue`、`eventport`等连接处理模式。只有Linux系统支持多线程发送文件。

更多内容参见`sencdfile`指令。

-----

> 语法：aio_write on | off;

> 默认：aio_write off;

> 上下文：http, server, location

    1.9.13起

如果`aio`己启用，指明是否用于写文件。目前，此指令只用于己启用`aio_threads`，并且限于自被代理服务器响应的内容写入临时文件时的情景。

-----

> 语法：alias path;

> 默认：---

> 上下文：location

定义指定`location`的别名，如下例：
```
location /i/ {
    alias /data/w3/images/;
}
```
当请求`/i/top.gif`时，将发送`/data/w3/images/top/gif`文件。`path`值可以包含变量（`$document_root`和`$realpath_root`除外）。
如果在`location`中使用了正则表达式，那么`alias`可以引用此正则表达式的捕获组（0.7.40起），如：
```
location ~ ^/users/(.+\.(?:gif|jpe?g|png))$ {
    alias /data/w3/images/$1;
}
```

当`location`指令只匹配指令值的最末段时：
```
location /images/ {
    alias /data/w3/images/;
}
```
这时还不如使用`root`指令呢：
```
location /images/ {
    root /data/w3/;
}
```

-----

> 语法：chunked_transfer_encoding on | off;

> 默认：chunked_transfer_encoding on;

> 上下文：http, server, location

可在HTTP/1.1版本下禁用分段传输编码，可用于客户端未实现标准请求不支持分段传输编码时禁用掉。

-----

> 语法：client_body_buffer_size size;

> 默认：client_body_buffer_size 8k | 16k;

> 上下文：http, server, location

设置读取客户端请求体（`译注：应该是请示头区之后的数体区`）读取时的缓冲区大小。如果请求体大于缓冲区大小，则整个或部分的请求体将写入到临时文件中。默认情况下，缓冲区大小将等于两个内存页大小。在x86平台、其它的32位平台及x86-64位平台上上是8k。64位平台上通常都是16K。

-----

> 语法：client_body_in_file_only on | clean | off;

> 默认：client_body_in_file_only off;

> 上下文：http, server, location

决定了nginx是否将整个客户端请求体保存到文件中去。此指令可用于调试期、使用`$request_body_file`变量、在`ngx_http_perl_module`模块中调用`$r->request_body_file`方法。

当设置为`on`，临时文件将在请求完成后`不删除`。

当设置为`clean`时，请求时留下的临时文件将会在处理完成后自动清除。

-----

> 语法：client_body_in_single_buffer on | off;

> 默认：client_body_in_single_buffer off;

> 上下文：http, server, location

决定了Nginx是否将请求体保存到同一个缓冲区。建议在使用`$request_body`变量时使用，可减少复制操作的次数。

-----

> 语法：client_body_temp_path path [level1 [level2 [level3]]]

> 默认：client_body_temp_path client_body_temp;

> 上下文：http, server, location

指定保存客户端请求体的保存目录。可以在指定的路径下创建3级目录。如下例配置：

> client_body_temp_path /spool/nginx/client_temp 1 2;

临时文件将会保存为如下路径：

> /spool/nginx/client_temp/7/45/00000123457

-----

> 语法：client_body_timeout time;

> 默认：client_body_timeout 60s;

> 上下文：http, server, location

设置读取请求体的超时时长。此超时时长用于两个成功的读取间的超时，不是读取整个请求体的时长。如果客户端在此时间里未发送任何内容，服务器将返回408（请求超时）错误到客户端。

-----

> 语法：client_header_buffer_size size;

> 默认：client_header_buffer_size 1k;

> 上下文：http, server

设置读取请求头（header）的缓冲区大小。1K字节对于绝大部分请求己经足够了，但是如果接收到了一个包含了长cookie、或是来自于WAP客户端的请求时，就不止1K大小了。如果是请求行（第1行，GET、 HTTP/1.0这一行上）或是某一请求头的值超过了设置值，则可以使用`large_client_header_buffers`指令进行设置。

-----

> 语法：client_header_timeout time;

> 默认：client_header_timeout 60s

> 上下文：http, server

设置读取客户端请求头的超时时长。如果客户在此内容内未发送完整个请求头，则将返回408错误（请求超时）。

-----

> 语法：client_max_body_size size;

> 默认：client_max_body_size 1m;

> 上下文：http, server, location

设置在“Content-Length”请求头里指定的请求体的最大大小。如果值超过了配置值，将返回413错误（请求体过大）到客户端。注意：客户端浏览器可能不能正确的显示此错误，可设置`size`值为0来禁用此检查项。

-----

> 语法：connection_pool_size size

> 默认：connection_pool_size 256|512;

> 上下文：http, server

允许精确调整每个连接的内存分配.该指令对性能影响极小而不应该使用。在32位系统上默认为256，64位系统上默认为516字节。

    在1.9.8之前，在所有平台上默认值都为256.

-----

> 语法：default_type mime-type

> 默认：default_type text/plain

> 上下文：http, server, location

设定默认的MIME响应. 可以使用`types`指令做文件扩展名与MIME类型的映射。

-----

> 语法：directio size | off;

> 默认：directio off;

> 上下文：http, server, location

    0。7.7版本起
    
当读取大于或等于`size`指令指定大小的文件时，启用FreeBSD及Linux系统上的`O_DIRECT`标志（）、或macOS系统上的`F_NOCACHE`标注、或Solaris系统上的`directio()`方法.自0.7.15起，此指令将在使用`sendfile`指令启用处理请求时自动禁用。在处理大文件时非常有用。

> directio 4m;

或在linux系统上使用aio。

-----

> 语法：directio_alignment size;

> 默认：directio_alignment 512;

> 上下文：http, server, location

    0.8.11版本起
    
设置`directio`对齐大小。大部分情况下，512字节对齐己经够用，但是在linux下使用XFS文件系统时，需要增加到4K。

-----

> 语法：disable_symlinks off;
        disable_symlinks on | if_not_owner [from=part];

> 默认：disable_symlinks off;

> 上下文：http, server, location

    1.1.15版本起
    
决定了打开文件时的符号链接的处理方式：

off

    允许路径中的符号链接并且不检查，这是默认行为。
    
on

    如果路径中任何一段是符号链接，访问将被拒绝。
    
if_not_owner

    如果路径中任何一段是符号链接，并且链接是属于不同的属主时访问被拒绝。
    
from=part

    当检查符号链接时（参数启用并且`if_not_owner`己设定），所有部分都会被检查，使用此参数将避开设定路径开头部分的符号链接检查。如果值不是路径的开头部分或是匹配了整个路径，则整个路径都不检查。参数值可以使用变量。

例：

> disable_symlinks on from=$document_root;

这个指令仅在支持`openat()`和`fstatat()`接口的系统上，包括FreeBSD、linux以及Solaris。
参数on以及if_not_owner增加了额外的处理花销。
    
    在不支持打开目录只能搜索的系统上，使用此参数需要工作进程具有对目标目录的读权限。

    `ngx_http_autoindex_module`、`ngx_http_random_index_module`及`ngx_http_dav_module`模块上将忽略此指令。

-----

> 语法：error_page code... [=[response]] uri;

> 默认：--

> 上下文：http, server, location, if in location

设置指定错误展示的URI，可使用变量。

例：

```
error_page 404              /404.html
error_page 500 502 503 504  /50x.html
```

这将在发生错误时，内部重定向到指定的uri上，并将所有非"GET"、"HEAD"的请求方法改为"GET"。

此外，你可以通过设置`=response`参数设置响应码，例如：

> error_page 404 =200   /empty.gif;

如果错误响应是由被代理服务器或FastCGI/uwcgi/SCGI等服务，并且响应了其它不同的响应码时（如200、302、401、404等），nginx将忽略此指令设置的响应码：

> error_page 404 = /404.php;

如果没有需要去修改内部跳转的URI，可以将错误处理代理到一个命名地址去：

```
location /
{
    error_page 404 = @fallback;
}

location @fallback
{
    proxy_pass http://backend;
}
```

> 如果uri处理出错，则将返回最后一个发生的错误码至客户端。

另外，你也可以使用URL跳转来做错误处理：

```
error_page 403      http://example.com/forbidden.html;
error_page 404 =301 http://example.com/notfound.html;
```

在这个案例里，默认将返回302跳转响应码到客户端，你可以修改为（301、302、303、307、308）之中的一个。

> 307在1.1.16和1.0.13前，不被视为跳转响应码。

> 308在1.13.0前，不被视为跳转响应码。

这些指令将继承上一级的设置，并且仅当当前上下文中没有设置`error_page`指令。

-----

> 语法：etag on | off;

> 默认：etag on;

> 上下文：http, server, location

    1.3.3版本起

启用或禁用静态资源的`ETag`响应头字段的自动生成。

-----

> 语法：http { ... }

> 默认：---

> 上下文：main

提供HTTP服务器的上下文配置块。

-----

> 语法：if_modified_since off | exact | before;

> 默认：if_modified_since exact;

> 上下文：http, server, location

设置带有"If-Modified-Since"请求头的响应内容的修改时间的比较方式：

off

    "If-Modified-Since"请求头将被忽略（0.7.34）。
    
exact

    精确匹配
    
before

    响应内容的修改时间小于或等于"If-Modified-Since"请求头的时间。

-----

> 语法：ignore_invalid_headers on | off;

> 默认：ignore_invalid_headers on;

> 上下文：http, server

控制请求中的无效头是否忽略。合法的请求头名称必须是英文字母、数字、连字符-、下划线（可由`underscores_in_headers`指令设置）。

如果此指令在`server`级别上设置，值将仅用于默认服务器。设定的值将同时适用于监听于同一地址与端口的虚拟服务器。

-----

> 语法：internal;

> 默认：---

> 上下文：location

设置指定location仅用于内部请求，外部请求将响应404错误（未找到）。内部请求如下：
* 来自于`error_page`、`index`、`random_index`、`try_files`指令的跳转请求
* 来自于上游服务器使用"X-Accel-Redirect"指令进行的跳转请求
* 使用`ngx_http_ssi_module`模块的"include virtual"指令、`ngx_http_addition_module`模块、`auth_request`及`mirror`等指令所引发的子请求。

例：

```
error_page 404  /404.html
location /404.html
{
    internal;
}
```

> 为了避免由于错误配置而引发循环跳转，每个请求最多只能产生10个内部重定向。如果达到上限，将返回500内部服务器错误。你可以在nginx的错误日志里看到“rewrite or internal redirection cycle”字样。

-----

> 语法：keepalive_disable none | browser...

> 默认：keepalive_disable msie6;

> 上下文：http, server, location

在连接处理不确定的浏览器上禁用持久连接，`browser`参数用于指定需要禁用的浏览器名称。一旦接收到POST请求，值“msie6”将禁用MSIE旧版本的持久连接。值“safari”将禁用macOS及macOS系的上的Safari浏览器和基于Safari内核的浏览器的keep-alive连接。设置为`none`将在所有浏览器上禁用持久连接。

> 在1.1.18版本前，值`safari`匹配所有操作系统上的Safari浏览器及基于Safari内核的浏览器。

-----

> 语法：keepalive_requests number;

> 默认：keepalive_requests 100;

> 上下文：http, server, location

    自0.8.0版本起

设置同一个持久连接所能处理的请求的最大数量，达到上限后，连接将被关闭。

-----

> 语法：keepalive_timeout timeout [header_timeout]

> 默认：keepalive_timeout 75s;

> 上下文：http, server, location

第一个参数用于设置持久连接在服务器端的保持打开的时长。值0将禁用持久连接。第二个可选参数设置响应头中的"Keep-Alive: timeout=time"的值。两个参数值可不相同。

目前Mozilla和Konqueror浏览器可识别"Keep-Alive: timeout=time"响应头，MISE在打开连接大约60秒后自行关闭连接。

-----

> 语法：large_client_header_buffers number size;

> 默认：large_client_header_buffers 4 8K;

> 上下文：http, server

设置读取客户端过大请求头的缓冲区的数量与大小的最大值。一个请求 头行 的大小不能超过缓冲区的大小，否则将返回414（Request-URI过大）错误。同样的，任意一个请求头的大小也不能超过一个缓冲区的大小，否则将返回400（错误的请求）错误。缓冲区将按需分配，默认8K大小。如果在处理完请求后连接转为持久连接，连接所申请的缓冲区将会被释放。

-----

> 语法：limit_except method... { ... }

> 默认：---

> 上下文：location

限定地址的请求方式，`method`参数值必须是GET、HEAD、POST、PUT、DELETE、MKCOL、COPY、MOVE、OPTIONS、PROPFIND、PROPATCH、LOCK、UNLOCK、PATCH等中间的一个。允许GET请求将同时也支持HEAD请求。想要通过其它请求方式请求，可使用`ngx_http_access_module`和`ngx_http_auth_basic_module`模块进行配置：

```
limit_except GET
{
    allow 192.168.1.0/32;
    deny all;
}
```

注意：这将只允许GET和HEAD请求。

-----

> 语法：limit_rate rate;

> 默认：limit_rate 0;

> 上下文：http, server, location, if in location

设置传输到客户端的速率，单位Bps（字节/秒），设置为0将禁用速率限制。此限制是限制每个请求的速率，如果客户端同时有两个请求，限制后的总速率将是设置值的2倍。

速率限制值可以设置在`$limit_rate`变量中，这在按需限速时很有用：

```
server
{
    if ($slow)
    {
        set $limit_rate 4k;
    }
    ...
}
```

速率限制可以通过被代理的服务器的`X-Accel-Limit-Rate`响应头进行设置。此功能可以通过`prox_ignore_headers`、`fastcgi_ignore_headers`、`uwsgi_ignore_headers`、`scgi_ignore_headers`指令来关闭。

-----

> 语法：limit_rate_after size;

> 默认：limit_rate_after 0;

> 上下文：http, server, location, if in location

设置满速传输多少字节后再开始限速，例：

```
location /flv/
{
    flv;
    limit_rate_after 500k;
    limit_rate 50k;
}
```

这将在前500k满速传输，500k后将限速于50k/s。

-----

> 语法：lingering_close off | on | always

> 默认：lingering_close on;

> 上下文：http, server, location

控制nginx如何关闭客户端连接。

默认值"on"通知nginx在完全关闭前，等待并且处理来自于客户端的额外的请求数据，仅当客户端 可能 还有数据发上来时启用。

值“always”将无条件的等待以处理客户端的额外请求数据。

值“off”告诉nginx不等待额外数据并且立即关闭连接，此行为违背HTTP协议规范，通常不建议使用。

-----

> 语法：lingering_time time;

> 默认：lingering_time 30s;

> 上下文：http, server, location

当`lingering_close`生效时，此指令设置nginx读取来自于客户端额外数据的最大时长。超过时长后，连接将强制关闭，即使还有数据可读取。

-----

> 语法：lingering_timeout time;

> 默认：lingering_timeout 5s;

> 上下文：http, server, location

当`lingering_close`生效时，此指令设置客户端数据到达的最大等待时间长度。如果在这个时间里未接收到数据连接将被关闭。否则的话，读取数据，忽略超时，nginx进行新一轮的读取等待。“超时内等待 - 读取数据 - 忽略超时”不断重复，但总时长不超过`lingering_time`的设置值。

-----

> 语法：listen address[:port] [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off [keepidle]: [keepintvl]: [keepcnt]];

>       listen port [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [defferd] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]: [keepintvl]: [keepcnt]]

>       listen unix:path [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [defferd] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]: [keepintvl]: [keepcnt]]

> 默认：listen *:80 | *:8000;

> 上下文：server

设置接收请求的服务器的监听的地址、端口或UNIX域套接字，地址或端口可以任意省略。地址也可以是主机名称，例：

```
listen 127.0.0.1:8000;
listen 127.0.0.1;
listen 8000;
listen *:8000;
listen localhost:8000;
```

自0.7.36版本起，IPv6地址可通过[]来设定：
```
listen [::]:8000;
listen [::1];
```

使用`unix:`前缀来设置UNIX域套接字：
```
listen unix:/var/run/nginx.sock;
```

如果只设定了地址，则端口默认为80。
如果此指令未设置，当nginx以高权限用户运行时，将监听于`*:80`，或是`*:8000`。

如果声明了参数`default_server`，则此服务器将成为绑定在`address:port`上的默认服务器。否则，第一个声明了同一个`address:port`的服务器将成为默认服务器。

    在0.8.21版本前，可以通过default参数来设置。
    
参数`ssl`指定所有在此端口上的连接都工作于SSL（安全链路层）模式。另外，可以将服务器配置为同时处理HTTP和HTTPS请求的模式。

参数`http2`配置端口接受HTTP/2连接。通常情况下，此参数最好与`ssl`参数同时指定，当然，nginx也支持配置为不使用SSL的HTTP/2来处理请求连接。

参数`spdy`将允许在此端口上接收SPDY连接。通常情况下，此参数最好与`ssl`参数同时指定。

参数`proxy_protocol`允许此端口上的连接使用`PROXY协议`。

`listen`指令可以在进行套接字相关的系统调用时附带多个参数，这些参数可以声明于任何一个`listen`指令，但对于同一个`address:port`对只能声明一次。

setfib=number

    此参数设置结合路由表，监听套接字的FIB（SO_SETFIB选项），当前只在FreeBSD上生效。
    
fastopen=number

    启用监听套接字的`TCP Fast Open`，并且限制未完成三方握手的连接队列的最大长度。
    
backlog=number

    设置`listen()`调用时的`backlog`参数，限制连接等待的队列的最大长度。默认情况下，在FreeBSD、DragonFly BSD及macOS，`backlog`设置为-1（即无限制），其它平台上为511.
    
rcvbuf=size

    设置监听套接字的接收缓冲区大小（SO_RCVBUF选项）。

sndbuf=size

    设置监听套接字的发送缓冲区大小（SO_SNDBUF）。

accept_filter=filter

    设置监听套接字的接收过滤器的名称（SO_ACCEPTFILTER选项），用于过滤传递到`accept()`的来源连接。此选项只工作于FreeBSD及NetBSD 5.0+。值可以是`dataready`或`httpready`。

deferred

    在linux上使用延迟`accept()`（TCP_DEFER_ACCEPT选项）。

bind

    为指定的`address:port`对进行一个隔离的`bind()`调用。这很有用，当有多个`listen`指令以不同的地址同一端口进行绑定，且有一个`listen`的指令以（*:port）进行监听时，nginx只会`bind()`到`*:port`上（以上说的是同一个port的情况）。应该注意的是，`getsockname()`调用将会用于确定哪个地址来接收连接。如果`setfib`、`backlog`、`rcvbuf`、`sndbuf`、`accept_filter`、`deferred`、`ipv6only`、`so_keepalive`等己启用，系统将始终进行隔离的`bind()`调用。

ipv6only=on|off

    此参数决定了只接收指定通配符的IPv6套接字连接（通过IPV6_V6ONLY选项），或是同时支持IPv4和IPv6的连接。此参数默认开启，它仅可以在启动时设置一次。

reuseport

    此参数将命令nginx为每一个工作进程创建独立的监听套接字（使用SO_REUSEPORT选项）。允许内核调度连入的连接到不同的工作进程中去。仅适用于Linux内核3.9+以及DragonFlyBSD上。

so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]

    此参数配置监听端口的"TCP keepalive"行为。如果省略了此参数，将使用操作系统的设置。如果设置为"on"，将为套接字启用“SO_KEEPALIVE”选项。值“off”将关闭此选项。有些操作系统支持通过`TCP_KEEPIDLE`、`TCP_KEEPINTVL`、`TCP_KEEPCNT`选项来设置每个套接字的 Tcp keepalive参数(目前，只有Linux 2.4+、NetBSD 5+、FreeBSD 9.0 STABLE支持)。如果省略了一两个选项，将以系统默认选项为准。
    
```
so_keepalive=30m::10
```

将设置闲置时间到30分钟（TCP_KEEPIDLE选项），探测间隔（TCP_KEEPINTVL选项）为系统默认值，探测次数（TCP_KEEPCNT选项）为10次。

例：

```
listen 127.0.0.1 default_server accept_filter=dataready backlog=1024;
```

-----

> 语法：location [ = | ~ | ~* | ^~ ] uri { ... }

>       location @name { ... }

> 默认：---

> 上下文：server, location

根据请求URI来设置配置项。
匹配工作首先要将URI常规化，包括将“%XX”形式编码的文本解码、解析相对路径"."、".."、相邻的多个斜线合并为一个之后。
一个地址可以定义为前缀字符串、或是一个正则表达式。正则表达式可以以`~*`修饰（大小写不敏感），或是以`~`修饰（大小写敏感）。为了查找请求地址的匹配项，nginx首先检查以前缀字符串字义的匹配项，最长匹配的项将会被记下来。然后接下来查找正则表达式里的匹配项，匹配顺序将以配置文件里的出现次序挨个进行，第一个匹配到的项将作为最终使用的配置项。如果没有找到，则以之前记下的前缀匹配项来进行配置。

`location`块可以嵌套，同时有些例外情况：
对于大小写敏感的操作系统如macOS、cygwin，matching with prefix strings ignores a case (0.7.7). However, comparison is limited to one-byte locales.

正则表达式可以包含捕获组，并且可以在接下来的其它指令中引用到。

如果最长的前缀匹配有`^~`修饰符，则将不再进行正则表达式匹配检查。

同样的，使用`=`修饰符可以定义一个精准的URI匹配，如果找到了，则终止匹配搜索（不再进行其它的前缀匹配或正则表达式匹配）。例如，如果"/"的请求很频繁，那么定义一个`location = /`将会加快请求的处理，它将在第一次比例后终止匹配搜索，同时，这样的地址不允许进行嵌套`location`指令。

> 从0.7.1到0.8.41版本，如果一个请求匹配的location不包含`=`和`~`修饰符，匹配搜索依然会终止，正则表达式匹配不会进行。

让我们用几个例子来说明：

```
location = /
{
    [ 配置 A ]
}
location /
{
    [ 配置 B ]
}
location /documents/
{
    [ 配置 C ]
}
location ^~ /images/
{
    [ 配置 D ]
}
location ~* \.(gif|jpg|jpeg)$
{
    [ 配置 E ]
}
```

请求`/`将匹配配置A，请求`/index.html`将匹配配置B，`/documents/document.html`请求将匹配配置C，请求`/images/1.gif`将匹配配置D，`/documents/1.jpg`将匹配配置E。

使用`@`前缀可以定义一个命名地址。此地址不用于常规的请求处理，而是用于请求重定向。它不能被嵌套，也不能包含嵌套地址。

如果一个地址定义为前缀匹配且以`/`结尾，并且请求处理于`proxy_pass`、`fastcgi_pass`、`uwsgi_pass`、`scgi_pass`或`memcached_pass`等指令，那么将会执行特殊的处理过程。如果请求的URL满足前缀匹配，但是没有`/`结尾，Nginx将响应301永久跳转到原请求的URL并加上`/`后缀。如果这不是你所期望的，那么应该像如下形式来定义精确匹配：

```
location /user/
{
    proxy_pass http://user.example.com;
}
location = /user
{
    proxy_pass http://login.example.com;
}
```

-----

> 语法：log_not_found on | off;

> 默认：log_not_found on;

> 上下文：http, server, location

启用或禁用404未找到的请求到错误日志（见`error_log`指令）里。

-----

> 语法：log_subrequest on | off;

> 默认：log_subrequest off;

> 上下文：http, server, location

启用或禁用内部请求到访问日志（见`access_log`指令）里。

-----

> 语法：max_ranges number;

> 默认：---

> 上下文：http, server, location

    自1.1.2版本起

设置`byte-range`分段请求的最大值。请求时超过此设定值将忽略`byte-range`的值。默认情况下，nginx未对`byte-range`值作限定。设置为0将禁用`byte-range`功能。

-----

> 语法：merge_slashes on | off;

> 默认：merge_slashes on;

> 上下文：http, server

启用或禁用对URL中连续的`/`合并为一个的功能。

注意，合并是正确前缀URL及正则表达式匹配的必要手段，如果没有合并，请求`//scripts/one.php`将不会匹配如下规则：

```
location / scripts/
{
    ...
}
```

同时可能会视为静态文件来响应请求。所以需要将其转换为`/scripts/one.php`。

当URL里包含有Base64编码的时，禁用合并功能是有必要的，因为Base64编码里含有`/`符号。不过为了安全起见，最好是启用此功能。

如果此指令设定于`server`级别，此设定仅当只服务器是默认服务器时才启用，同时适用于所有监听于同一地址与端口的虚拟服务器。

-----

> 语法：msie_padding on | off;

> 默认：msie_padding on;

> 上下文：http, server, location

启用或禁用添加HTML注释到MSIE浏览器的大于400状态码的请求，将其响应内容补充到至少512字节。（译注：IE浏览器在响应错误页面时，如果服务器响应的内容长度不足，将会显示浏览器自带的错误页面。）

-----

> 语法：msie_refresh on | off;

> 默认：msie_refresh off;

> 上下文：http, server, location

为MSIE浏览器启用或禁用发送刷新以替代重定向。

-----

> 语法：open_file_cache off;

>       open_file_cache max=N [inactive=time];

> 默认：open_file_cache off;

> 上下文：http, server, location

配置缓存可以保存如下内容：
* 打开的文件描述符的大小及修改次数。
* 目录是否存在的信息。
* 文件查找错误，比如“文件未找到”、“无读取权限”等。
    错误信息的缓存应该由`open_file_cache_errors`指令分别启用。

指令拥有如下参数：

max

    设置缓存的最大个数。当缓存己满时，最近最少未使用（LRU）的将会被移除。

inactive

    设置元素未使用而被删除的闲置时间长度，默认60秒。

off

    禁用此缓存。

例：
```
open_file_cache             max=100 inactive=20x;
open_file_cache_valid       30s;
open_file_cache_min_uses    2;
open_file_cache_errors      on;
```

-----

> 语法：open_file_cache_errors on | off;

> 默认：open_file_cache_errors off;

> 上下文：http, server, location

启用或禁用文件查找错误的缓存。

-----

> 语法：open_file_cache_min_uses number;

> 默认：open_file_cache_min_uses 1;

> 上下文：http, server, location

设置由`open_file_cache`指令设置的`inactive`闲置时间内文件的最少访问次数。需要维持一个文件描述符以保持在缓存中的打开状态。

-----

> 语法：open_file_cache_valid time;

> 默认：open_file_cache_valid 60s;

> 上下文：http, server, location

设置在此时间之后，由`open_file_cache`的缓存元素应该被验证。

-----

> 语法：output_buffers number size;

> 默认：output_buffers 2 32k;

> 上下文：http, server, location

设置从磁盘上读取响应内容的缓冲区的数量、大小。

-----

> 语法：port_in_redirect on | off;

> 默认：port_in_redirect on;

> 上下文：http, server, location

启用或禁用由nginx发出的绝对跳转时指明端口。

重定向时的首选服务器名称由`server_name_in_redirect`指令来控制。

-----

> 语法：postone_ouput size;

> 默认：postone_output 1460;

> 上下文：http, server, location

如果可能，nginx将延迟发送客户端数据，直到准备了`size`指定大小的数据才开始。值设置为0将禁用延迟发送。

-----

> 语法：read_ahead size;

> 默认：read_ahead 0;

> 上下文：http, server, location

设置系统内核在处理文件时的预读数量。

在linux上，系统调用`posix_fadvise(0, 0, POSIX_FADV_SEQUENTIAL)`，所以`size`参数将被忽略。

在FreeBSD上，自FreeBSE 9.0-CURRENT版本起，系统调用`fcntl(O_READAHEAD, size)`。FreeBSD 7需要打[补丁](http://sysoev.ru/freebsd/patch.readahead.txt)来支持。

-----

> 语法：recursive_error_pages on | off;

> 默认：recursive_error_pages off;

> 上下文：http, server, location

启用或禁用使用`error_page`指令进行多次跳转。最大重定向次数同`internal`内部请求的限制，10次。

-----

> 语法：request_pool_size size;

> 默认：request_pool_size 4k;

> 上下文：http, server

允许精确调整每个请求的内存分配。此指令对性能的影响较小，不需要修改。

-----

> 语法：reset_timeout_connection on | off;

> 默认：reset_timeout_connection off;

> 上下文：http, server, location

启用或禁用重置己超时的连接。在关闭套接字前，`SO_LINGER`选项启用并且超时值为0。关闭时发送`TCP RST`到客户端，并且释放所有分配的内存，这可以避免己关闭的套接字处于长时间的`FIN_WAIT1`状态。

注意：keep-alive的持久连接将是正常关闭。

-----

> 语法：resolver address ... [valid=time] [ipv6=on|off]

> 默认：---

> 上下文：http, server, location

配置针对于上游服务器到IP地址的域名解析服务器，例如：

> resolver 127.0.0.1 [::1]:4343;

域名服务器的地址可以是域名或IP，及一个可选的端口，默认端口为53。域名服务器以round-robin轮询式进行查询。

    1.1.7版本前，只能配置一个域名服务器，自1.3。1及1.2.2版本起，域名服务器支持以IPv6的地址进行设定。

默认情况下，nginx将同时查询IPv4及IPv6地址，如果不需要IPv6地址，可以以`ipv6=off`参数来关闭它。

    自1.5.8版本起，Nginx才支持IPv6地址的解析。

nginx将以响应结果里的TTL值进行缓存，你可以使用`valid`参数来覆盖此值：

> resolver 127.0.0.1 [::1]:5353 valid=30s;

> 在1.1.9前，nginx不可调整缓存时间，其始终缓存5分钟。

> 为了避免DNS污染，推荐使用可信任的局域网络服务器或DNSS。

-----

> 语法：resolver_timeout time;

> 默认：resolver_timeout 30s;

> 上下文：http, server, location

设置域名解析超时，如：

> resolver_timeout 5s;

-----

> 语法：root path;

> 默认：root html;

> 上下文：http, server, location, if in location

设置请求的根目录，例如：

```
location /i/
{
    root /data/w3/;
}
```

文件`/data/w3/i/top.gif`将会响应`/i/top.gif`的请求。

参数`path`的值可以包含变量，除了`$document_root`和`$realpath_root`。

请求的URL路径仅仅只是追加到`root`指令指定的值的后面，如果需要修改URL，应该使用`alias`指令。

-----

> 语法：satisfy all | any;

> 默认：satisfy all;

> 上下文：http, server, location

如果一个URL由`ngx_http_access_module`、`ngx_http_auth_basic_module`、`ngx_http_auth_request_module`、`ngx_http_auth_jwt_module`等模块设定了访问控制权限，那该指令决定了是当全部符合时允许还是任意一个符合时允许访问。（即：满足所有条件或是满足任意一个条件）。

例外：

```
location /
{
    satisfy any;
    
    allow 192.168.1.0/32;
    deny all;
    
    auth_basic              "closed site";
    auth_basic_user_file    conf/htpasswd;
}
```

-----

> 语法：send_lowat size;

> 默认：send_lowat 0;

> 上下文：http, server, location

如果设定为非0值，nginx将会尝试最小化发送操作的数量，通过使用`kqueue`方法的`NOTE_LOWAT`标志或是`SO_SNDLOWAT`套接字选项。此二次都需要`size`参数值。

此指令在linux、Solaris、Windows系统上不生效。

-----

> 语法：send_timeout time;

> 默认：send_timeout 60s;

> 上下文：http, server, location

设置发送到客户端时的超时时长。此超时只用于两个成功的发送操作之间隔，不是用于整个下发。如果客户端在此时间内未接收到数据，则将关闭掉此连接。

-----

> 语法：sendfile on | off;

> 默认：sendfile off;

> 上下文：http, server, location, if in location

启用或禁用`sendfile()`。

自nginx 0.8.12及FreeBSD 5.2.1版本起，`aio`可以为`sendfile()`预加载数据。

```
location /video/
{
    sendfile    on;
    tcp_nopush  on;
    aio         on;
}
```

在这个配置样例里，以`SF_NODISKIO`标志位调用`sendfile()`，磁盘I/O将不阻塞，而是报告数据不在内存中。nginx通过读取一个字节来初始化一个异步数据加载。在第一次读取时，FreeBSD内核载入文件的前128K字节到内存中，虽然接下来的读取只会加载16K块的数据。这可以通过`read_ahead`指令进行修改。

    在1.7.11前，预载入可以由`aio sendfile`来启用。

-----

> 语法：sendfile_max_chunk size;

> 默认：sendfile_max_chunk 0;

> 上下文：http, server, location

设置为非0值，将限制单个`sendfile()`调用发送的数据量。如果没有这个限制，一个快速的连接将占用整个工作进程。

-----

> 语法：server { ... }

> 默认：---

> 上下文：http

配置一个虚拟服务器。基于IP的服务器和基于域名的服务器间没有明确的区分。`listen`指令描述了所有接收连接的用于监听的地址与端口，`server_name`指令列出了所有的服务器名称。详情可见“**nginx如何处理请求**”章节。

-----

> 语法：server_name name...;

> 默认：server_name *;

> 上下文：server

设置虚拟服务器的名称，比如：

```
server
{
    server_name example.com www.example.com;
}
```

第一个名称将作为首选服务器名称。
服务器名称可包含星号（*）来代替第一部分或最后一部分。

```
server
{
    server_name example.com *.exmaple.com www.exmaple.*;
}
```

这样的名称叫做通配名称。
其中，前两个名称可以合并成同一个。

```
server
{
    server_name .example.com;
}
```

同样的，你可以在名称前加上`~`符号来使用正则表达式匹配：

```
server
{
    server_name www.example.com ~^www\d+\.example\.com$;
}
```

正则表达式可以使用捕获组，可以在接下来的其它指令中使用。（0.7.40版本起）

```
server
{
    server_name ~^(www\.)?(.+)$
    
    location /
    {
        root /sites/$2;
    }
}

server
{
    server_name _;
    location /
    {
        root /sites/default;
    }
}
```

命名捕获组将创建相应的变量，你可以在接下来的其它指令中引用它：

```
server
{
    server_name ~^(www\.)?(?<domain>.+)$;
    
    location /
    {
        root /sites/$domain;
    }
}

server
{
    server_name _;
    
    location /
    {
        root /sites/default;
    }
}
```

如果指令的参数值设置为`$hostname`，将会用主机名（hostname）代替。

同样，你可以使用空的服务器名称：

```
server
{
    server_name www.example.com "";
}
```

这将允许服务器处理无`Host`请求头的请求，用于代替同一地址：端口上的默认服务器。

    在0.8.48版本前，默认会使用机器的主机名。

在以服务器名称进行搜索时，名称匹配优先于变量匹配（通配符及正则表达式），匹配顺序如下：

1. 精确字符串匹配。
2. 最长的前缀通配符匹配，如：`*.example.com`。
3. 最长的后缀通配符匹配，如：`mail.*`。
4. 第一个匹配的正则表达式（以配置文件中的出现顺序进行查找）。

更多关于服务器名称的描述详见“[服务器名称](http://nginx.org/en/docs/http/server_names.html)”章节。

-----

> 语法：server_name_in_redirect on | off;

> 默认：server_name_in_redirect off;

> 上下文：http, server, location

启用或禁用由nginx发起的绝对跳转时的`server_name`设定的首先服务器名称的使用。如果禁用此选项，则将使用请求头里的`Host`字段值，如果无`Host`字段，则将使用此虚拟服务器的IP。

-----

> 语法：server_name_hash_bucket_size size;

> 默认：server_name_hash_bucket_size 512;

> 上下文：http

设置服务器名称的哈希表的最大大小。

-----

> 语法：server_tokens on | off | build | string;

> 默认：server_tokens on;

> 上下文：http, server, location

启用或禁用在响应错误页面时在响应头`Server`字段里附带nginx版本号。

`build`参数将指明同时发送nginx build版本名称。

另外，自1.9.13版本起，可以使用带有变量的`string`参数来指定下发错误页面时的`Server`响应头字段信息。使用空字符串将不再下发`Server`字段。

-----

> 语法：tcp_nodelay on | off;

> 默认：tcp_nodelay on;

> 上下文：http, server, location

启用或禁用`TCP_NODELAY`选项，此选项仅当连接转变为持久连接状态时启用。

-----

> 语法：tcp_nopush on | off;

> 默认：tcp_nopush off;

> 上下文：http, server, location

启用或禁用在FreeBSD上使用`TCP_NOPUSH`选项，或是在Linux上的`TCP_CORK`选项。此选项仅在使用`sendfile`时启用，它将允许：

* 在同一个包里发送响应头以及文件的开始部分（Linux及FreeBSD 4.*）； 
* 在一个完整包里发送一个文件。

-----

> 语法：try_files file ... uri;

>       try_files file ... =code;

> 默认：---

> 上下文：server, location

在当前上下文中以指定的顺序进行文件是否存在的检查来处理请求。`file`的路由由`root`或`alias`指定的路由组合而成，如果以`/`结尾的如`$uri/`，则将检查目录是否存在。如果文件不存在，则将产生内部跳转到由`uri`参数指定的路径上。例：

```
location /images/
{
    try_files $uri /images/default.gif;
}

location = /images/default.gif
{
    expires 30s;
}
```

最后一个参数可以指向一个命名地址，如下例所示，自0.7.51版本起，最后一个参数可以是一个响应码：

```
location /
{
    try_files $uri $uri/index.html $uri.html =404;
}
```

混合代理样例：

```
location /
{
    try_files /system/maintenance.html
                $uri $uri/index.html $uri.html
                @mongrel;
}

location @mongrel
{
    proxy_pass http://mongrel;
}
```

Drupal/FastCGI样例：

```
location /
{
    try_files $uri $uri/ @drupal;
}

location ~ \.php$
{
    try_files $uri @drupal;
    
    fastcgi_pass ...;
    
    fastcgi_param SCRIPT_FILENAME   /path/to$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME       $fastcgi_script_name;
    fastcgi_param QUERY_STRING      $args;
    
    ... 其它fastcgi参数
}

location @drupal
{
    fastcgi_pass ...;
    
    fastcgi_param SCRIPT_FILENAME   /path/to/index.php;
    fastcgi_param SCRIPT_NAME       /index.php;
    fastcgi_param QUERY_STRING      q=$uri&$args;
}
```

在以下例子中：

```
location /
{
    try_files $uri $uri/ @drupal;
}
```

`try_files`等价于：

```
location /
{
    error_page 404 =@drupal;
    log_not_found off;
}
```

以及：

```
location ~ \.php$
{
    try_files $uri @drupal;
    
    fastcgi_pass ...;
    
    fastcgi_param SCRIPT_FILENAME   /path/to$fastcgi_script_name;
    
    ...
}
```

`try_files`在发送到FastCGI服务器之前选检查PHP文件是否存在。

Wordpress以及Joomla样例：

```
location / {
    try_files $uri $uri/ @wordpress;
}

location ~ \.php$ {
    try_files $uri @wordpress;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;
    ... other fastcgi_param's
}

location @wordpress {
    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to/index.php;
    ... other fastcgi_param's
}
```

-----

> 语法：types { ... }

> 默认：types { text/html html; image/gif gif; image/jpeg jpg; }

> 上下文：http, server, location

配置响应文件扩展名的MIME映射。扩展名大小写不敏感，多个扩展名可以映射到同一个类型上，例如：

```
types 
{
    application/octet-stream bin exe dll;
    application/octet-stream deb;
    application/octet-stream dmg;
}
```

nginx的完整的映射表存放于conf/mime.conf文件中。

为指定URL发送`application/octet-stream`MIME类型头，可以像如下配置：

```
location / download/
{
    types {};
    default_type application/octet-stream;
}
```

-----

> 语法：types_hash_bucket_size size;

> 默认：types_hash_bucket_size 64;

> 上下文：http, server, location

设置MIME类型的哈希表桶大小。

-----

> 语法：types_hash_max_size size;

> 默认：types_hash_max_size 1024;

> 上下文：http, server, location

设置MIME类型哈希表的最大大小。

-----

> 语法：underscores_in_headers on | off;

> 默认：underscores_in_headers off;

> 上下文：http, server

启用或禁用客户端请求头里的下划线。如果禁用下划线，则会将带有下划线的请求头交由`ignore_invalid_headers`指令进行处理。

如果指令声明在`server`级别，则仅在服务器是默认服务器时才使用该指令。指命的值也适用于监听于相同地址与端口的所有虚拟服务器。

-----

> 语法：variables_hash_bucket_size size;

> 默认：variables_hash_bucket_size 64;

> 上下文：http

设置变量的哈希表的桶大小。

-----

> 语法：variables_hash_max_size size;

> 默认：variables_hash_max_size 1024;

> 上下文：http

设置变量的哈希表的最大大小。

## 内置变量表

`ngx_http_core_module`模块支持所有Apache服务器的变量。首先，客户端请求头的每一个字段值都将表述为变量，比如：`$http_user_agent`、`$http_cookie`等等。

$arg_name

    请求行里的参数`name`，相当于参数值引用。
    
$args

    请求行里的所有参数。

$binary_rmote_addr

    二进制形式的客户端地址，IPv4地址始终是4字节，IPv6地址是16字节。

$body_bytes_sent

    发送到客户端的总字节数，不包含响应头。此变量兼容于Apache的`mod_log_config`模块的"%B"参数。

$bytes_sent

    发送到客户端的总字节数（含响应头）。

$connection

    连接序列号。

$connection_requests

    当前连接所处理的请求数。

$content_length

    `Content-Length`请求头字段值。

$cookie_name

    获取名为`name`的cookie项的值。

$document_root

    当前请求的由`root`或`alias`指定的值。

$document_uri

    同$uri;

$host

    客户端请求的服务器名称，优先级：请求行里的host名称，或是请求头里的`Host`字段，或是nginx匹配到的。

$hostname

    主机名/机器名称

$http_name

    获取请求头的字段值，变量名的最后一段由请求头字段转换而来：全部小写，中划线改为下划线。

$https

    值如果是`on`则表明当前请求是SSL模式，否则将是空字符串。

$is_args

    如果请求行有参数，值为`?`，否则为空字符串。

$limit_rate

    设置此值将启用速率限制。

$msec

    以毫秒为单位的当前时间。

$nginx_version

    nginx版本号

$pid

    工作进程的PID

$pipe

    “p” if request was pipelined, “.” otherwise。（不知道什么叫piplined）

$proxy_protocol_addr

    PROXY协议头里的客户端地址，否则为空字符串。PROXY协议必须由`listen`指令的`proxy_protocol`参数启用才行。

$proxy_protocol_port

    PROXY协议的客户端端口，否则为空字符串。

$query_string

    等同于$args

$realpath_root

    当前请求的绝对路径，所有的符号链接都将转为实际路径。

$remote_addr

    客户端IP

$remote_user

    客户端请求的由Basic认证的用户名。

$request

    原始的请求行。请求里的第一行。

$request_body

    请求体，header之后的原始内容。

$request_body_file

    请求体的临时文件名。在请求处理完成后，此文件将被删除，为了始终能够保存请求体到文件中，应该启用`client_body_in_file_only`选项。当临时文件名传递到被代理的请求里或是请求到FastCGI/uwsgi/SCGI服务器上时，应当用`proxy_pass_request_body off`、`fastcgi_pass_request_body off`、`uwsgi_pass_request_body off`、`scgi_pass_request_body off`指令来禁用传递文件名。

$request_completion

    如果请求完成，值为OK，否则为空字符串。

$request_filename

    相对于`root`或`alias`及请求URI的当前请求的路径。

$request_id

    唯一的请求标识符，16位，十六进制。

$request_length

    请求大小，包含请求行、请求头及请求体。

$reqest_method

    请求方法，一般为GET或POST。

$request_time

    以毫秒计的请求处理时长。自客户端读取的第一个字节起计时。

$request_uri

    整个原始请求URI，含参数。

$scheme

    请求协议，如http、https

$sent_http_name

    强制下发响应的头，变量的最后一段是转换后的请求头名称，全部转小写，中划线转下划线。

$sent_trailer_name

    在响应结束时发送的任意字段，变量的最后一段是转换后的请求头名称，全部转小写，中划线转下划线。

$server_addr

    接受此请求的服务器地址。为了得到此变量值通常会有系统调用的代价，应该在`listen`指令指明IP地址，并且使用`bind`参数来避免。

$server_name

    接受此请求的服务器名称。

$server_port

    接受此请求的服务器端口。

$server_protocol

    请求协议，通常是`HTTP/1.0`、`HTTP/1.1`或`HTTP/2.0`。

$status

    响应码。

$tcpinfo_rtt, $tcpinfo, $tcpinfo_snd_cwnd, $tcpinfo_rcv_space

    客户端TCP连接信息，在支持`TCP_INFO`选项的系统上可用。

$time_iso8601

    ISO 8601标准格式的本地时间。

$time_local

    标准日志格式的本地时间。

$uri

    当前请求的URI。
    此变量的值在处理中可能会发生变化，比如在内部转跳或是使用索引文件时。


