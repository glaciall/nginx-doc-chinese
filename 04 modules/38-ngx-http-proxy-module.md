## ngx_http_proxy_module 模块

此模块允许传递请求到其它的服务器上。

### 配置样例

```
location /
{
    proxy_pass          http://localhost:8000;
    proxy_set_header    Host $host;
    proxy_set_header    X-Real-IP $remote_addr;
}
```

此样例将会把请求到`/`下所有的请求都转发到`http://localhost:8000`上，并且附加上`Host`及`X-Real-IP`请求头。

### 指令表

> 语法：proxy_bind address [transparent] | off;

> 默认：---

> 上下文：http, server, location

    自0.8.22版本起











































































































