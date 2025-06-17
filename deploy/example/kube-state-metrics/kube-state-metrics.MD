# kube-state-metrics

## 安装
helm install [RELEASE_NAME] prometheus-community/kube-state-metrics

参考：https://artifacthub.io/packages/helm/prometheus-community/kube-state-metrics

## 配置采集任务
```yaml
scrape_configs:
  - job_name: 'kube-state-metrics'
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
            - default

    relabel_configs:
      - action: keep
        source_labels:
          - __meta_kubernetes_pod_label_app
        regex: kube-state-metrics

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