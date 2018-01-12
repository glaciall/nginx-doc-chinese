## ngx-http-core模块

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















































