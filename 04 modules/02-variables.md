## Nginx内置变量表

$arg_name
> 以`name`为名称的请求行里的参数值。

$args
> 请求行里的所有参数。

$binary_rmote_addr
> 二进制形式的客户端地址，IPv4地址始终是4字节，IPv6地址是16字节。

$body_bytes_sent
> 发送到客户端的总字节数，不包含响应头。此变量兼容于Apache的`mod_log_config`模块的"%B"参数。

$bytes_sent

> 发送到客户端的总字节数（含响应头）。

$connection

> 连接序列号。

$connection_requests

> 当前连接所处理的请求数。。

$content_length

> `Content-Length`请求头字段值。

$cookie_name

> 获取名为`name`的cookie项的值。

$document_root

> 当前请求的由`root`或`alias`指定的值。

$document_uri

> 同$uri;

$host

> 客户端请求的服务器名称，优先级：请求行里的host名称，或是请求头里的`Host`字段，或是nginx匹配到的。

$hostname

> 主机名/机器名称

$http_name

> 获取请求头的字段值，变量名的最后一段由请求头字段转换而来：全部小写，中划线改为下划线。

$https

> 值如果是`on`则表明当前请求是SSL模式，否则将是空字符串。

$is_args

> 如果请求行有参数，值为`?`，否则为空字符串。

$limit_rate

> 设置此值将启用速率限制。

$msec

> 以毫秒为单位的当前时间。

$nginx_version

> nginx版本号

$pid

> 工作进程的PID

$pipe

> 值如果是`p`则表示管线式请求，否则为`.`。
> 管线化：将多个HTTP请求（request）整批提交的技术，而在发送过程中不需先等待服务端的回应。
> 参见：[wiki: HTTP管线化](https://zh.wikipedia.org/wiki/HTTP%E7%AE%A1%E7%B7%9A%E5%8C%96)

$proxy_protocol_addr

> PROXY协议头里的客户端地址，否则为空字符串。PROXY协议必须由`listen`指令的`proxy_protocol`参数启用才行。

$proxy_protocol_port

> PROXY协议的客户端端口，否则为空字符串。

$query_string

> 等同于$args

$realpath_root

> 当前请求的绝对路径，所有的符号链接都将转为实际路径。

$remote_addr

> 客户端IP

$remote_user

> 客户端请求的由Basic认证的用户名。

$request

> 原始的请求行。请求里的第一行。

$request_body

> 请求体，header之后的原始内容。

$request_body_file

> 请求体的临时文件名。在请求处理完成后，此文件将被删除，为了始终能够保存请求体到文件中，应该启用`client_body_in_file_only`选项。当临时文件名传递到被代理的请求里或是请求到FastCGI/uwsgi/SCGI服务器上时，应当用`proxy_pass_request_body off`、`fastcgi_pass_request_body off`、`uwsgi_pass_request_body off`、`scgi_pass_request_body off`指令来禁用传递文件名。

$request_completion

> 如果请求完成，值为OK，否则为空字符串。

$request_filename

> 相对于`root`或`alias`及请求URI的当前请求的路径。

$request_id

> 唯一的请求标识符，16位，十六进制。

$request_length

> 请求大小，包含请求行、请求头及请求体。

$reqest_method

> 请求方法，一般为GET或POST。

$request_time

> 以毫秒计的请求处理时长。自客户端读取的第一个字节起计时。

$request_uri

> 整个原始请求URI，含参数。

$scheme

> 请求协议，如http、https

$sent_http_name

> 强制下发响应的头，变量的最后一段是转换后的请求头名称，全部转小写，中划线转下划线。

$sent_trailer_name

> 在响应结束时发送的任意字段，变量的最后一段是转换后的请求头名称，全部转小写，中划线转下划线。

$server_addr

> 接受此请求的服务器地址。为了得到此变量值通常会有系统调用的代价，应该在`listen`指令指明IP地址，并且使用`bind`参数来避免。

$server_name

> 接受此请求的服务器名称。

$server_port

> 接受此请求的服务器端口。

$server_protocol

> 请求协议，通常是`HTTP/1.0`、`HTTP/1.1`或`HTTP/2.0`。

$status

> 响应码。

$tcpinfo_rtt, $tcpinfo, $tcpinfo_snd_cwnd, $tcpinfo_rcv_space

> 客户端TCP连接信息，在支持`TCP_INFO`选项的系统上可用。

$time_iso8601

> ISO 8601标准格式的本地时间。

$time_local

> 标准日志格式的本地时间。

$uri

> 当前请求的URI。
> 此变量的值在处理中可能会发生变化，比如在内部转跳或是使用索引文件时。
