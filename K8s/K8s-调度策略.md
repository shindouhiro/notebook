# K8s 调度策略

## 🎯 调度器工作原理

```
┌─────────────────────────────────────────────────────────────┐
│                    Scheduler 调度流程                        │
│                                                              │
│   新 Pod 创建 ──► 调度队列 ──► 过滤节点 ──► 打分排序 ──► 绑定  │
│                      │           │           │               │
│                      ▼           ▼           ▼               │
│               Scheduling    Filtering    Scoring            │
│                 Queue       (预选)       (优选)              │
└─────────────────────────────────────────────────────────────┘
```

---

## 📍 nodeName 与 nodeSelector

### nodeName（直接指定）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node-1  # 直接指定节点，跳过调度器
  containers:
  - name: nginx
    image: nginx
```

### nodeSelector（标签选择）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeSelector:
    disktype: ssd
    zone: asia-east1
  containers:
  - name: nginx
    image: nginx
```

```bash
# 给节点添加标签
kubectl label nodes node-1 disktype=ssd
kubectl label nodes node-1 zone=asia-east1

# 查看节点标签
kubectl get nodes --show-labels
```

---

## 💪 Node Affinity（节点亲和性）

比 nodeSelector 更强大、更灵活。

### 硬亲和性（Required）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
          - key: disktype
            operator: In
            values:
            - ssd
            - nvme
  containers:
  - name: nginx
    image: nginx
```

### 软亲和性（Preferred）

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80  # 权重 1-100
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - zone-a
      - weight: 20
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - zone-b
```

### 操作符

| 操作符 | 含义 |
|-------|------|
| In | 在值列表中 |
| NotIn | 不在值列表中 |
| Exists | 标签存在 |
| DoesNotExist | 标签不存在 |
| Gt | 大于 |
| Lt | 小于 |

---

## 🤝 Pod Affinity/Anti-Affinity

控制 Pod 之间的调度关系。

### Pod 亲和性（放在一起）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: cache
        topologyKey: kubernetes.io/hostname  # 同一节点
  containers:
  - name: frontend
    image: nginx
```

### Pod 反亲和性（分散部署）

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: web
            topologyKey: kubernetes.io/hostname
      containers:
      - name: nginx
        image: nginx
```

**效果：** 确保每个节点最多运行一个 web Pod

### topologyKey 常用值

| Key | 含义 |
|-----|------|
| kubernetes.io/hostname | 节点级别 |
| topology.kubernetes.io/zone | 可用区级别 |
| topology.kubernetes.io/region | 区域级别 |

---

## ☠️ Taints 与 Tolerations

用于排斥 Pod 或允许特定 Pod 调度到节点。

### Taint 污点

```bash
# 添加污点
kubectl taint nodes node-1 key=value:NoSchedule
kubectl taint nodes node-1 gpu=true:NoExecute

# 查看污点
kubectl describe node node-1 | grep Taints

# 移除污点
kubectl taint nodes node-1 key=value:NoSchedule-
```

### 污点效果

| 效果 | 说明 |
|-----|------|
| **NoSchedule** | 不调度新 Pod |
| **PreferNoSchedule** | 尽量不调度 |
| **NoExecute** | 不调度 + 驱逐已有 Pod |

### Toleration 容忍

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  - key: "node.kubernetes.io/not-ready"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300  # 容忍 5 分钟
  containers:
  - name: cuda
    image: nvidia/cuda
```

### 常见系统污点

| 污点 | 说明 |
|-----|------|
| node.kubernetes.io/not-ready | 节点未就绪 |
| node.kubernetes.io/unreachable | 节点不可达 |
| node.kubernetes.io/disk-pressure | 磁盘压力 |
| node.kubernetes.io/memory-pressure | 内存压力 |
| node.kubernetes.io/pid-pressure | PID 压力 |

---

## ⚖️ 资源调度

### 资源请求与限制

```yaml
spec:
  containers:
  - name: app
    image: myapp
    resources:
      requests:      # 调度依据
        memory: "64Mi"
        cpu: "250m"   # 0.25 核
      limits:        # 运行限制
        memory: "128Mi"
        cpu: "500m"
```

### LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - type: Container
    default:        # 默认 limits
      cpu: "500m"
      memory: "128Mi"
    defaultRequest: # 默认 requests
      cpu: "100m"
      memory: "64Mi"
    max:           # 最大值
      cpu: "2"
      memory: "1Gi"
    min:           # 最小值
      cpu: "50m"
      memory: "32Mi"
```

### ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
```

---

## 🎖️ Priority 与 Preemption

### PriorityClass

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "高优先级任务"
```

### 使用优先级

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: important-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: myapp
```

---

## 📊 调度策略总结

```
┌─────────────────────────────────────────────────────────────┐
│                    调度策略选择指南                          │
│                                                              │
│  需求：指定节点硬件            →  nodeSelector / Affinity    │
│  需求：Pod 就近部署            →  Pod Affinity              │
│  需求：Pod 分散部署（高可用）   →  Pod Anti-Affinity         │
│  需求：节点专用（GPU/存储）     →  Taints + Tolerations      │
│  需求：资源隔离               →  ResourceQuota              │
│  需求：优先保证核心业务        →  Priority                  │
└─────────────────────────────────────────────────────────────┘
```

---

返回 [[K8s快速入门]]

#K8s #调度 #Affinity #Taint
