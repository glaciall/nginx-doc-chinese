## 源码编绎安装nginx

编绎使用**configure**命令，它定义了系统的各方各面包括nginx的连接处理模式。在执行完成后会创建了**Makefile**，**configure**命令支持以下参数：

- --prefix=path 定义了安装目录，同时也会用于所有**configure**命令设置的其它相对路径（库源码路径除外）。默认目录为：/usr/local/nginx。
- --sbin-path=path 设置nginx可执行文件的文件名及路径（仅在安装期可用）。默认为：prefix/sbin/nginx。
- --conf-path=path 设置nginx.conf的文件名及路径。编绎后，可以通过命令行参数`-c file`来设置所使用的配置文件。默认文件为：prefix/conf/nginx.conf。
- --pid-path=path 设置nginx.pid的文件名及路径。它保存了nginx主进程的进程ID。安装后，可以通过nginx.conf配置文件的**pid**指令来设置。默认文件为：prefix/logs/nginx.pid。
- --error-log-path=path 设置错误、警告及诊断日志文件名及路径。安装后，可以通过nginx.conf的**error_log**指令来设置。默认文件为：prefix/logs/error.log。
- --http-log-path=path 设置HTTP服务器请求日志文件名及路径。安装后可以通过nginx.conf的**access_log**指令来设置。默认文件为：prefix/logs/access.log。
- --build=name 设置可选的nginx构建名称。
- --user=name 设置worker进程的运行者用户名称。安装后，可以通过nginx.conf的**user**指令来设置。默认用户为：nobody。
- --group=name 设置worker进程的运行者所属用户群组。安装后，可以通过nginx.conf的**user**指令来设置。默认情况下，群组为nobody。
- --with-select_module
- --without-select_module 启用或禁用服务器端使用`select()`方法来处理连接。此模块将在系统不支持比如kqueue、epoll、/dev/poll等更优模式下自动构建。
- --with-poll_module
- --without-poll_module 启用或禁用服务器端使用`poll()`方法来处理连接。此模块将在系统不支持kqueue、epoll、/dev/poll等更优模式下自动构建。
- --without-http_gzip_module 禁止使用HTTP服务器的gzip压缩模块。启用此模块需要**zlib**模块支持（使用yum install zlib-devel安装）。
- --without-http_rewrite_module 禁止使用HTTP服务器的URL重写模块，启用此模块需要**PCRE**模块支持（使用yum install pcre-devel安装）。
- --without-http_proxy_module 禁用**proxy**模块（将不能用于代理服务器）。
- --with-http_ssl_module 启用HTTPS支持。此模块默认不启用，需要**OpenSSL**模块支持（使用yum install openssl-devel安装）。
- --with-pcre=path 设置PCRE依赖库的源码路径。此库需要从[PCRE](http://www.pcre.org/)站点上下载并解压（版本4.4 - 8.41），接下来的事情将由./configure及**make**命令行完成。此模块在**location**指令使用正则表达式时启用。
- --with-pcre-jit 使用**just-in-time 编绎**支持来编绎PCRE依赖（1.1.12，pcre_jit指令）（**注：提高URL重写时的正则表达式性能**）。
- --with-zlib=path 使置zlib依赖库的源码路径。此库需要从[zlib](http://zlib.net/)站点下载并解压，接下来的事情将由./configure及make命令行自动完成。ngx_http_gzip_module模块依赖于此库。
- --with-cc-opt=parameters 设置能够添加到CFLAGS变量的附加参数。当在FreeBSD下使用PCRE依赖时，需要使用`-with-cc-opt="-I /usr/local/include"`来指定。如果`select()`方法支持设置文件数并且需要修改，可通过来`--with-cc-opt="-D FD_SETSIZE=2048"`指定。
- --with-ld-opt=parameters 设置链接期参数，当在FreeBSD系统下使用PCRE依赖时，需要指定`--with-ld-opt="-L /usr/local/lib"`。

配置使用样例（需要在同一行输入并回车）：
```shell
./configure
    --sbin-path=/usr/local/nginx/nginx
    --conf-path=/usr/local/nginx/nginx.conf
    --pid-path=/usr/local/nginx/nginx.pid
    --with-http_ssl_module
    --with-pcre=../pcre-8.41
    --with-zlib=../zlib-1.2.11
```

配置完成后，可使用**make**、**make install**进行编绎和安装。























