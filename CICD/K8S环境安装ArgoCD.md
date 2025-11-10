# K8S环境下安装ArgoCD

## 验证环境
```declarative
kubetl get nodes
```

## 创建 Argo CD Namespace
```declarative
kubectl create namespace argocd
```

## 安装 Argo CD
```declarative
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
### 查看 Argo CD 状态
```declarative
kubectl get pods -n argocd
```
你会看到 Deployment、Service、Pod、ConfigMap 等资源。
```declarative
[root@localhost ~]# kubectl get pod -n argocd
NAME                                               READY   STATUS    RESTARTS       AGE
argocd-application-controller-0                    1/1     Running   1 (178m ago)   24h
argocd-applicationset-controller-79887fd6-vg5cg    1/1     Running   1 (178m ago)   24h
argocd-dex-server-655b4448b5-nkmxx                 1/1     Running   1 (178m ago)   24h
argocd-notifications-controller-77fd6f9885-6c9rw   1/1     Running   1 (178m ago)   24h
argocd-redis-7fdcfb697b-5mj2j                      1/1     Running   1 (178m ago)   24h
argocd-repo-server-7d969c8c68-r4rst                1/1     Running   1 (178m ago)   24h
argocd-server-757b96bd6b-m6qcd                     1/1     Running   1 (178m ago)   23h

```
## 访问 Argo CD
Argo CD 默认使用 Service 类型 ClusterIP，不能直接外部访问。
### 方式1: 端口转发（适合本地测试）
```declarative
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
### 方式二：暴露 NodePort/LoadBalancer（适合远程服务器）
```declarative
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc argocd-server -n argocd

```
之后浏览器通过IP+端口访问
## 登录ArgoCD
默认账号是 admin，密码为 Argo CD API Server 的初始密码
```declarative
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
登录之后页面可以修改登录密码
