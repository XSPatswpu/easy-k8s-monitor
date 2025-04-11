# issue

## 集群部署完成后 work1 节点上 pod 无法访问日志
现象：
1. etcd 中能看到 work1 节点上的 pod，但是 logs 返回 not found
2. kubectl 删除 pod，在 work1 节点上观察实时 kubelet 日志，发现 kubelet 准备下发删除 container 请求时，找不到对应 container。

解决：
暂未排查根因，直接销毁 work1，重新创建 work4 节点，问题消失。
## flannel 安装完成，容器网络不通
问题列表：
1. flannel 插件网卡选择错误
```txt
I0403 10:20:41.115061       1 match.go:264] Using interface with name enp0s3 and address 10.0.2.15
```
修改 flannel ds containers args 解决：
```yaml
      containers:
      - args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface-regex=vboxnet0|enp0s8
```
2. 容器中 curl 访问 svc 域名解析错误。
重启 kube-proxy、core-dns 解决。猜测可能与问题 3 相关。
3. kube-proxy pod 异常日志
```txt
W0408 01:41:53.631340       1 reflector.go:539] vendor/k8s.io/client-go/informers/factory.go:159: failed to list *v1.Service: Get "https://192.168.31.134:6443/api/v1/services?labelSelector=%21service.kubernetes.io%2Fheadless%2C%21service.kubernetes.io%2Fservice-proxy-name&resourceVersion=4088645": dial tcp 192.168.31.134:6443: connect: network is unreachable
E0408 01:41:53.631397       1 reflector.go:147] vendor/k8s.io/client-go/informers/factory.go:159: Failed to watch *v1.Service: failed to list *v1.Service: Get "https://192.168.31.134:6443/api/v1/services?labelSelector=%21service.kubernetes.io%2Fheadless%2C%21service.kubernetes.io%2Fservice-proxy-name&resourceVersion=4088645": dial tcp 192.168.31.134:6443: connect: network is unreachable
W0408 01:42:19.489163       1 reflector.go:539] vendor/k8s.io/client-go/informers/factory.go:159: failed to list *v1.EndpointSlice: Get "https://192.168.31.134:6443/apis/discovery.k8s.io/v1/endpointslices?labelSelector=%21service.kubernetes.io%2Fheadless%2C%21service.kubernetes.io%2Fservice-proxy-name&resourceVersion=4088716": dial tcp 192.168.31.134:6443: connect: network is unreachable
E0408 01:42:19.489215       1 reflector.go:147] vendor/k8s.io/client-go/informers/factory.go:159: Failed to watch *v1.EndpointSlice: failed to list *v1.EndpointSlice: Get 
```
重启 kube-proxy、core-dns 解决。猜测可能与问题 1 相关。

## 如何开放物理机防火墙规则
因为后续准备以阿里云服务器为跳板机访问服务器，需要配置服务器防火墙规则，开放 8080 端口。

检查防火墙状态：
```bash
sudo ufw status
```

开放 8080 端口：
```bash
sudo ufw allow 8080/tcp
```

重新加载防火墙规则：
```bash
sudo ufw reload
```

验证防火墙规则：
```bash
sudo ufw status | grep 8080
```