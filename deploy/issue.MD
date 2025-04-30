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

## prometheus 启动失败
启动失败日志：
```txt
root@xsp-Venus-series:/home/xsp/dev/prometheus# k -n monitor logs prometheus-58b586598b-rjxmk
ts=2025-04-30T08:30:18.255Z caller=main.go:516 level=info msg="Starting Prometheus" version="(version=2.34.0, branch=HEAD, revision=881111fec4332c33094a6fb2680c71fffc427275)"
ts=2025-04-30T08:30:18.255Z caller=main.go:521 level=info build_context="(go=go1.17.8, user=root@121ad7ea5487, date=20220315-15:18:00)"
ts=2025-04-30T08:30:18.255Z caller=main.go:522 level=info host_details="(Linux 5.4.0-187-generic #207-Ubuntu SMP Mon Jun 10 08:16:10 UTC 2024 x86_64 prometheus-58b586598b-rjxmk (none))"
ts=2025-04-30T08:30:18.255Z caller=main.go:523 level=info fd_limits="(soft=1048576, hard=1048576)"
ts=2025-04-30T08:30:18.255Z caller=main.go:524 level=info vm_limits="(soft=unlimited, hard=unlimited)"
ts=2025-04-30T08:30:18.255Z caller=query_logger.go:90 level=error component=activeQueryTracker msg="Error opening query log file" file=/prometheus/queries.active err="open /prometheus/queries.active: permission denied"
panic: Unable to create mmap-ed active query log

goroutine 1 [running]:
github.com/prometheus/prometheus/promql.NewActiveQueryTracker({0x7ffebd3fbe00, 0xb}, 0x14, {0x3637a40, 0xc0007bea00})
	/app/promql/query_logger.go:120 +0x3d7
main.main()
	/app/cmd/prometheus/main.go:569 +0x6049
```
查看 prometheus 对应的 Dockerfile 文件,https://github.com/prometheus/prometheus/blob/main/Dockerfile
对于用户为 nobody

使用 local pvc 挂载节点磁盘路径 /data/k8s/prometheus 默认用户是 root，可能是 nobody 权限不足
