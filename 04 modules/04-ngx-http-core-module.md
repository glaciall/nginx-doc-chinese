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
当请求`/i/top.gif`时，将发送`/data/w3/images/top/gif`文件。




















