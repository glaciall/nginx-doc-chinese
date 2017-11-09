## 新手手册
>This guide gives a basic introduction to nginx and describes some simple tasks that can be done with it. It is supposed that nginx is already installed on the reader’s machine. If it is not, see the Installing nginx page. This guide describes how to start and stop nginx, and reload its configuration, explains the structure of the configuration file and describes how to set up nginx to serve out static content, how to configure nginx as a proxy server, and how to connect it with a FastCGI application.

此章节提供关于nginx的基础介绍，以完成一些简单的任务，首先确保你的机器上己经安装了nginx（可参考安装章节），本文介绍了如何启动、停止、重载配置、介绍配置文件结构及配置一个提供静态内容页、如何把nginx配置为代理服务器、如何与FastCGI配合工作。

>nginx has one master process and several worker processes. The main purpose of the master process is to read and evaluate configuration, and maintain worker processes. Worker processes do actual processing of requests. nginx employs event-based model and OS-dependent mechanisms to efficiently distribute requests among worker processes. The number of worker processes is defined in the configuration file and may be fixed for a given configuration or automatically adjusted to the number of available CPU cores (see worker_processes).

nginx有一个主进程及多个worker进程，主进程负责读取、解析配置文件及维持worker进程。worker进程负责实际的请求处理。nginx使用事件模型和OS相关的相制去高效的调度请求与worker进程，worker进程的数量由配置文件决定，或是自动的由CPU核数决定（详见**worker_processes**）。

>The way nginx and its modules work is determined in the configuration file. By default, the configuration file is named nginx.conf and placed in the directory /usr/local/nginx/conf, /etc/nginx, or /usr/local/etc/nginx.

nginx及其模块的功能全部由配置文件进行定义，一般情况下，配置文件的路径默认是
1. /usr/local/nginx/conf
2. /etc/nginx
3. /usr/local/etc/nginx

### 启动、停止、重载配置
>To start nginx, run the executable file. Once nginx is started, it can be controlled by invoking the executable with the -s parameter. Use the following syntax:

直接执行nginx可执行文件即可启动Nginx，一旦启动完成，可通过可执行文件的-s参数进行通信，一般语法如下：
> nginx -s *signal*
sinal信号可以是以下几种：
1. stop - 快速退出
2. quit - 安全退出
3. reload - 重载配置文件
4. reopen - 重新打开日志文件

>For example, to stop nginx processes with waiting for the worker processes to finish serving current requests, the following command can be executed:

比如，在停止nginx进程前，等待worker进程处理完当前所有的请求，可以通过如下指令：
> nginx -s quit
==注意，此命令必须与启动Nginx时是同一用户环境==

>Changes made in the configuration file will not be applied until the command to reload configuration is sent to nginx or it is restarted. To reload configuration, execute:

配置文件修改后并不会立即生效，除非发送了重载指令，或是重启了nginx，发送重载指令如下：
> nginx -s reload

>Once the master process receives the signal to reload configuration, it checks the syntax validity of the new configuration file and tries to apply the configuration provided in it. If this is a success, the master process starts new worker processes and sends messages to old worker processes, requesting them to shut down. Otherwise, the master process rolls back the changes and continues to work with the old configuration. Old worker processes, receiving a command to shut down, stop accepting new connections and continue to service current requests until all such requests are serviced. After that, the old worker processes exit.

一量主进程接收到了重载配置的指令，将会先进行配置文件的语法检查，然后应用新的配置项，如果成功，主进程将启动新的worker进程并且发送消息到worker请求退出进程。否则（配置文件有错）将会回滚并使用旧的配置项，旧的worker进程将会收到指令关闭，停止处理新连接，并且完成当前的连接请求直到全部完成，最后worker进程全部退出。

>A signal may also be sent to nginx processes with the help of Unix tools such as the kill utility. In this case a signal is sent directly to a process with a given process ID. The process ID of the nginx master process is written, by default, to the nginx.pid in the directory /usr/local/nginx/logs or /var/run. For example, if the master process ID is 1628, to send the QUIT signal resulting in nginx’s graceful shutdown, execute:

在*inx系统上，你还可以使用kill命令行工具发送指令，这需要通过nginx的主进程的PID进行发送，默认情况下，PID进程号会被写入到文件中，比如/usr/local/nginx/logs或/var/run下面。假如主进程的ID是1628，发送一个QUIT指令令其安全退出，执行：
> kill -s QUIT 1628

>For getting the list of all running nginx processes, the ps utility may be used, for example, in the following way:

使用ps命令来列出运行中的nginx进程，例如：
> ps -ax | grep nginx

> For more information on sending signals to nginx, see Controlling nginx.

想了解更多的nginx指令信号，参见**nginx控制**章节

### 配置文件的结构
> An important web server task is serving out files (such as images or static HTML pages). You will implement an example where, depending on the request, files will be served from different local directories: /data/www (which may contain HTML files) and /data/images (containing images). This will require editing of the configuration file and setting up of a server block inside the http block with two location blocks.

WEB服务器的一个重要功能是提供文件浏览（比如图片、静态HTML页面），你可以实现一个基于请求的案例，文件都放在不同的目录，/data/www（这里存放了些HTML文件），/data/images（图片文件），这需要修改配置文件在**http**下建立一个**server**节点，包含有两个**location**节点。

>First, create the /data/www directory and put an index.html file with any text content into it and create the /data/images directory and place some images in it.

> Next, open the configuration file. The default configuration file already includes several examples of the server block, mostly commented out. For now comment out all such blocks and start a new server block:

首先，创建/data/www目录，创建index.html并写入一些文本内容，并在/data/images目录下存放一些图片文件。
然后，打开配置文件，默认的配置文件己经有一些server节点的样例了（大部分被注释掉了），现在创建一个新的server节点开始：
```
http {
	server {
    
    }
}
```

> Generally, the configuration file may include several server blocks distinguished by ports on which they listen to and by server names. Once nginx decides which server processes a request, it tests the URI specified in the request’s header against the parameters of the location directives defined inside the server block.
> Add the following location block to the server block:

通常情况下，配置文件会有多个server节点，通过**listen**与**server_name**所指定的端口与域名进行区分，nginx在得知某一请求由哪个server节点处理一个请求，接下来将会通过**location**指令所定义的去拦截请示中的URI参数来处理具体的请求。

添加**location**节点到**server**节点之下：
```
location / {
	root /data/www;
}
```

>This location block specifies the “/” prefix compared with the URI from the request. For matching requests, the URI will be added to the path specified in the root directive, that is, to /data/www, to form the path to the requested file on the local file system. If there are several matching location blocks nginx selects the one with the longest prefix. The location block above provides the shortest prefix, of length one, and so only if all other location blocks fail to provide a match, this block will be used.

**location**节点声明了"/"前缀以对比请求中的URI，如果满足匹配内容，URI将会加到**root**指定的路径上，然后去请求本地文件系统上的文件。如果URI能够匹配多个**location**节点，那么nginx将会选择最长匹配的**location**前缀。上面的**location**指令使用了最短的前缀，除非有其它的**location**节点拦截了请求，否则这个节点都会启用。

> Next, add the second location block:

接下来，添加第二个**location**节点：
```
location /images/ {
	root /data;
}
```

>It will be a match for requests starting with /images/ (location / also matches such requests, but has shorter prefix).

这一段将会匹配由/images/开头的请求（"/"依然会匹配这个请求，但是它的前缀更短）。

>The resulting configuration of the server block should look like this:

现在整体的配置大致如下：
```
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```

>This is already a working configuration of a server that listens on the standard port 80 and is accessible on the local machine at http://localhost/. In response to requests with URIs starting with /images/, the server will send files from the /data/images directory. For example, in response to the http://localhost/images/example.png request nginx will send the /data/images/example.png file. If such file does not exist, nginx will send a response indicating the 404 error. Requests with URIs not starting with /images/ will be mapped onto the /data/www directory. For example, in response to the http://localhost/some/example.html request nginx will send the /data/www/some/example.html file.

这己经是一个可用的配置文件了，它监听在80标准端口，而且可以在本机上使用http://localhost进行访问，响应以/images/开头的请求时，服务器将会响应/data/images目录下的文件，比如，当有请求http://localhost/images/example.png时，nginx将响应/data/images/example.png这个文件。如果此文件不存在，nginx将会响应404错误。不以/images/开头的请求将会映射到/data/www目录下。例如，http://localhost/some/example.html，此请求nginx将会响应/data/www/some/example.html这个文件。

>To apply the new configuration, start nginx if it is not yet started or send the reload signal to the nginx’s master process, by executing:

需要应用此配置项，启动Nginx或是发送重载配置的指令到nginx的主进程，执行如下命令：
> nginx -s reload
==对于某些未预期到的一些问题，你可以从日志文件中查找原因，一般在/usr/local/nginx/logs或/var/log/nginx下查找access.log文件==

### 配置简单的代理服务器
