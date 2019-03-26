---
title: Dockerfile指令总结
date: 2019-03-26 20:50:50
tags: Dockerfile
---

这里根据一些常用的dockerfile指令，通过查询官网文档、翻译总结了一些常用的dockerfile语法。

#### From

首先Dockerfile必须由`from`指令开头（除非有指令解析器或者ARG），from指令表示了这个镜像（image）的一个***Base Image***（这些可以从[Public Repositories](https://hub.docker.com/)里面拉下来），这里基础镜像可以当作是一种继承的机制。一般有3种写法

```dockerfile
FROM <image> [AS <name>]
```

```dockerfile
FROM <image>[:<tag>] [AS <name>]
```

```dockerfile
FROM <image>[@<digest>] [AS <name>]
```

* `ARG`指令可以在`FROM`之前。

  `FROM`可以支持在其之前声明的任何`ARG`指令，但在`FROM`后面它的变量就不能被其它指令所使用，除非用默认的`ARG`再声明一次。例子：

  ```dockerfile
  ARG VERSION=latest
  FROM busybox:$VERSION
  ARG VERSION
  RUN echo $VERSION > image_version
  ```

* `FROM`可以在一个`Dockerfile`里面出现多次来创造多个镜像或者用一个构建(stage)阶段来作为另一个镜像的依赖

* `name`是一个可选项，他能给一个构建阶段命名。以后能够用在`FROM`和`COPY --from=<name|index>`指令上

* `tag`和`disgest`是可选项，如果省略，则会默认使用最新的tag

#### 注释

在Dockerfile里面，在以`#`开头的一行中后面的内容都会被当作注释处理。如果`#`出现在行中则会被当作符号处理。



#### 指令解析器

这是可选的，而且只能出现在Dockerfile文件的开头，必须要遵守一定的规则如`# directive=value`，且单个指令只能使用一次，不能重复。其中现在有两种指令解析器：

##### syntax

```dockerfile
# syntax=[remote image reference]
```

其例子如下所示：

```dockerfile
# syntax=docker/dockerfile
# syntax=docker/dockerfile:1.0
# syntax=docker.io/docker/dockerfile:1
# syntax=docker/dockerfile:1.0.0-experimental
# syntax=example.com/user/repo:tag@sha256:abcdef...
```

这个语法只能在*BuildKit*后端开启的时候才能使用。这个语法指令解析器给出了Dockerfile编译器的路径位置。也就是允许使用自己喜欢的编译器版本对文件进行编译。

##### escape

而这一种指令解析器有两种写法

```dockerfile
# escape=\ (backslash)
```

或者说

```dockerfile
# escape=` (backtick)
```

如果加上这个指令解析器就可以在编写*Dockerfile*的时候不需要转义。



#### RUN

`RUN`指令有两种形式

```dockerfile
RUN <command> (shell form)
```

```dockerfile
RUN ["executable", "param1", "param2"] (exec form)
```

`RUN`指令会在当前镜像顶部一个新的层上执行任何指令并且给出结果，而这个结果镜像会在`Dockerfile`下一步被使用到。一般用来安装软件包。



#### CMD

`CMD`则有三种形式：

```dockerfile
CMD ["executable","param1","param2"] (exec for, preferred)
```

```dockerfile
CMD ["param1","param2"] (默认参数为ENTRYPOINT)
```

```
CMD command param1 param2 (shell form)
```

**一般在`Dockerfile`中只能使用一个`CMD`指令**，如果有多个，则以最后出现的为准。`CMD`命令是当Docker镜像被启动后Docker容器将会默认执行的命令。



#### LABEL

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

指令`LABEL`把元数据加到一个镜像里面。这种`LABEL`是一种键值对，在一个dockerfile可以重复使用。



#### EXPOSE

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

这条指令会告诉`Docker`其容器运行的时候在特地网络上监听的端口。而且还可以指定协议是TCP还是UDP，没有指定的时候默认为TCP。`EXPOSE`指令没有真的建立一个端口，而且作为一种在建立镜像的人和运行容器的人之间的文档，来约定哪一个端口希望被建立。

当在运行容器的时候则需要用到`-p`去对端口做重定向，如

```sh
$ docker run -p 4000:80 friendlyhello
```

这里就相当于把本地4000端口与image的80端口绑在了一起。



#### ENV

```dockerfile
ENV <key> <value>
ENV <key>=<value> ...
```

通过`ENV`可以为镜像定义起环境变量，如

```dockerfile
ENV PATH /usr/local/bin:$PATH
```



#### WORKDIR

```dockerfile
WORKDIR /path/to/workdir
```

这条指令给其他指令设置了工作目录，而且我们可以用环境变量来设置工作目录。

```dockerfile
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```



#### ADD

这条指令有两种形式（这里就省略说明chown部分）

```dockerfile
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>
```

这条指令会从`src`处复制新文件、目录或者经过URL的远程文件然后添加到镜像的文件系统下的`dest`目录中。

##### src

这里的`src`可以包含通配符，例如

```dockerfile
ADD hom* /mydir/        # 添加任何以hom开头的文件
ADD hom?.txt /mydir/    # ?能被任意字符替换
```

##### dest

而对于`dest`，他只能是一个相对的，或者绝对的路径，他指定了`src`里的内容需要复制去的一个位置。

```dockerfile
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```

同时`ADD`指令还有下面的规则：

* 其中`src`的路径必须在build上下文中。即不能使用`ADD ../sth/sth`这样的写法。
* 如果`src`是一个URL，并且`dest`没有以斜杠结尾，则从URL下载一个文件并复制到`dest`。
* 如果`src`是一个URL，并且`dest`以斜杠结尾，则文件名需要从URL推断出来。
* 如果`src`是一个目录，则复制目录的全部内容，包括文件系统元数据。
* ...



#### COPY

```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

这个`COPY`指令从`src`里面复制新文件、目录到容器的文件系统下的`dest`目录下。（只复制内容不复制目录本身

因为`COPY`和`ADD`其实看起来作用相差不大，这里主要讲一下他们的区别。

* 首先如果只是把本地的文件复制到容器中，`COPY`已经就很合适了

* 对于`ADD`，相较于`COPY`。它还能解压文件然后将解压后的文件复制到镜像里面。而且它还能用url来进行复制。
* 总的来说，正常使用下`COPY`已经可以满足很多情况了，这两个指令也可以这样去理解：`COPY`是`ADD`的子集。