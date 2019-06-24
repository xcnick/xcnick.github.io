title: Docker笔记
date: 2019-06-24 16:30:23
tags: Docker
---

> Docker笔记

# 什么是Docker


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