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
### 3.2 安装网络插件
确保梯子正常启动

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
### 3.3 使用 ansible 部署 work 节点
参考 ansible-install-k8s.yaml

### 3.4 容器网络测试
创建 test 命名空间
```bash
kubectl create ns test
```

参考 debug-pod.yaml，在 ns = test 中，部署两个 debug-pod 并创建对应的 svc，保证 pod 调度到不同节点上。
```bash
kubectl apply -f debug-pod.yaml
```

使用 exec 进入到 debug-pod-1 pod 中，使用 curl 访问 svc，查看是否正常。
```bash 
kubectl exec -it debug-pod-1 -n test -- sh

curl -v http://debug-pod-2.test.svc.cluster.local
```
