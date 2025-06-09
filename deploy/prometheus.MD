# prometheus

## 1 概览
通过手动搭建 prometheus、grafana 验证流程通路，为后续做自动化监控系统做准备。

主要流程：
1. 手动部署 prometheus
2. 控制面组件监控配置 & 节点监控
    * coreDns
    * apiserver
    * etcd
    * kube-controller
    * kube-scheduler
    * 节点监控
3. 手动部署 grafana
4. grafana 初始化监控面板镜像

## 2 安装 prometheus
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

## 配置控制面组件监控


## 节点监控
### 安装 node-exporter
安装流程参考：https://artifacthub.io/packages/helm/prometheus-community/prometheus-node-exporter

node-exporter scrapeJob config:
```yaml
scrape_configs:
  - job_name: 'node-exporter'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - action: keep
        source_labels:
          - __meta_kubernetes_pod_label_app
        regex: node-exporter  # Match pods with label app=node-exporter
      - action: keep
        source_labels:
          - __meta_kubernetes_pod_annotation_prometheus_io_scrape
        regex: "true"
      - source_labels:
          - __meta_kubernetes_pod_annotation_prometheus_io_path
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels:
          - __address__
          - __meta_kubernetes_pod_annotation_prometheus_io_port
        action: replace
        regex: (.+):(?:\d+);(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)

```