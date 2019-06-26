title: Clash透明网关
date: 2019-06-24 16:30:23
tags: Docker
---

> Clash透明网关搭建流程
<!-- more -->
# 安装系统

本文已`Alpine`为例，下载地址为：https://alpinelinux.org/downloads/。

若使用虚拟机安装，下载`Virtual`版本即可。

*安装过程略*

# 配置依赖项

```Bash
# 更新软件库
$ apk update

# 安装https支持、iptables
$ apk add ca-certificates iptables
```

# 配置iptables

iptables设置脚本如下，可保存至.sh脚本中执行。

```Bash
iptables -t nat -A PREROUTING -p tcp -m multiport --dport 22,15401 -j ACCEPT
iptables -t nat -A PREROUTING -d 0.0.0.0/8 -j RETURN
iptables -t nat -A PREROUTING -d 10.0.0.0/8 -j RETURN
iptables -t nat -A PREROUTING -d 127.0.0.0/8 -j RETURN
iptables -t nat -A PREROUTING -d 169.254.0.0/16 -j RETURN
iptables -t nat -A PREROUTING -d 172.16.0.0/12 -j RETURN
iptables -t nat -A PREROUTING -d 192.168.0.0/16 -j RETURN
iptables -t nat -A PREROUTING -d 224.0.0.0/4 -j RETURN
iptables -t nat -A PREROUTING -d 240.0.0.0/4 -j RETURN
iptables -t nat -A PREROUTING -p tcp -j REDIRECT --to-ports 7892
```

```Bash
# 保存iptables设置
$ service iptables save

# 启动时加载iptables
$ rc-update add iptables
```

# 安装clash

将`clash`及其配置文件拷贝到系统中。

## 设置开机自动启动`clash`

```Bash
# 创建clash.start文件
$ vi /etc/local.d/clash.start
```

向`clash.start`文件中写入启动脚本：
```Bash
/root/clash-linux-amd64 -d /root > /dev/null 2>&1 &
```

设置可执行权限
```Bash
$ chmod +x /etc/local.d/clash.start
```

设置`local`服务开机启动
```Bash
$ rc-update add local
```