## 核心功能

### 配置样例

```
user www www;
worker_processes 2;

error_log /varlog/nginx-error.log info;

events
{
    use kqueue;
    worker_connections 2048;
}
```

### 指令表

> 语法：accept_mutex on | off;

> 默认：accept_mutex off;

> 上下文：events

如果启用`accept_mutex`，工作进程将轮流接受新的连接。否则，Nginx将通知所有工作进程有新的连接，并且如果新连接太少的话，某些工作进程将会浪费系统资源。

    在支持`EPOLLEXCLUSIVE`标志的系统上或使用`reuseport`时，没有必要启用`accept_mutex`。
    
    在1.11.3版本前，默认值为on。

-----

> 语法：accept_mutex_delay time;

> 默认：accept_mutex_delay 500ms;

> 上下文：events

当启用`accept_mutex`时，如果其它的工作进程正在接受新连接，当前工作进程重新接收新的连接的最大间隔时间。

-----

> 语法：daemon on | off;

> 默认：daemon on;

> 上下文：main

决定Nginx是否以守护进程的模式运行，大多用于扩展开发期。

-----

> 语法：debug_connection address | CIDR | unix:

> 默认：---

> 上下文：events

启用对指定连接的调试日志，其它连接将以`error_log`指令指定的日志级别来记录。需要调试的连接可以IPv4、IPv6、主机名等形式描述。`unix:`参数将输出所有通过UNIX域套接字的连接调试信息。

```
events {
    debug_connection 127.0.0.1;
    debug_connection localhost;
    debug_connection 192.0.2.0/24;
    debug_connection ::1;
    debug_connection 2001:0db8::/32;
    debug_connection unix:;
    ...
}
```

    需要使用此指令，需要在以`--with-debug`参数来编绎nginx。

-----

> 语法：debug_points abort | stop;

> 默认：---

> 上下文：main

用于调试。

当内部错误如工作进程重启时的socket套接字泄露，启用`debug_points`指令将导致创建`core`文件（值为abort时），或是停止进程（值为stop时）以调用系统调试器进行进一步的分析。

-----

> 语法：error_log file [level];

> 默认：error_log logs/error.log error;

> 上下文：main, http, mail, stream, server, location

日志配置。在同一个级别上可以设置多个日志。如果在`main`级别上未明确定义错误日志，则将输出到默认日志文件里去。

第一个参数`file`将指明日志文件保存路径。使用值`stderr`将记录到标准错误流中。使用`syslog:`前缀将输出到[syslog](http://nginx.org/en/docs/syslog.html)中。使用`memory:`前缀将输出到循环缓冲区中（一般用于开发调试过程）。

第二个参数指定日志级别，可以是`debug`、`info`、`notice`、`warn`、`error`、`crit`、`alert`、`emerg`等值，上述值的严重程度将顺序递增，每一个级别都会将其后级别的日志都记录下来，比如`error`级别将同时记录`crit`、`alert`、`emerg`三个级别的日志。忽略将参数时值默认为`error`。

    自1.7.11版本起，可在stream级别上使用，在1.9.0版本起，可在mail级别上使用。

-----

> 语法：env variable[=value];

> 默认：env TZ;

> 上下文：main

默认情况下，nginx将移除除了变量`TZ`以外的其它所有继承自上级进程的环境变量。此指令允许保留、修改一些继承下来的变量或是创建一些新的环境变量，例如：

* 继承自可执行文件热更新过程中的变量
* `ngx_http_perl_module`模块所使用的
* 由工作进程使用。需要注意的是，通过此方式控制系统库并不总是可行的，因为系统库仅在初始化时检查变量。热更新可执行文件除外。

变量`TZ`始终继承并且在`ngx_http_perl_module`模块中可用，即使未明确配置。

使用样例：

```
env MALLOC_OPTIONS;
env PERL5LIB=/data/site/modules;
env OPENSSL_ALLOW_PROXY_CERTS=1;
```

    NGINX环境变量只是内部使用，不应当由用户直接设置。

-----

> 语法：events { ... }

> 默认：---

> 上下文：main

提供连接处理的配置上下文。

-----

> 语法：include file | mask;

> 默认：---

> 上下文：任意

引入其它文件到当前配置里来。可使用通配符。被包含的文件会被替换进来，并且进行语法检查。

使用样例：

```
include mime.types;
include vhosts/*.conf;
```

-----

> 语法：load_module file;

> 默认：---

> 上下文：main

    自1.9.11版本起

加载动态模块。

样例：

```
load_module modules/ngx_mail_module.so;
```

-----

> 语法：lock_file file;

> 默认：lock_file logs/nginx.lock;

> 上下文：main

nginx使用锁机制来实现`accept_mutex`和连序访问共享内存。大多数系统上使用原子操作来实现，届时此指令将会被忽略，其它系统上将使用文件锁。此指令指定锁文件名的前缀。

-----

> 语法：master_process on | off;

> 默认：master_process on;

> 上下文：main

决定是否启动工作进程，此指令为开发者所准备。

-----

> 语法：multi_accept on | off;

> 默认：multi_accept off;

> 上下文：events

如果`multi_accept`己禁用，工作进程同时只接受一个连接。否则将同时接受多个连接。

    此指令在使用`kqueue`处理连接时将被忽略，因为它报告了需要被接受的连接数量。

-----

> 语法：pcre_jit on | off;

> 默认：pcre_jit off;

> 上下文：main

    自1.1.12版本起

启用或禁用配置解析时的正则表达式启用即时编绎，它将对正则匹配起到很明显的加速作用。

    自8.20版本起，编绎期使用`--enable-jit`参数来启用即时编绎。如果PCRE库是通过`--with-pcre=`参数来指定时，可通过`--with-pcre-jit`参数来启用即时编绎。

-----

> 语法：pid file;

> 默认：pid logs/nginx.pid;

> 上下文：main

定义主进程的PID文件保存路径。

-----

> 语法：ssl_engine device;

> 默认：---

> 上下文：main

设置硬件SSL加速器的名称。

-----

> 语法：thread_pool name threads=number [max_queue=number];

> 默认：thread_pool default threads=32 max_queue=65535;

> 上下文：main

    自1.7.11版本起

定义不阻塞工作进程的用于读取或发送文件的命名线程池。

参数`threads`定义线程池的线程数。

当线程池里线程全部处于工作中时，新的任务将处于等待队列，参数`max_queue`限定用于等待的队列的最大长度，默认最多65535个等待。当超过最大等待后，任务将视为错误而结束。

-----

> 语法：timer_resolution interval;

> 默认：---

> 上下文：main

用于减少系统函数`gettimeofday()`方法的调用。默认情况下，每次接收到内核事件时都会调用`gettimeofday()`，如果设置了此值，则在指定的时间间隔内只调用一次`gettimeofday()`方法。例如：

```
timer_reslution 100ms;
```

定义器的内部实现依赖于：
* 如果使用kqueue则使用`EVFILT_TIMER`。
* 如果使用eventport则使用`timer_create()`。
* 否则使用`settimer()`。

-----

> 语法：use method;

> 默认：---

> 上下文：events

设置连接处理方式。一般来说不需要明确设置此值，nginx将自动选择最优的处理模式。

-----

> 语法：user user [group];

> 默认：user nobody nobody

> 上下文：main

设置运行工作进程的用户及用户组。如果忽略`group`参数，将使用同名于`user`的用户组。

-----

> 语法：worker_aio_requests number;

> 默认：worker_aio_requests 32;

> 上下文：events

当使用epoll连接处理模式的aio方法时，设置单个工作进程未完成的异步IO操作的最大数量。

-----

> 语法：worker_connections number;

> 默认：worker_connections 512;

> 上下文：events

设置一个工作进程能够同时打开的最大连接数。

需要注意的是，此数字包括了到被代理服务器的连接及其它，不仅仅是客户端的连接数。同时此值不能超过最大打开文件数（可通过`worker_rlimit_nofile`指令修改）。

-----

> 语法：worker_cpu_affinity cpumask ...;

>       worker_cpu_affinity auto [cpumask];

> 默认：---

> 上下文：main

绑定工作进程到具体CPU内核上。CPU内核以位掩码来表示。如果启用此指令，则应该为每一个工作进程都单独指令CPU。默认情况下，工作进程不绑定到任何一个CPU内核上。

例：

```
worker_processes        4;
worker_cpu_affinity     0001 0010 0100 1000;
```

再比如：

```
worker_processes        2;
worker_cpu_affinity     0101 1010;
```

这将第一个工作进程绑定到CPU0和CPU2上，第二个工作进程绑定到CPU1和CPU3上。此适用于支持超线程的CPU。

值`auto`将允许nginx自动绑定工作进程到可用的CPU上：

```
worker_processes        auto;
worker_cpu_affinity     auto;
```

可选的`mask`参数可用于自动绑定时的CPU可用性（即只有哪些CPU内核才检查可用性，跳过为0的CPU内核）。

```
worker_cpu_affinity auto 01010101;
```

    此指令只在FreeBSD及linux上可用。

-----

> 语法：worker_priority number;

> 默认：worker_priority 0;

> 上下文：main

定义工作进程的调度优先级，类似于`nice`命令工具，负值意味着更高的优先级，可允许的值范围为：-20到20。

例如：

```
worker_priority -10;
```

-----

> 语法：worker_processes number | auto;

> 默认：worker_processes 1;

> 上下文：main

定义工作进程的数量。

最佳值取决于CPU内核数（但并不限于）、存储数据的磁盘数以及负载模式。如果不确定，设置为可用的CPU内核数是个好的开始（值`auto`将自动检测）。

    参数值`auto`自1.3.8和1.2.5版本起支持。

-----

> 语法：worker_rlimit core size;

> 默认：---

> 上下文：main

修改工作进程的最大core文件(RLIMIT_CORE)大小的限制。用于在不重启主进程的情况下增加限制的大小。

-----

> 语法：worker_rlimit_nofile number;

> 默认：---

> 上下文：main

修改工作进程的最大打开文件数量（RLIMIT_NOFILE）。用于在不重启主进程的情况下修改限制的大小。

-----

> 语法：worker_shutdown_timeout time;

> 默认：---

> 上下文：main

    自1.11.11版本起

配置安全退出工作进程的超时时长。当超时时，Nginx将尝试关闭所有打开的连接来且于关闭进程。

-----

> 语法：working_directory directory;

> 默认：---

> 上下文：main

设置工作进程的当前工作目录。主要用于写入CORE文件时，注意目标目录有无写权限。
