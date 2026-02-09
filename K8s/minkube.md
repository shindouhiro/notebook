+ 清理回收
```bash
minikube delete
```

+ 创建deployment

```bash
kubectl create deployment nginx --image=nginx
```
+ 创建Service

```bash
kubectl expose deployment nginx --type=NodePort --port=80
```