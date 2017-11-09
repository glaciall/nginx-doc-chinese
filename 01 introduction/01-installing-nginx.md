## 安装Nginx
>nginx can be installed differently, depending on the operating system.
基于操作系统不同，nginx可以有多种不同的安装方式，内容如下：

### 在Linux系统上安装
>For Linux, nginx packages from nginx.org can be used.

可以在nginx.org上通过包管理器进行安装，详见[Packages](http://nginx.org/en/linux_packages.html)。

### 在FreeBSD上安装
>On FreeBSD, nginx can be installed either from the packages or through the ports system. The ports system provides greater flexibility, allowing selection among a wide range of options. The port will compile nginx with the specified options and install it.

在FreeBSD上，nginx可以通过ports或是[包管理器](http://nginx.org/en/linux_packages.html)进行安装。通过**ports**进行的安装具有更好的伸塑性，其会提供更多选择进行编绎、安装。

### 源码编绎安装
>If some special functionality is required, not available with packages and ports, nginx can also be compiled from source files. While more flexible, this approach may be complex for a beginner. For more information, see Building nginx from Sources.

如果通过ports或包管理器安装的Nginx未能满你的特殊需求，nginx同样可以通过编绎源码的方法进行安装，这将更具有可塑性，但对新手来说有一点复杂，详情可参见**源码安装**章节。