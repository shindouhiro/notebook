# K8s 常用命令速查表

## 📋 基础命令

### 集群信息

```bash
# 集群信息
kubectl cluster-info
kubectl version

# 查看配置
kubectl config view
kubectl config current-context
kubectl config get-contexts

# 切换上下文
kubectl config use-context <context-name>

# 设置默认命名空间
kubectl config set-context --current --namespace=<namespace>
```

### 节点操作

```bash
# 查看节点
kubectl get nodes
kubectl get nodes -o wide

# 节点详情
kubectl describe node <node-name>

# 节点标签
kubectl label node <node-name> <key>=<value>
kubectl label node <node-name> <key>-  # 删除标签

# 节点污点
kubectl taint node <node-name> <key>=<value>:<effect>
kubectl taint node <node-name> <key>-  # 删除污点

# 节点维护
kubectl cordon <node-name>    # 标记不可调度
kubectl uncordon <node-name>  # 恢复可调度
kubectl drain <node-name>     # 驱逐 Pod 并标记不可调度
```

---

## 📦 Pod 操作

```bash
# 查看 Pod
kubectl get pods
kubectl get pods -A                  # 所有命名空间
kubectl get pods -o wide             # 详细信息
kubectl get pods -o yaml             # YAML 格式
kubectl get pods -w                  # 监听变化
kubectl get pods --show-labels       # 显示标签
kubectl get pods -l app=nginx        # 按标签过滤

# Pod 详情
kubectl describe pod <pod-name>

# 创建 Pod
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# 删除 Pod
kubectl delete pod <pod-name>
kubectl delete pod <pod-name> --force --grace-period=0  # 强制删除

# 进入 Pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh

# 复制文件
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file

# 端口转发
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward svc/<svc-name> 8080:80
```

---

## 📝 日志与调试

```bash
# 查看日志
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # 多容器
kubectl logs -f <pod-name>                   # 实时追踪
kubectl logs --tail=100 <pod-name>           # 最后 100 行
kubectl logs --since=1h <pod-name>           # 最近 1 小时
kubectl logs -p <pod-name>                   # 上一个容器日志

# 事件
kubectl get events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events --field-selector reason=Failed

# 资源使用
kubectl top nodes
kubectl top pods
kubectl top pods --containers

# Debug 容器（K8s 1.23+）
kubectl debug -it <pod-name> --image=busybox

# 临时调试 Pod
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash
```

---

## 🚀 Deployment 操作

```bash
# 查看
kubectl get deployments
kubectl get deploy -o wide
kubectl describe deploy <deploy-name>

# 创建
kubectl create deployment nginx --image=nginx
kubectl create deployment nginx --image=nginx --replicas=3

# 扩缩容
kubectl scale deployment <deploy-name> --replicas=5

# 更新镜像
kubectl set image deployment/<deploy-name> <container>=<new-image>

# 滚动更新状态
kubectl rollout status deployment/<deploy-name>

# 暂停/恢复滚动更新
kubectl rollout pause deployment/<deploy-name>
kubectl rollout resume deployment/<deploy-name>

# 回滚
kubectl rollout history deployment/<deploy-name>
kubectl rollout undo deployment/<deploy-name>
kubectl rollout undo deployment/<deploy-name> --to-revision=2

# 重启
kubectl rollout restart deployment/<deploy-name>
```

---

## 🔗 Service 操作

```bash
# 查看
kubectl get svc
kubectl get svc -o wide
kubectl describe svc <svc-name>

# 创建
kubectl expose deployment <deploy-name> --port=80 --target-port=8080
kubectl expose deployment <deploy-name> --type=NodePort --port=80

# 查看 Endpoints
kubectl get endpoints
kubectl get ep <svc-name>

# 临时访问Service
kubectl port-forward svc/<svc-name> 8080:80
```

---

## 📁 ConfigMap 与 Secret

```bash
# ConfigMap
kubectl create configmap <cm-name> --from-literal=key=value
kubectl create configmap <cm-name> --from-file=config.properties
kubectl create configmap <cm-name> --from-file=./config-dir/

kubectl get configmap
kubectl describe configmap <cm-name>
kubectl get configmap <cm-name> -o yaml

# Secret
kubectl create secret generic <secret-name> --from-literal=password=123456
kubectl create secret generic <secret-name> --from-file=./cert.pem
kubectl create secret tls <secret-name> --cert=tls.crt --key=tls.key

kubectl get secrets
kubectl describe secret <secret-name>
kubectl get secret <secret-name> -o jsonpath='{.data.password}' | base64 -d
```

---

## 💾 存储操作

```bash
# PersistentVolume
kubectl get pv
kubectl describe pv <pv-name>

# PersistentVolumeClaim
kubectl get pvc
kubectl describe pvc <pvc-name>

# StorageClass
kubectl get sc
kubectl describe sc <sc-name>
```

---

## 🔐 RBAC

```bash
# 查看权限
kubectl auth can-i create pods
kubectl auth can-i create pods --as=user1
kubectl auth can-i --list

# ServiceAccount
kubectl get sa
kubectl create sa <sa-name>

# Role/ClusterRole
kubectl get roles
kubectl get clusterroles
kubectl describe role <role-name>

# RoleBinding/ClusterRoleBinding
kubectl get rolebindings
kubectl get clusterrolebindings
```

---

## 🏷️ 资源管理

```bash
# 命名空间
kubectl get namespaces
kubectl create namespace <ns-name>
kubectl delete namespace <ns-name>

# 资源配额
kubectl get resourcequotas
kubectl describe resourcequota <quota-name>

# LimitRange
kubectl get limitranges
kubectl describe limitrange <lr-name>
```

---

## 📄 YAML 操作

```bash
# Apply（创建或更新）
kubectl apply -f <file.yaml>
kubectl apply -f <directory>/
kubectl apply -f https://example.com/resource.yaml

# Delete
kubectl delete -f <file.yaml>

# Create（仅创建）
kubectl create -f <file.yaml>

# Replace（替换）
kubectl replace -f <file.yaml>

# Diff（对比差异）
kubectl diff -f <file.yaml>

# 生成 YAML 模板
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

---

## 🛠️ 实用技巧

### JSONPath 查询

```bash
# 获取所有 Pod 的镜像
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'

# 获取所有节点 IP
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# 获取指定 Pod 的 IP
kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'
```

### 快捷别名

```bash
# ~/.bashrc 或 ~/.zshrc
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kgn='kubectl get nodes'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
alias kdp='kubectl describe pod'
alias kds='kubectl describe svc'
alias kl='kubectl logs'
alias klf='kubectl logs -f'
alias ke='kubectl exec -it'

# 自动补全
source <(kubectl completion bash)  # bash
source <(kubectl completion zsh)   # zsh
```

### 常用资源缩写

| 全称 | 缩写 |
|-----|------|
| pods | po |
| services | svc |
| deployments | deploy |
| replicasets | rs |
| configmaps | cm |
| secrets | - |
| persistentvolumes | pv |
| persistentvolumeclaims | pvc |
| namespaces | ns |
| nodes | no |
| ingresses | ing |
| statefulsets | sts |
| daemonsets | ds |

---

## 🆘 故障排查

```bash
# Pod 无法启动
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
kubectl get events --sort-by='.lastTimestamp'

# 服务无法访问
kubectl get endpoints <svc-name>
kubectl run test --rm -it --image=busybox -- wget -O- <svc-name>

# 节点问题
kubectl describe node <node-name>
kubectl get node <node-name> -o yaml | grep -A 10 conditions

# 资源不足
kubectl top nodes
kubectl top pods
kubectl describe node | grep -A 5 "Allocated resources"
```

---

返回 [[K8s快速入门]]

#K8s #命令 #运维 #速查
