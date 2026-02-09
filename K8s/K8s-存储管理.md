# K8s 存储管理

## 📦 存储概述

Kubernetes 提供了灵活的存储抽象层，将存储的提供与消费分离。

```
┌─────────────────────────────────────────────────────────┐
│                    存储架构                              │
│                                                          │
│   Pod                                                    │
│    │                                                     │
│    ▼                                                     │
│   Volume  ←── 挂载点                                     │
│    │                                                     │
│    ▼                                                     │
│   PVC (PersistentVolumeClaim) ←── 用户申请              │
│    │                                                     │
│    ▼                                                     │
│   PV (PersistentVolume) ←── 管理员提供                  │
│    │                                                     │
│    ▼                                                     │
│   实际存储（NFS、云盘、本地磁盘等）                       │
└─────────────────────────────────────────────────────────┘
```

---

## 🗂️ Volume 类型

### 临时存储

| 类型 | 说明 | 生命周期 |
|-----|------|---------|
| **emptyDir** | 临时空目录 | Pod 生命周期 |
| **configMap** | 配置文件挂载 | ConfigMap 存在 |
| **secret** | 敏感数据挂载 | Secret 存在 |

#### emptyDir 示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - name: test-container
    image: busybox
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```

### 持久化存储

| 类型 | 说明 |
|-----|------|
| **hostPath** | 宿主机路径挂载 |
| **nfs** | NFS 网络存储 |
| **persistentVolumeClaim** | PVC 申请 |
| **csi** | 容器存储接口 |

---

## 💾 PersistentVolume (PV)

PV 是集群资源，由管理员创建或动态供应。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany        # 访问模式
  persistentVolumeReclaimPolicy: Retain  # 回收策略
  storageClassName: nfs-storage
  nfs:
    server: 192.168.1.100
    path: /data/nfs
```

### 访问模式

| 模式 | 缩写 | 说明 |
|-----|-----|------|
| ReadWriteOnce | RWO | 单节点读写 |
| ReadOnlyMany | ROX | 多节点只读 |
| ReadWriteMany | RWX | 多节点读写 |
| ReadWriteOncePod | RWOP | 单 Pod 读写（K8s 1.22+） |

### 回收策略

| 策略 | 说明 |
|-----|------|
| **Retain** | 保留数据，手动回收 |
| **Delete** | 删除 PV 和后端存储 |
| **Recycle** | 已废弃，清除数据后复用 |

---

## 📝 PersistentVolumeClaim (PVC)

PVC 是用户对存储的申请。

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: nfs-storage
```

### 在 Pod 中使用 PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

---

## ⚙️ StorageClass

StorageClass 用于动态存储供应。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs  # 供应商
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true  # 允许扩容
```

### 常见 Provisioner

| 云厂商 | Provisioner |
|-------|-------------|
| AWS EBS | kubernetes.io/aws-ebs |
| GCP PD | kubernetes.io/gce-pd |
| Azure Disk | kubernetes.io/azure-disk |
| 阿里云 | alicloud/disk |
| NFS | nfs-subdir-external-provisioner |

### 动态供应流程

```
用户创建 PVC (指定 StorageClass)
        │
        ▼
StorageClass 触发 Provisioner
        │
        ▼
自动创建 PV
        │
        ▼
PVC 与 PV 绑定
        │
        ▼
Pod 挂载使用
```

---

## 📊 PV/PVC 状态

### PV 状态

| 状态 | 说明 |
|-----|------|
| Available | 可用，未绑定 |
| Bound | 已绑定到 PVC |
| Released | PVC 已删除，等待回收 |
| Failed | 自动回收失败 |

### PVC 状态

| 状态 | 说明 |
|-----|------|
| Pending | 等待绑定 PV |
| Bound | 已绑定 |
| Lost | 绑定的 PV 已丢失 |

---

## 🛠️ 常用命令

```bash
# 查看 PV
kubectl get pv
kubectl describe pv <pv-name>

# 查看 PVC
kubectl get pvc
kubectl describe pvc <pvc-name>

# 查看 StorageClass
kubectl get sc
kubectl describe sc <sc-name>

# 扩容 PVC（需 StorageClass 支持）
kubectl patch pvc my-pvc -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'
```

---

## 💡 最佳实践

1. **使用 StorageClass** 实现动态供应
2. **根据场景选择访问模式**：Web 应用用 RWO，共享存储用 RWX
3. **设置合理的资源配额**
4. **生产环境使用 Retain 策略** 防止数据丢失
5. **定期备份持久化数据**

---

返回 [[K8s快速入门]]

#K8s #存储 #PV #PVC
