---
title: Docker学习日记-Day1
date: 2019-04-02 21:43:54
tags:
---

>  由于实训要求搭建一个树莓派集群，故需要学习docker及docker swarm。便开始了docker的学习，也随之记下相关的笔记。

### 概念

**Docker**是一个提供给开发者、系统管理员利用容器(*containers*)进行开发，部署以及运行程序的平台。其中容器化(*containerization*)是指在**Linux**上使用*container*去部署应用。

##### 容器化的特性

* 灵活(*Flexible*): 即使最复杂的应用都能进行容器化。
* 轻量(*Lightweight*): 容器利用、共享主机内核
* 可互换的(Interchangeable): 用户可以动态进行部署更新、升级
* 便携(Portable): 用户可以在本地搭建，部署到云端，甚至在任何地方运行
* 可伸展(Scalable): 用户可以添加并自动分发*container*副本
* 可堆叠(Stackable): 用户能垂直地动态地堆叠服务



#### Images与containers

首先*Image*是一个可执行的包，它包括了需要运行一个应用程序的任何东西，其中有代码、运行时间、库、环境、变量以及配置文件。

一个*container*是*image*的一个运行时间实例，也就是说当实际执行时，在内存中*image*会变成*container*。

如果要查看现在有多少容器在后台运行，可以在Linux终端输入

```shell
$ docker ps
```



#### 容器与虚拟机对比

容器在Linux上运行并且和其他容器共享同一个主机内核。容器的过程是离散性的，它只占有其程序运行所需的最小的内存，不会过多占用机器内存。这也体现了其轻量的特点。

而虚拟机会运行整一个操作系统，包括各种程序不会使用的接口，它提供的环境和资源比绝大多数应用程序所需的要多，相比之下就显得很冗杂。



### 安装及测试

#### 在Centos安装

因为centos自带yum，这里可以直接下载

在下载的过程遇到了一些小麻烦，因为一开始下载的版本不对后面还重装了一遍，重装过程可以点[这里](https://blog.csdn.net/qq_21816375/article/details/79832593)。

总而言之，可以通过下面的命令直接安装。

```shell
$ sudo yum install docker-ce
```

接着启动docker

```shell
$ service docker start
```

也可以查看docker版本

```shell
$ docker version
```

##### 小报错以及解决方案

>[root@localhost ~]# docker run hello-world
>
>docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
>See 'docker run --help'.

出现这个情况一般是因为docker服务没有启动，所以一般执行一下`service docker start`就可以恢复正常。



#### Docker使用

1. 首先我们可以尝试去从`Docker Hub`的库里面把*helloworld*的`image`给拉到本地来，运行

```shell
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

2. 这时候我们就可以查看到我们刚下载的*image*镜像

   ```shell
   $ docker image ls
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   hello-world         latest              fce289e99eb9        2 months ago        1.84kB
   ```

3. 而如果查看docker里面的容器，可以运行(如果不加上all只能看到正在运行的容器)

   ```shell
   $ docker container ls --all
   CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
   e8c17790a5f7        hello-world         "/hello"            31 minutes ago      Exited (0) 31 minutes ago                       keen_lalande
   ```



### Dockfile

因为当我们运行程序的时候，我们一般都会首先为软件搭建一个开发环境。但在*Docker*上，*Dockefile*是一种可携带性的运行环境、镜像，他没有安装的必要。

Dockfile定义了容器里面的环境，它会虚拟化容器的网络接口和硬件驱动，这样我们可以app需要的环境、文件都写在里面，之后整个app就能像我们想象的一样运行起来。

#### 需求文件

这里我首先就简单的分析官网上的例子，如果需要查看dockerfile各种规范语法，具体看[这里](<https://chaoscodes.github.io/2019/03/26/Dockerfile%E6%8C%87%E4%BB%A4%E6%80%BB%E7%BB%93/>).

`Dockerfile`

```dockerfile
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

这里还需要有另外两个文件`requirements.txt`和`app.py`与dockerfile放在同一目录下。

`requirements.txt`

```tx
Flask
Redis
```

`app.py`

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

#### 新建镜像

首先新建三个文件如上所示，然后运行

```shell
docker build --tag=friendlyhello .
```

其中`.`指定了build的路径，而`tag`指定了镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签，这里还可以这样写`-t friendlyhello`。

于是运行之后docker就会根据dockerfile的指令一步一步新建一个`image`，在下面我们可以看到每一步的步骤。

```shell
Sending build context to Docker daemon  9.216kB
Step 1/7 : FROM python:2.7-slim
2.7-slim: Pulling from library/python
27833a3ba0a5: Pull complete
8b35abcb27de: Pull complete
cd1fc6dee9fe: Pull complete
2c6a92003566: Pull complete
Digest: sha256:55c18f359c28061ddf70ee194439e9a5b2bdc205df3e7559b419dad10086fc89
Status: Downloaded newer image for python:2.7-slim
 ---> 48e3247f2a19
Step 2/7 : WORKDIR /app
 ---> Running in b2fc58222a81
Removing intermediate container b2fc58222a81
 ---> b4dacec4bec5
Step 3/7 : COPY . /app
 ---> a37d72004a37
Step 4/7 : RUN pip install --trusted-host pypi.python.org -r requirements.txt
 ---> Running in fc0a07eeb8d1
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that date. A future version of pip will drop support for Python 2.7.
Collecting Flask (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7f/e7/08578774ed4536d3242b14dacb4696386634607af824ea997202cd0edb4b/Flask-1.0.2-py2.py3-none-any.whl (91kB)
Collecting Redis (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/ac/a7/cff10cc5f1180834a3ed564d148fb4329c989cbb1f2e196fc9a10fa07072/redis-3.2.1-py2.py3-none-any.whl (65kB)
Collecting itsdangerous>=0.24 (from Flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting Jinja2>=2.10 (from Flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl (126kB)
Collecting Werkzeug>=0.14 (from Flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/24/4d/2fc4e872fbaaf44cc3fd5a9cd42fda7e57c031f08e28c9f35689e8b43198/Werkzeug-0.15.1-py2.py3-none-any.whl (328kB)
Collecting click>=5.1 (from Flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl (81kB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.10->Flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/fb/40/f3adb7cf24a8012813c5edb20329eb22d5d8e2a0ecf73d21d6b85865da11/MarkupSafe-1.1.1-cp27-cp27mu-manylinux1_x86_64.whl
Installing collected packages: itsdangerous, MarkupSafe, Jinja2, Werkzeug, click, Flask, Redis
Successfully installed Flask-1.0.2 Jinja2-2.10 MarkupSafe-1.1.1 Redis-3.2.1 Werkzeug-0.15.1 click-7.0 itsdangerous-1.1.0
Removing intermediate container fc0a07eeb8d1
 ---> 3dad0b0e4b68
Step 5/7 : EXPOSE 80
 ---> Running in 473579a3f06d
Removing intermediate container 473579a3f06d
 ---> 6eee048df31d
Step 6/7 : ENV NAME World
 ---> Running in 970f404be828
Removing intermediate container 970f404be828
 ---> bc91bb38ab83
Step 7/7 : CMD ["python", "app.py"]
 ---> Running in b3eb8764a188
Removing intermediate container b3eb8764a188
 ---> 53f7b2ae41d3
Successfully built 53f7b2ae41d3
Successfully tagged friendlyhello:latest
```

这个时候我们就可以查看到刚才新建的image

```shell
$  docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED                  SIZE
python              2.7-slim            48e3247f2a19        Less than a second ago   120MB
friendlyhello       latest              53f7b2ae41d3        14 seconds ago           131MB
```

#### 运行程序

但是这个时候只是新建了一个image，它还没开始运行。所以如果要把这个image跑起来，可以执行下面的指令，并且把容器的80端口与本地的4000端口绑定起来做一个映射的关系。

```shell
docker run -p 4000:80 friendlyhello
```

于是运行可以看到

![](localhost.png)

如果要让container在后台运行，就需要在运行的时候加上`detached`状态

```shell
docker run -d -p 4000:80 friendlyhello
```

于是这是就可以通过`docker container ls`来查看刚刚运行的`container`

```shell
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
8afe24b60ce3        friendlyhello       "python app.py"     2 minutes ago       Up 2 minutes        0.0.0.0:4000->80/tcp   lucid_snyder
```

然后用`docker container stop`来中止后台的容器，例如结束刚刚运行的容器

```shell
$ docker container stop 8afe24b60ce3
8afe24b60ce3
```



