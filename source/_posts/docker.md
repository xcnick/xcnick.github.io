title: Docker笔记
date: 2019-06-24 16:30:23
tags: Docker
---

> Docker笔记
<!-- more -->
# 安装Docker

以Ubuntu为例：

## 卸载旧版本

```Bash
$ sudo apt-get remove docker \
               docker-engine \
               docker.io
```

## 使用APT安装

```Bash
# apt update
$ sudo apt-get update

# apt install
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# Add Docker GPG key
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add stable repository
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# Update the apt package index.
$ sudo apt-get update

# Install the latest version of Docker CE and containerd, or go to the next step to install a specific version:
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

# Add user to "docker" group
$ sudo usermod -aG docker user-name
```

或者使用简便脚本安装

```Bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

## 使用镜像

### 运行

```Bash
$ docker run -it --rm \
    ubuntu:18.04 \
    bash
```

`docker run`是运行容器的命令，参数包括：
* `-it`: `-i`是指交互式操作;`-t`是指终端。
* `--rm`: 容器推出后即删除。
* 镜像以及命令。

当利用`docker run`来创建容器时，Docker 在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个 ip 地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

后台运行：使用`docker run -d`指令。注意，容器是否会长久运行，是和`docker run`指定的命令有关，和`-d`参数无关。

### 启动一个已终止的容器

```Bash
$ docker container start
```

### 终止一个运行中的容器

```Bash
$ docker container stop
```

### 进入容器

有两个方法：
* `docker attach`
* `docker exec`

`attach`命令中，如果从这个stdin中exit，容器会终止。

`exec`命令，后面可以跟-it参数，可以进入命令提示符。并且在stdin中exit，不会导致容器停止。

### 删除容器

使用`docker container rm`删除一个处于终止状态的容器。如果要删除运行中的容器，可使用`-f`参数。

使用`docker container prune`清理所有处于终止状态的容器。


### 列出镜像

```Bash
$ docker image ls
```

### 镜像体积

```Bash
$ docker system df
```

### 虚悬镜像

由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为`<none>`的镜像。这类无标签镜像也被称为**虚悬镜像(dangling image)**

```Bash
$ docker image ls -f dangling=true
```

删除虚悬镜像

```Bash
$ docker image prune
```

### 中间层镜像

为了加速镜像构建、重复利用资源，Docker 会利用 中间层镜像

```Bash
$ docker image ls -a
```

### 删除本地镜像

```Bash
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```
可以使用`Image ID`来指定镜像，也可以使用`<仓库名>:<标签>`来删除镜像。

删除行为有两类：
* `Untagged`: 由于一个镜像可以对应多个标签，删除指定标签的镜像后，若还有别的标签指向这个镜像，则`Deleted`行为不会发生。
* `Deleted`: 当该镜像的所有标签都被取消了，则会触发删除行为。删除的时候是从上层向基础层方向依次进行判断删除。
* 若有容器依赖此镜像，则也不能删除这个镜像。

### 慎用 docker commit

---

## 使用Dockerfile定制镜像

### 构建镜像

```Bash
$ docker build -t nginx:v3 .
```

### 镜像构建上下文(Context)

在`docker build`命令最后有一个`.`，表示上下文路径。

Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API，而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。

在这种C/S架构中，为了让服务端获取本地文件，则引入了上下文概念，用户会指定上下文路径，`docker build`命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。

一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一个 .dockerignore，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。

### 使用Git repo进行构建

docker build 还支持从 URL 构建，比如可以直接从 Git repo 中构建：

```Bash
$ docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1
```

这行命令指定了构建所需的 Git repo，并且指定默认的 master 分支，构建目录为 /11.1/，然后 Docker 就会自己去 git clone 这个项目、切换到指定分支、并进入到指定目录后开始构建。

### 用给定的tar压缩包构建

```Bash
$ docker build http://server/context.tar.gz
```

### FROM 指定基础镜像

```Bash
FROM scratch
...
```

`scratch`镜像为空白镜像

### RUN 执行命令

* shell格式：
  `RUN <命令>`，类似在命令行中输入命令。
* exec格式：
  `RUN ["可执行文件", "参数1", "参数2"]`，类似函数调用格式。

> Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。

尽量使用`&&`将多个命令串起来，使用`\`进行换行。

### COPY 复制文件

格式：

* `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
* `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`
和`RUN`指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。

`COPY`指令将从构建上下文目录中`<源路径>`的文件/目录复制到新的一层的镜像内的`<目标路径>`位置，如：

```Bash
COPY package.json /usr/src/app/
```

`<目标路径>`可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用`WORKDIR`指令来指定）。

### ADD 更高级的复制文件

`ADD`指令和`COPY`的格式和性质基本一致。但是在`COPY`基础上增加了一些功能。

`ADD`指令可解压tar文件，比如官方镜像ubuntu中

```Dockerfile
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
...
```

**应尽可能使用`COPY`, 而不是`ADD`，因为`COPY`语义明确**

### CMD容器启动命令

类似`RUN`指令
* shell格式：
  `CMD <命令>`，类似在命令行中输入命令。
* exec格式：
  `CMD ["可执行文件", "参数1", "参数2"]`，类似函数调用格式。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令.

一般推荐使用`exec`格式，一定要使用双引号。

若使用`shell`格式，实际命令会被包装为`sh -c`的参数形式进行执行。

```Dockerfile
CMD echo $HOME
# 相当于
CMD ["sh", "-c", "echo $HOME"]
```

对于容器来说，启动程序就是主进程，以前台形式运行。

### ENTRYPOINT 入口点

类似`RUN`指令，分为`exec`格式和`shell`格式。

当指定了`ENTRYPOINT`后，`CMD`的含义就发生了改变，不再是直接的运行其命令，而是将`CMD`的内容作为参数传给 `ENTRYPOINT`指令。

* 让镜像变成像命令一样使用

在`docker run`指令中，后续的参数将会替换`CMD`的内容，但当`ENTRYPOINT`存在后，`CMD`的内容会作为参数传递给`ENTRYPOINT`, 则后续参数会被增加到之前的`CMD`之后。

* 应用运行前的准备工作

对于预处理工作，可以使用一个脚本，放入`ENTRYPOINT`中执行。这个脚本会将接收到的参数，即`CMD`中的内容，放在脚本最后执行。

### ENV 设置环境变量

两种格式：

* `ENV <key> <value>`
* `ENV <key1>=<value1> <key2>=<value2>...`

在后文中，无论是`RUN`指令，还是其他运行时应用，都可以用`$KEY`这种方式使用已定义的环境变量。

### ARG 构建参数

格式：`ARG <参数名>[=<默认值>]`

效果与`ENV`类似，都是设置环境变量。不同的是，`ARG`设置的构建环境的环境变量，在容器的运行时时不会存在。该参数的默认值可以用`docker build`中的`--build-arg <参数值>=<值>`来覆盖。

### VOLUME 匿名卷

格式：

* `VOLUME ["路径1", "路径2"...]`
* `VOLUME <路径>`

为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

```Dockerfile
VOLUME /data
```

运行时可以覆盖这个挂在设置：
```Bash
docker run -d -v mydata:/data xxxx
```

### EXPOSE 声明端口

格式：`EXPOSE <端口1> [<端口2>...]`

声明运行时容器提供的服务端口，这只是一个声明。

运行时使用`-p <宿主端口>:<容器端口>`是映射宿主和容器端口。但`EXPOSE`仅仅是声明。

### WORKDIR 指定工作目录

格式：`WORKDIR <工作目录路径>`

指定工作目录，以后各层的当前目录就被改为指定的目录，若不存在，则`WORKDIR`会帮助建立此目录。

### USER 指定当前用户

格式：`USER <用户名>[:<用户组>]`

`USER`指令和`WORKDIR`相似，都是改变环境状态并影响以后的层。`WORKDIR`是改变工作目录，`USER`则是改变之后层的执行`RUN`, `CMD`以及`ENTRYPOINT`这类命令的身份。该用户必须是事先建立好的。

### HEALTHCHECK 健康检查

格式：

* `HEALTHCHECK [选项] CMD <命令>`: 设置检查容器健康状况的命令
* `HEALTHCHECK NONE`: 如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

`HEALTHCHECK`指令是告诉 Docker 应该如何进行判断容器的状态是否正常。

当在一个镜像指定了`HEALTHCHECK`指令后，用其启动容器，初始状态会为`starting`，在`HEALTHCHECK`指令检查成功后变为`healthy`，如果连续一定次数失败，则会变为`unhealthy`。`HEALTHCHECK`支持下列选项：

* `--interval=<间隔>`：两次健康检查的间隔，默认为 30 秒；
* `--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
* `--retries=<次数>`：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。

### ONBUILD 为他人做嫁衣裳

格式：`ONBUILD <其他指令>`

`ONBUILD`是一个特殊的指令，它后面跟的是其它指令，比如`RUN`, `COPY`等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

`Dockerfile`中的其它指令都是为了定制当前镜像而准备的，唯有`ONBUILD`是为了帮助别人定制自己而准备的。

---

## 数据管理

### 数据卷

* `数据卷`可以在容器之间共享和重用
* 对`数据卷`的修改会立马生效
* 对`数据卷`的更新，不会影响镜像
* `数据卷`默认会一直存在，即使容器被删除


```Bash
# 创建一个数据卷
$ docker volume create my-vol

# 查看所有数据卷
$ docker volume ls
```

在用`docker run`命令的时候，使用`--mount`标记来将`数据卷`挂载到容器里。在一次`docker run`中可以挂载多个`数据卷`。

```Bash
$ docker run -d -P \
    --name web \
    # -v my-vol:/wepapp \
    --mount source=my-vol,target=/webapp \
    training/webapp \
    python app.py
```

```Bash
# 删除数据卷
$ docker volume rm my-vol

# 清理数据卷
$ docker volume prune
```

### 挂载主机目录

使用`--mount`标记可以指定挂在一个本地的目录到容器中。使用`-v`参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用`--mount`参数时如果本地目录不存在，Docker会报错。Docker 挂载主机目录的默认权限是读写，用户也可以通过增加`readonly`指定为只读。

`--mount`也可以挂载单个文件到容器中。

---

## 网络管理

### 外部访问容器

使用`-P`标记时Docker会随机映射一个`49000~49900`端口到容器内部的开放端口。

使用`-p`可以指定要映射的端口。一个端口只能绑定到一个容器。

```Bash
# 映射所有接口的5000端口 hostPort:containerPort
$ docker run -d -p 5000:5000 training/webapp python app.py

# 映射到指定地址的指定端口 ip:hostPort:containerPort
$ docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py

# 映射到指定地址的任意端口 ip::containerPort
$ docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

可以多次使用`-p`标记来绑定多个端口。

### 配置DNS

`docker run`命令时使用
* `-h HOSTNAME`或者`--hostname=HOSTNAME`设定容器主机名，写入容器内的`/etc/hostname`中
* `--dns=IP_ADDRESS`添加DNS服务器到容器的`/etc/resolv.conf`中