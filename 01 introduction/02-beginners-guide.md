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

**注意，此命令必须与启动Nginx时是同一用户环境**

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

这己经是一个可用的配置文件了，它监听在80标准端口，而且可以在本机上使用http://localhost/ 进行访问，响应以/images/开头的请求时，服务器将会响应/data/images目录下的文件，比如，当有请求http://localhost/images/example.png 时，nginx将响应/data/images/example.png这个文件。如果此文件不存在，nginx将会响应404错误。不以/images/开头的请求将会映射到/data/www目录下。例如，http://localhost/some/example.html ，此请求nginx将会响应/data/www/some/example.html这个文件。

>To apply the new configuration, start nginx if it is not yet started or send the reload signal to the nginx’s master process, by executing:

需要应用此配置项，启动Nginx或是发送重载配置的指令到nginx的主进程，执行如下命令：
> nginx -s reload

**对于某些未预期到的一些问题，你可以从日志文件中查找原因，一般在/usr/local/nginx/logs或/var/log/nginx下查找access.log文件**

### 配置简单的代理服务器
>One of the frequent uses of nginx is setting it up as a proxy server, which means a server that receives requests, passes them to the proxied servers, retrieves responses from them, and sends them to the clients.

nginx最常用的用途是作为代理服务器，它接收请求并转发到被代理的服务器，并且将其的响应内容发送到客户端。
>We will configure a basic proxy server, which serves requests of images with files from the local directory and sends all other requests to a proxied server. In this example, both servers will be defined on a single nginx instance.

我们将配置一个基础的代理服务器，它并代理图片文件的请求到被代理的服务器上，在这个例子里，代理服务器与被代理服务器都将定义在同一个nginx实例里。

>First, define the proxied server by adding one more server block to the nginx’s configuration file with the following contents:

首先，添加如下**server**节点到nginx的配置文件中，作为被代理的服务器：
```
server {
    listen 8080;
    root /data/up1;

    location / {
    }
}
```

>This will be a simple server that listens on the port 8080 (previously, the listen directive has not been specified since the standard port 80 was used) and maps all requests to the /data/up1 directory on the local file system. Create this directory and put the index.html file into it. Note that the root directive is placed in the server context. Such root directive is used when the location block selected for serving a request does not include own root directive.

这是一个简易的服务器，监听在8080端口（因为80标准端口己经在之前的配置里使用过了而8080没有），并且映射了所有的所有的请求到/data/up1目录下。在本地文件系统上创建这个目录，并且建立一个index.html的文件。注意，**root**指令放在了**server**节点之内，这样**root**指令将拦截所有**location**未匹配的请求项。
>Next, use the server configuration from the previous section and modify it to make it a proxy server configuration. In the first location block, put the proxy_pass directive with the protocol, name and port of the proxied server specified in the parameter (in our case, it is http://localhost:8080):

接下来，使用上一段内容里的服务器配置并且稍作修改，令其成为一个代理服务器。在第一个**location**节点，写入**proxy_pass**指令，参数包括被代理服务器的协议、域名及端口（在我们的例子里，应该是http://localhost:8080 ）：
```
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location /images/ {
        root /data;
    }
}
```
>We will modify the second location block, which currently maps requests with the /images/ prefix to the files under the /data/images directory, to make it match the requests of images with typical file extensions. The modified location block looks like this:

我们现在修改第二个**location**节点，它现在映射前缀为/images/的请求到/data/images目录，为了让它匹配常规的图片文件后缀，我们修改**location**节点成如下：
```
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```

>The parameter is a regular expression matching all URIs ending with .gif, .jpg, or .png. A regular expression should be preceded with ~. The corresponding requests will be mapped to the /data/images directory.

**location**的参数是一个正则表达式，它匹配所有以.gif、.jpg、.png结尾的URI请求（正则表达式须以~开头）。如此，相应的请求都将映射到/data/images目录下。

>When nginx selects a location block to serve a request it first checks location directives that specify prefixes, remembering location with the longest prefix, and then checks regular expressions. If there is a match with a regular expression, nginx picks this location or, otherwise, it picks the one remembered earlier.

nginx首先依前缀匹配到了**location**节点，但是**location**是以最长匹配来最终决定的，所以它会测试这个正则表达式，如果相匹配，nginx将会选择这个**location**节点，否则会选用前一个。
>The resulting configuration of a proxy server will look like this:

最终，这个代理服务器的配置如下：
```
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

>This server will filter requests ending with .gif, .jpg, or .png and map them to the /data/images directory (by adding URI to the root directive’s parameter) and pass all other requests to the proxied server configured above.

>To apply new configuration, send the reload signal to nginx as described in the previous sections.

>There are many more directives that may be used to further configure a proxy connection.

这个服务器将.gif、.jpg、.png结尾的请求映射到/data/images目录（通过**root**指令添加到URI上），并且透过所有其它的请求到我们的被代理的服务器上。

需要应用这些配置项，发送**reload**信号到nginx进程（详见上文所述）。

另外还有更多的指令可以应用在代理服务器上，可令其更加强大。

### 设置FastCGI代理
>nginx can be used to route requests to FastCGI servers which run applications built with various frameworks and programming languages such as PHP.

nginx可以转发请求到具有运行FastCGI应用服务器上，比如像PHP这类拥有大量开发框架的编程语言。
>The most basic nginx configuration to work with a FastCGI server includes using the fastcgi_pass directive instead of the proxy_pass directive, and fastcgi_param directives to set parameters passed to a FastCGI server. Suppose the FastCGI server is accessible on localhost:9000. Taking the proxy configuration from the previous section as a basis, replace the proxy_pass directive with the fastcgi_pass directive and change the parameter to localhost:9000. In PHP, the SCRIPT_FILENAME parameter is used for determining the script name, and the QUERY_STRING parameter is used to pass request parameters. The resulting configuration would be:

我们使用**fastcgi_pass**指令代替**proxy_pass**来转发请求到FastCPU服务器上，**fastcgi_param**负责转发参数。假设FastCGI运行于localhost:9000，使用上一段里的代理服务器的配置，将**proxy_pass**指令替换为**fastcgi_pass**，将参数改为localhost:9000。在PHP里，*SCRIPT_FILENAME*参数用于确定php脚本文件，*QUERY_STRING*参数用于传递请求参数，配置如下：
```
server {
    location / {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param QUERY_STRING    $query_string;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

>This will set up a server that will route all requests except for requests for static images to the proxied server operating on localhost:9000 through the FastCGI protocol.

这将除了静态图片以外的所有请求都转发到运行于localhost:9000的被代理的FastCGI服务器上。






























































