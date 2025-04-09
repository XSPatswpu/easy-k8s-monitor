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