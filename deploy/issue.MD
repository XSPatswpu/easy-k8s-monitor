# issue

## 集群部署完成后 work1 节点上 pod 无法访问日志

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