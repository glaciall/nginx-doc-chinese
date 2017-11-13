## 连接处理模式
> nginx supports a variety of connection processing methods. The availability of a particular method depends on the platform used. On platforms that support several methods nginx will normally select the most efficient method automatically. However, if needed, a connection processing method can be selected explicitly with the use directive.
  
nginx支持一系列的连接处理模式，一些独有的处理模式取决于操作系统。如果操作系统同时支持多种，nginx将会自动选择最优模式。当然，如果你也可以使用**use**指令来指定。

> The following connection processing methods are supported:
  
nginx共支持如下模式：

- select — standard method. The supporting module is built automatically on platforms that lack more efficient methods. The --with-select_module and --without-select_module configuration parameters can be used to forcibly enable or disable the build of this module.
  
- select - 标准模式。当系统不支持更优模式时将自动编绎进来。可以使用**--with-select_module**或**--without-select_module**参数在编绎时强制启用或禁用此模块。

- poll — standard method. The supporting module is built automatically on platforms that lack more efficient methods. The --with-poll_module and --without-poll_module configuration parameters can be used to forcibly enable or disable the build of this module.
  
- poll - 标准模式。当系统不支持更优模式时将自动编绎进来。可以使用**-with-poll_module**或**-without-poll_module**参数在编绎时强制启用或禁用此模块。

- kqueue — efficient method used on FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0, and macOS.
  
- kqueue - 在FreeBSD 4.1+、OpenBSD 2.9+、NetBSD 2.0、macOS上有效。

- epoll — efficient method used on Linux 2.6+.
  
- epoll - 在Linux 2.6+以上有效。

> The EPOLLRDHUP (Linux 2.6.17, glibc 2.8) and EPOLLEXCLUSIVE (Linux 4.5, glibc 2.24) flags are supported since 1.11.3.
> Some older distributions like SuSE 8.2 provide patches that add epoll support to 2.4 kernels.

> EPOLLRDHUP(需要Linux 2.6.17及glibc 2.8)及EPOLLEXCLUSIVE标记在1.11.3起才支持
> 在一些于的发行版（如SuSE 8.2）可在Linux内核2.4版本下安装补丁可增加支持epoll

- /dev/poll — efficient method used on Solaris 7 11/99+, HP/UX 11.22+ (eventport), IRIX 6.5.15+, and Tru64 UNIX 5.1A+.
  
- /dev/poll - 在Solaris 7.11/99+，HP/UX 11.22+(eventport)、IRIX 6.5.15+及Tru64 UNIX 5.1A+上有效

- eventport — event ports, method used on Solaris 10+ (due to known issues, it is recommended using the /dev/poll method instead).

- eventport - Solaris 10+有效（因为一些未知的问题，推荐使用/dev/poll来代替）






























