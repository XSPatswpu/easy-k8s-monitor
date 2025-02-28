# k8s集群安装

## 1 前提
1. 使用 kubeadm 安装控制面组件
2. 容器运行时使用 containerD
3. 网络插件使用 flannel
4. 存储插件太复杂了，暂时直接使用 Local PV

## 2 准备工作
1. 使用 vagrant 创建虚机
2. 准备 kubeadm 配置文件
3. 准备 ansible 配置文件

## 3 安装
### 3.1 准备 master 节点环境
#### 软件包更新
apt update 更新软件包，其中涉及到 GPG 密钥与软件包签名，相关知识先跳过，直接按照 k8s 官方 doc 更新密钥以及软件包签名。
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
主要参考文档 Installing kubeadm, kubelet and kubectl 相关内容

#### 安装 containerd
参考：https://github.com/containerd/containerd/blob/main/docs/getting-started.md
注意 containerd.service 路径 

#### 安装 kubeadm

#### 执行 kubeadm
kubeadm init --config kubeadm-init.yaml

kubeadm-init.yaml 内容如下
```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
nodeRegistration:
  criSocket: "unix:///run/containerd/containerd.sock"
  kubeletExtraArgs:
    cgroup-driver: "systemd"
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: "v1.29.3"
networking:
  podSubnet: "10.244.0.0/16"
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```
#### 安装网络插件

执行 ansible
1. 安装控制面组件
2. 准备 work 节点环境
3. work 节点加入 k8s 集群