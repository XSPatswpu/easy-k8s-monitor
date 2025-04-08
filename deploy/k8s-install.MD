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
```yaml
---
- hosts: worker
  become: yes
  tasks:
  - name: Update apt repo
    apt:
      update_cache: yes
  - name: Install dependencies
    apt:
      name:
        - apt-transport-https
        - ca-certificates
        - curl
        - gnupg
        - lsb-release
      state: present
  # 安装 containerd
  - name: Add Docker official GPG key
    apt_key:
      url: https://download.docker.com/linux/ubuntu/gpg
      state: present
  - name: Add Docker repository
    apt_repository:
      repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
      state: present
      filename: docker-ce.list
  - name: Install containerd with dependencies
    apt:
      name:
        - containerd.io
        - containernetworking-plugins
      update_cache: yes
  - name: Generate default containerd config
    shell: |
      mkdir -p /etc/containerd
      containerd config default > /etc/containerd/config.toml
    args:
      creates: /etc/containerd/config.toml
  - name: Enable systemd cgroup driver
    replace:
      path: /etc/containerd/config.toml
      regexp: 'SystemdCgroup = false'
      replacement: 'SystemdCgroup = true'
    notify: restart containerd
  - name: Ensure containerd user systemd directory exists
    ansible.builtin.file:
      path: /etc/systemd/system/containerd.service.d
      state: directory
      mode: '0755'
      owner: root
      group: root
  - name: Deploy existing proxy config
    ansible.builtin.copy:
      src: /etc/systemd/system/containerd.service.d/http-proxy.conf
      dest: /etc/systemd/system/containerd.service.d/http-proxy.conf
      owner: root
      group: root
      mode: '0644'
    notify:
      - reload systemd
      - restart containerd

  # Kubernetes 安装部分
  - name: Ensure APT keyring directory
    file:
      path: /etc/apt/keyrings
      state: directory
      mode: 0755
  - name: Add Kubernetes GPG key
    apt_key:
      url: https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key
      keyring: /etc/apt/keyrings/kubernetes-apt-keyring.gpg
      state: present
  - name: Add Kubernetes repository
    apt_repository:
      repo: "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /"
      state: present
      filename: kubernetes.list
  - name: Update package index
    apt:
      update_cache: yes
  - name: Install Kubernetes packages
    apt:
      name:
        - kubelet
        - kubeadm
        - kubectl
      state: present
  - name: Hold Kubernetes packages
    dpkg_selections:
      name: "{{ item }}"
      selection: hold
    loop:
      - kubelet
      - kubeadm
      - kubectl

  handlers:
    - name: restart containerd
      systemd:
        name: containerd
        state: restarted
        daemon_reload: yes
    - name: reload systemd
      ansible.builtin.systemd:
        daemon_reload: yes
    - name: restart containerd
      ansible.builtin.systemd:
        name: containerd
        state: restarted

- hosts: control
  become: yes
  tasks:
  - name: 创建 kubeadm token
    command: kubeadm token create
    register: token_output
  - name: 保存 join 命令所需的信息
    set_fact:
      kubeadm_token: "{{ token_output.stdout }}"
- hosts: worker
  become: yes
  tasks:
  - name: 使用获取的 token 加入 Kubernetes 集群
    shell: >
      kubeadm join 192.168.0.103:6443
      --token {{ hostvars['localhost'].kubeadm_token }}
      --discovery-token-ca-cert-hash sha256:11f5e97cec64609ed9e6027f4a7b60e17e992cab83fd8e184761f536bce65fce
```
### 3.4 容器网络测试