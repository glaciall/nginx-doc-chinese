## 关于nginScript
> nginScript is a subset of the JavaScript language that allows implementing location and variable handlers in http and stream. nginScript is created in compliance with ECMAScript 5.1 with some ECMAScript 6 extensions. The compliance is still evolving.

nginScript是一个JavaScript语言的子集，可以在**http**、**stream**节点下实现**location**和变量的控制，nginScript兼容ECMAScript 5.1以及部分ECMAScript 6扩展，目前兼容性正在扩展。

### 当前支持
- 布尔值、数字、字符串、对象、数组、函数及正则表达式
- ES5.1运算符、ES7求幂运算符
- ES5.1语句：var、if、else、switch、for、for in、while、do while、break、continue、return、try、catch、throw、finally
- ES6 Number及Math的属性及方法
- String成员方法
    - ES5.1：fromCharCode、concat、slice、substring、substr、charAt、charCodeAt、indexOf、lastIndexOf、toLowerCase、toUpperCase、trim、search、match、split、replace
    - ES6：fromCodePoint、codePointAt、includes、startsWith、endsWith、repeat
    - 非标准：fromUTF8、toUTF8、fromBytes、toBytes
- Object成员方法
    - ES5.1：create(不支持属性列表)、keys、defineProperty、defineProperties、getOwnPropertyDescriptor、getPropertyOf、hasOwnProperty、isPrototypeOf、preventExtensions、isExtensible、freeze、isFrozen、seal、isSealed
- Array成员方法
    - ES5.1：isArray、slice、splice、push、pop、unshift、shift、reverse、sort、join、concat、indexOf、lastIndexOf、forEach、some、every、filter、map、reduce、reduceRight
    - ES6：of、fill、find、findIndex
    - ES7：include
- ES5.1 Function方法：call、apply、bind
- ES5.1 RegExp方法：test、exec
- ES5.1 Date方法：未标明
- ES5.1 全局函数：isFinite、isNaN、parseFloat、parseInt、decodeURI、decodeURIComponent、encodeURI、encodeURIComponent

### 哪些不支持？
- ES6 let语句及const声明
- label标号
- 函数arguments数组
- evel函数
- JSON对象
- Error对象
- setTimeout、setInterval、setImmediate等定时/延时函数
- 非整形标量（.235）、二进制（0b0101）、八进制（0o77）。

### 下载和安装
nginScript目前在如下两个模块下启用：
- ngx_http_js_module
- ngx_stream_js_module

这两个模块默认都不安装，你需要通过源码编绎安装或是以Linux包的形式安装。

### 以Linux包管理器进行安装
在Linux上，有如下nginScript模块包可用：
- nginx-module-njs -- nginScript动态模块
- nginx-module-njs-dbg -- nginx-module-njs调试模块

### 从源代码进行编绎安装
nginScript源码可以用如下命令行进行克隆（需安装Mercurial）：
> hg clone http://hg.nginx.org/njs

然后使用--add-module配置安装：
> ./configure --add-module=/path-to-njs/nginx

> 注：这句应该在nginx源码目录执行，而非在njs的源码目录，需要在nginx源码编绎时声明此模块的源码路径

此模块可编绎为动态模块：
> ./configure --add-dynamic-module=path-to-njs/nginx
