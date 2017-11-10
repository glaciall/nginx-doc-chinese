## 管理nginx

>nginx can be controlled with signals. The process ID of the master process is written to the file /usr/local/nginx/logs/nginx.pid by default. This name may be changed at configuration time, or in nginx.conf using the pid directive. The master process supports the following signals:

nginx可以使用信号进行管理。主进程的PID默认情况下会被写入/usr/local/nginx/logs/nginx.pid，文件名可以在配置时进行修改，也可以在nginx.conf使用**pid**指令进行定义。主进程支持以下信号：
```
TERM, INT   fast shutdown
QUIT        graceful shutdown
HUP         changing configuration, keeping up with a changed time zone (only for FreeBSD and Linux), starting new worker processes with a new configuration, graceful shutdown of old worker processes
USR1	        re-opening log files
USR2        upgrading an executable file
WINCH       graceful shutdown of worker processes
```

```
TERM, INT       快速退出
QUIT            完出退出（处理完当前所有请求）
HUP             更新配置，维护一个己修改的时间区（仅在FreeBSD和Linux下），使用新的配置开启新的worker进程，安全退出旧的worker进程
USR1            重新打开日志文件
USR2            升级可执行文件
WINCH           安全退出worker进程
```

>Individual worker processes can be controlled with signals as well, though it is not required. The supported signals are:

每一个独立的worker进程也可以使用信号进行控制（尽管可能是不那么需要），支持的信号如下：
```
TERM, INT	fast shutdown
QUIT	graceful shutdown
USR1	re-opening log files
WINCH	abnormal termination for debugging (requires debug_points to be enabled)
```

```
TERM, INT       快速退出
QUIT            安全退出
USR1            重新打开日志文件
WINCH           中断进行调试（需要启用debug_points）
```

### 修改配置
>In order for nginx to re-read the configuration file, a HUP signal should be sent to the master process. The master process first checks the syntax validity, then tries to apply new configuration, that is, to open log files and new listen sockets. If this fails, it rolls back changes and continues to work with old configuration. If this succeeds, it starts new worker processes, and sends messages to old worker processes requesting them to shut down gracefully. Old worker processes close listen sockets and continue to service old clients. After all clients are serviced, old worker processes are shut down.
 
为了让nginx重新读取配置文件，需要把HUP信号发送到主进程，主进程首先进行配置文件的语法检查，然后试着应用新的配置，也就是打开日志文件以及新的sockets监听。如果失败了，将回滚修改并且继续使用旧的配置项，如果成功了，将会启用新的worker进程，并且通知旧的worker进程安全退出。旧的worker进程关闭sockets连接不再接收新请求，直接处理完旧的客户请求，当所有的处理完成，旧的worker进程也就随之退出。

>Let’s illustrate this by example. Imagine that nginx is run on FreeBSD 4.x and the command

我们用例子来说明，想像一下一个运行于FreeBSD 4.x上的Nginx，在控制台输入如下：
>ps axw -o pid,ppid,user,%cpu,vsz,wchan,command | grep '(nginx|PID)'

上面输出：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
 33126     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
 33127 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
 33128 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
 33129 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
```

如果向主进程发送了HUP信号，则上面的输出将变成：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33129 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```

其中的PID为33129的worker进程将继续运行，片刻之后退出，上面的命令行输出：
```
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```































