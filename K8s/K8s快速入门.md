# Kubernetes 快速入门指南

## 🎯 什么是 Kubernetes

Kubernetes（K8s）是一个开源的容器编排平台，用于自动化部署、扩展和管理容器化应用程序。

> **名字由来**：Kubernetes 源自希腊语，意为"舵手"或"领航员"。K8s 是因为 K 和 s 之间有 8 个字母。

---

## 🏗️ 核心架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        Kubernetes Cluster                        │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                     Control Plane (Master)                   │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐ │ │
│  │  │ API Server   │ │   etcd       │ │ Controller Manager   │ │ │
│  │  └──────────────┘ └──────────────┘ └──────────────────────┘ │ │
│  │  ┌──────────────┐ ┌──────────────────────────────────────┐  │ │
│  │  │  Scheduler   │ │    Cloud Controller Manager          │  │ │
│  │  └──────────────┘ └──────────────────────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                      Worker Nodes                            │ │
│  │  ┌─────────────────┐  ┌─────────────────┐                   │ │
│  │  │     Node 1      │  │     Node 2      │  ...              │ │
│  │  │ ┌─────┐ ┌─────┐ │  │ ┌─────┐ ┌─────┐ │                   │ │
│  │  │ │ Pod │ │ Pod │ │  │ │ Pod │ │ Pod │ │                   │ │
│  │  │ └─────┘ └─────┘ │  │ └─────┘ └─────┘ │                   │ │
│  │  │    kubelet      │  │    kubelet      │                   │ │
│  │  │    kube-proxy   │  │    kube-proxy   │                   │ │
│  │  └─────────────────┘  └─────────────────┘                   │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Control Plane 组件

| 组件 | 功能 |
|-----|------|
| **API Server** | 集群的入口，处理所有 REST 请求 |
| **etcd** | 分布式键值存储，保存集群状态 |
| **Scheduler** | 决定 Pod 运行在哪个 Node |
| **Controller Manager** | 运行控制器进程（副本、节点、端点等） |

### Node 组件

| 组件 | 功能 |
|-----|------|
| **kubelet** | 确保容器在 Pod 中运行 |
| **kube-proxy** | 维护网络规则，实现 Service |
| **Container Runtime** | 运行容器（Docker、containerd） |

---

## 📦 核心概念

### Pod - 最小部署单元

```yaml
# pod-example.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:
  - name: my-container
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```bash
# 创建 Pod
kubectl apply -f pod-example.yaml

# 查看 Pod
kubectl get pods

# 查看详情
kubectl describe pod my-pod

# 进入容器
kubectl exec -it my-pod -- /bin/bash

# 删除 Pod
kubectl delete pod my-pod
```

### Deployment - 管理无状态应用

```yaml
# deployment-example.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3  # 副本数
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```bash
# 创建 Deployment
kubectl apply -f deployment-example.yaml

# 查看状态
kubectl get deployments
kubectl rollout status deployment/nginx-deployment

# 扩缩容
kubectl scale deployment nginx-deployment --replicas=5

# 更新镜像（滚动更新）
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# 回滚
kubectl rollout undo deployment/nginx-deployment
```

### Service - 服务发现与负载均衡

```yaml
# service-example.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80        # Service 端口
    targetPort: 80  # Pod 端口
  type: ClusterIP   # 类型：ClusterIP / NodePort / LoadBalancer
```

#### Service 类型对比

| 类型 | 访问范围 | 使用场景 |
|-----|---------|---------|
| **ClusterIP** | 集群内部 | 内部服务通信 |
| **NodePort** | 节点 IP + 端口 | 开发测试环境 |
| **LoadBalancer** | 外部负载均衡器 | 云环境生产 |
| **ExternalName** | DNS 别名 | 外部服务引用 |

### ConfigMap 与 Secret

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "mysql.default.svc.cluster.local"
  DATABASE_PORT: "3306"

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=  # base64 编码
```

在 Pod 中使用：
```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secret
```

---

## ⚡ 常用命令速查

### 基础命令

```bash
# 集群信息
kubectl cluster-info
kubectl get nodes

# 命名空间
kubectl get namespaces
kubectl create namespace dev
kubectl config set-context --current --namespace=dev

# 获取资源
kubectl get pods -A              # 所有命名空间
kubectl get pods -o wide         # 更多信息
kubectl get pods -o yaml         # YAML 格式
kubectl get all                  # 所有资源

# 描述资源
kubectl describe pod <pod-name>
kubectl describe node <node-name>

# 日志
kubectl logs <pod-name>
kubectl logs -f <pod-name>       # 实时追踪
kubectl logs <pod-name> -c <container-name>  # 多容器

# 执行命令
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -- ls /app

# 端口转发
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward svc/<service-name> 8080:80

# 资源操作
kubectl apply -f <file.yaml>     # 创建/更新
kubectl delete -f <file.yaml>    # 删除
kubectl edit deployment <name>   # 编辑

# 调试
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods                 # 资源使用
kubectl top nodes
```

### 快捷别名（推荐配置）

```bash
# ~/.bashrc 或 ~/.zshrc
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployment'
alias kgn='kubectl get nodes'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias klf='kubectl logs -f'

# 自动补全
source <(kubectl completion zsh)
```

---

## 🔧 实战示例

### 部署完整应用

```yaml
# complete-app.yaml
---
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"

---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: myapp:1.0
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: my-app-config
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

---

## 🚀 进阶主题目录

- [[K8s-存储管理]] - PV、PVC、StorageClass
- [[K8s-网络原理]] - CNI、Service、Ingress
- [[K8s-调度策略]] - 亲和性、污点容忍
- [[K8s-资源管理]] - Limits、Requests、QoS
- [[K8s-安全机制]] - RBAC、NetworkPolicy
- [[K8s-监控告警]] - Prometheus、Grafana

---

## 📚 学习路线图

```
┌────────────────────────────────────────────────────────────┐
│                    K8s 学习路线                             │
│                                                             │
│  Level 1: 基础概念                                          │
│  ├── Docker 基础                                           │
│  ├── Pod、Deployment、Service                              │
│  └── kubectl 常用命令                                       │
│                                                             │
│  Level 2: 核心功能                                          │
│  ├── ConfigMap、Secret                                     │
│  ├── 存储（PV/PVC）                                        │
│  ├── Ingress                                               │
│  └── 健康检查（Probe）                                     │
│                                                             │
│  Level 3: 运维管理                                          │
│  ├── 资源限制与 QoS                                        │
│  ├── 调度策略                                              │
│  ├── RBAC 权限管理                                         │
│  └── Helm 包管理                                           │
│                                                             │
│  Level 4: 生产实践                                          │
│  ├── 监控与日志                                            │
│  ├── CI/CD 集成                                            │
│  ├── 高可用部署                                            │
│  └── 故障排查                                              │
└────────────────────────────────────────────────────────────┘
```

---

## 🔗 参考资源

- [Kubernetes 官方文档](https://kubernetes.io/zh-cn/docs/)
- [Kubernetes GitHub](https://github.com/kubernetes/kubernetes)
- [CNCF 云原生基金会](https://www.cncf.io/)

---

#K8s #容器 #云原生 #DevOps
