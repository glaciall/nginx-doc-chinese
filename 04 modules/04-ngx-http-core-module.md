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































































































