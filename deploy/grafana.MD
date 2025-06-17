# Grafana

## 部署
### 安装参考
https://artifacthub.io/packages/helm/grafana/grafana

### 安装完成提示
```txt
NAME: xsp-release
LAST DEPLOYED: Mon Jun  9 17:40:01 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get your 'admin' user password by running:

   kubectl get secret --namespace default xsp-release-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


2. The Grafana server can be accessed via port 80 on the following DNS name from within your cluster:

   xsp-release-grafana.default.svc.cluster.local

   Get the Grafana URL to visit by running these commands in the same shell:
     export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=xsp-release" -o jsonpath="{.items[0].metadata.name}")
     kubectl --namespace default port-forward $POD_NAME 3000

3. Login with the password from step 1 and the username: admin
#################################################################################
######   WARNING: Persistence is disabled!!! You will lose your data when   #####
######            the Grafana pod is terminated.                            #####
#################################################################################
```

### 创建 svc
```yaml
apiVersion: v1
kind: Service
metadata:
  name: public-grafana-svc
  namespace: default
  labels:
    app.kubernetes.io/name: grafana
spec:
  selector:
    app.kubernetes.io/name: grafana
  type: NodePort
  ports:
    - name: web
      port: 80
      targetPort: grafana
```
## 外网访问
使用和 prometheus 同样的外网访问方式。

nginx 反向代理配置（/etc/nginx/sites-available/grafana-site）：
```txt
map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}

upstream grafana {
  server <k8s集群apiserver外网IP>:<grafana svc nodePort>;
}

server {
  listen 8888;
  server_name <nginx ecs IP>;

  location / {
    proxy_set_header Host $host;
    proxy_pass http://grafana;
  }

  location /api/live/ {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host $host;
    proxy_pass http://grafana;
  }
}
```

参考文档：
https://grafana.com/tutorials/run-grafana-behind-a-proxy/

## 内置面板数据
