# 编写 Dockerfile 文件
```Dcokerfile
# 基础镜像
FROM nginx:1.25-alpine

# 安装工具包
RUN apk update && apk add --no-cache \
netcat-openbsd \
net-tools \
curl \
iputils \
wget \
tcpdump \
busybox-extras \
bind-tools \
vim \
less \
ca-certificates

# 保持默认的 nginx 配置和启动命令
CMD ["nginx", "-g", "daemon off;"]
```
# build 容器镜像
```bash
docker build -t k8s-base/debug-pod:v1.0.0 .
```
# docker run 测试
```bash
docker run -it --rm k8s-base/debug-pod:v1.0.0 sh
```
# 推送仓库
```bash
sudo docker login --username=<docker-hub-username> 

docker tag k8s-base/debug-pod:v1.0.0 cherry22/debug-pod:v1.0.0

docker push --platform linux/amd64 cherry22/debug-pod:v1.0.0
```