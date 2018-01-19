## ngx_http_access_module模块

`ngx_http_access_module`允许对特定客户端地址进行访问权限控制。

你可以通过`password`密码、内部请求的结果、`JWT模块`来进行访问控制设置。可以使用`satisfy`指令来同时通过地址及密码进行访问控制。

### 配置样例

```
location /
{
    deny 192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny all;
}
```

这些规则按顺序进行匹配，直到第一个符合的为止。在这个例子里，只有IPv4的10.1.1.0/16及192.168.1.0/24（192.168.1.1除外）、IPv6的2001:0db8::/32的地址可以访问。对于有大量访问控制规则的配置里，使用`ngx_http_geo_module`模块来创建变量更好一些。

### 指令表

> 语法：allow address | CIDR | unix: | all;

> 默认：---

> 上下文：http, server, location, limit_except

允许指定的网络或地址的访问。如果值为`unix:`，则将允许所有UNIX域套接字的访问。

-----

> 语法：deny address | CIDR | unix: | all;

> 默认：---

> 上下文：http, server, location, limit_except

禁止指定的网络或地址的访问。如果值为`unix:`，则将禁止所有UNIX域套接字的访问。






















































