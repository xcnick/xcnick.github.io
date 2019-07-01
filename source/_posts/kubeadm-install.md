title: 使用kubeadm部署k8s集群
date: 2019-07-01 09:40:12
tags: kubernetes
---

> 使用kubeadm部署k8s集群
<!-- more -->
# 安装Docker和kubeadm

切换到root用户，执行如下脚本。
```Bash
apt-get update && apt-get install -y ca-certificates gnupg-agent apt-transport-https software-properties-common curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

apt-get update
apt-get install -y kubelet kubeadm kubectl docker-ce

usermod -aG docker your-user
```

* 脚本运行完成后，docker和kubeadm部署完成。
* `Master`和`Node`节点都需要安装这两项。

## 安装Master节点

### 初始化Master节点

```Bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16  --apiserver-advertise-address=192.168.1.63
```

### 使普通用户具有kubectl权限：

```Bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 下载安装dashboard

```Bash
wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

### 修改dashboard的文件

1. 增加NodePort
```Yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  # 添加Service的type为NodePort
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      # 添加映射到虚拟机的端口,k8s只支持30000以上的端口
      nodePort: 30001
  selector:
    k8s-app: kubernetes-dashboard
```

2. 增加权限
```Yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```

### 安装flannel

```Bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### 安装dashboard

```Bash
kubectl create -f kubernetes-dashboard.yaml
```

## 获取token，登录Master节点
```Bash
kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token
```
若按上述命令，`Master`节点端口号为`30001`。

## 初始化Node节点

1. `Node`节点上部署`docker`和`kubeadm`；
2. 使用`kubeadm init`时显示的`kubeadm join`脚本进行初始化脚本，初始化`Node`节点。