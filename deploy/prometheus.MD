# prometheus

## 概览
主要流程：
1. 手动部署 prometheus
2. 控制面组件监控配置
3. 节点监控
4. 手动部署 grafana
5. grafana 初始化监控面板镜像

## install
### 准备 yaml 资源
参考 deploy/example
### 安装 yaml 资源
安装如下顺序执行：
```bash
kubectl apply -f prometheus-cm.yaml
kubectl apply -f prometheus-volume.yaml
kubectl apply -f prometheus-rbac.yaml
kubectl apply -f prometheus-deploy.yaml
```
### 外网访问 prometheus 方案
![img.png](../image/prometheus-access.png)
