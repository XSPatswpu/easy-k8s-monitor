# easy-k8s-monitor
**项目介绍：**

围绕 k8s 建立容器监控平台的托管系统。

**项目第一阶段目标：**

使用一个 cr 文件，完成自动化部署监控系统，包括 grafana 服务、exporter 、vmagent 采集器、指标存储后端集群（prometheus or vm）。

**项目愿景：**

实现可观测三大系统的自动化部署能力：指标监控、日志、链路追踪。

## 项目进度
- [x] 完成基于单机 + 虚拟机 部署 k8s 集群。
    - [x] 使用 ansible + vagrant 一行命令拉起 k8s 集群
        - [x] 容器运行时 containerD
        - [x] 网络插件使用 flannel
    - [x] 部署网络插件，部署完成，待验证网络可用性
    - [x] 存储方案，暂时放弃 rook-ceph 框架，采用 HostPath + local PV

- [ ] 手动搭建监控系统，验证整理流程通路。
    - [x] 安装 prometheus
    - [ ] 配置控制面组件监控、节点监控
        - [x] node-exporter
        - [ ] kube-state-metrics
    - [x] 安装 grafana

- [ ] 基于 operator 自动化部署方案设计。

- [ ] 开发 operator。
