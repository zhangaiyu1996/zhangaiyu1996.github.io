---
layout: post
title: "Kubernetes (K8s) 完全入门指南 - 从零开始玩转容器编排"
date: 2026-02-25 23:45:00 +0800
categories: [kubernetes, docker]
tags: [k8s, kubernetes, 容器, 云原生, DevOps]
---

# Kubernetes (K8s) 完全入门指南

> 🎯 本文目标：让你从"K8s是啥"到"我能用K8s部署应用"，全程大白话讲解！

## 一、K8s 是什么？为什么要用它？

### 1.1 一句话解释

**Kubernetes（简称 K8s）就是一个"容器管家"**，帮你自动管理成百上千个 Docker 容器。

### 1.2 生活化比喻

想象你开了一家外卖店：

| 场景 | 没有 K8s | 有 K8s |
|------|----------|--------|
| 订单暴增 | 你手忙脚乱地开新灶台 | 自动开新灶台应对 |
| 某个灶台坏了 | 你得自己发现并修理 | 自动发现并换一个新的 |
| 分配订单 | 你得记住哪个灶台空闲 | 自动把订单分给空闲灶台 |
| 升级菜谱 | 停业装修 | 边营业边悄悄升级 |

**K8s 就是那个全自动的店长！**

### 1.3 K8s 能帮你做什么？

- ✅ **自动扩缩容**：流量大了自动加机器，流量小了自动减机器
- ✅ **自动恢复**：程序挂了自动重启
- ✅ **负载均衡**：自动把请求分发到不同的容器
- ✅ **滚动更新**：不停机升级应用
- ✅ **配置管理**：统一管理环境变量、密钥等

---

## 二、K8s 核心概念（必须掌握！）

### 2.1 Pod - 最小单位

**Pod 就像一个"小房间"**，里面住着一个或多个容器。

```
┌─────────────────────────┐
│         Pod             │
│  ┌─────┐    ┌─────┐    │
│  │容器1│    │容器2│    │
│  └─────┘    └─────┘    │
│     共享网络和存储       │
└─────────────────────────┘
```

**记住**：K8s 不直接管理容器，而是管理 Pod。

### 2.2 Deployment - 部署管理器

**Deployment 就像一个"包工头"**，负责：
- 确保有指定数量的 Pod 在运行
- Pod 挂了自动创建新的
- 管理应用的升级和回滚

### 2.3 Service - 服务入口

**Service 就像一个"前台接待"**：
- 给一组 Pod 提供统一的访问入口
- 自动负载均衡
- Pod 换了 IP 也不影响访问

### 2.4 Namespace - 命名空间

**Namespace 就像"楼层"**：
- 把不同项目/环境隔离开
- 比如：`dev`（开发）、`test`（测试）、`prod`（生产）

### 2.5 概念关系图

```
Namespace (楼层)
    │
    ├── Deployment (包工头)
    │       │
    │       └── Pod (房间)
    │             │
    │             └── Container (住户)
    │
    └── Service (前台)
            │
            └── 指向一组 Pod
```

---

## 三、动手实践：部署你的第一个应用

### 3.1 前置准备

确保你已经安装了：
- Docker
- kubectl（K8s 命令行工具）
- 一个 K8s 集群（可以用 minikube 本地搭建）

```bash
# 检查 kubectl 是否安装
kubectl version --client

# 检查集群连接
kubectl cluster-info
```

### 3.2 第一个 Deployment

创建文件 `my-app.yaml`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx          # 部署名称
  labels:
    app: nginx
spec:
  replicas: 3             # 运行 3 个副本
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
        image: nginx:latest
        ports:
        - containerPort: 80
```

**应用配置：**

```bash
kubectl apply -f my-app.yaml
```

**查看结果：**

```bash
# 查看 Deployment
kubectl get deployments

# 查看 Pod
kubectl get pods

# 输出类似：
# NAME                        READY   STATUS    RESTARTS   AGE
# my-nginx-6b7f6cb5b7-abc12   1/1     Running   0          30s
# my-nginx-6b7f6cb5b7-def34   1/1     Running   0          30s
# my-nginx-6b7f6cb5b7-ghi56   1/1     Running   0          30s
```

### 3.3 创建 Service 暴露服务

创建文件 `my-service.yaml`：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-service
spec:
  selector:
    app: nginx            # 选择带有 app=nginx 标签的 Pod
  ports:
  - port: 80              # Service 端口
    targetPort: 80        # Pod 端口
  type: NodePort          # 暴露到节点端口
```

**应用并查看：**

```bash
kubectl apply -f my-service.yaml
kubectl get services

# 输出类似：
# NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# my-nginx-service   NodePort   10.96.123.456   <none>        80:31234/TCP   10s
```

现在可以通过 `节点IP:31234` 访问你的 Nginx 了！

---

## 四、常用 kubectl 命令速查表

### 4.1 查看资源

```bash
# 查看所有 Pod
kubectl get pods

# 查看所有 Pod（更多信息）
kubectl get pods -o wide

# 查看所有 Deployment
kubectl get deployments

# 查看所有 Service
kubectl get services

# 查看所有资源
kubectl get all

# 查看指定命名空间的资源
kubectl get pods -n kube-system
```

### 4.2 查看详情

```bash
# 查看 Pod 详情
kubectl describe pod <pod名称>

# 查看 Pod 日志
kubectl logs <pod名称>

# 实时查看日志
kubectl logs -f <pod名称>

# 查看容器内部
kubectl exec -it <pod名称> -- /bin/bash
```

### 4.3 创建和删除

```bash
# 应用配置文件
kubectl apply -f xxx.yaml

# 删除资源
kubectl delete -f xxx.yaml

# 直接删除 Pod
kubectl delete pod <pod名称>

# 删除 Deployment
kubectl delete deployment <deployment名称>
```

### 4.4 扩缩容

```bash
# 扩容到 5 个副本
kubectl scale deployment my-nginx --replicas=5

# 缩容到 2 个副本
kubectl scale deployment my-nginx --replicas=2
```

### 4.5 更新和回滚

```bash
# 更新镜像
kubectl set image deployment/my-nginx nginx=nginx:1.19

# 查看更新状态
kubectl rollout status deployment/my-nginx

# 查看历史版本
kubectl rollout history deployment/my-nginx

# 回滚到上一版本
kubectl rollout undo deployment/my-nginx

# 回滚到指定版本
kubectl rollout undo deployment/my-nginx --to-revision=2
```

---

## 五、实战案例：部署一个完整的 Web 应用

### 5.1 场景描述

部署一个前后端分离的应用：
- 前端：Nginx 静态页面
- 后端：Node.js API
- 数据库：MySQL

### 5.2 目录结构

```
k8s-demo/
├── namespace.yaml
├── mysql/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── secret.yaml
├── backend/
│   ├── deployment.yaml
│   └── service.yaml
└── frontend/
    ├── deployment.yaml
    └── service.yaml
```

### 5.3 创建命名空间

`namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-app
```

### 5.4 MySQL 配置

`mysql/secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: my-app
type: Opaque
data:
  password: bXlwYXNzd29yZA==  # base64 编码的 "mypassword"
```

`mysql/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
```

`mysql/service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: my-app
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
  type: ClusterIP  # 只在集群内部访问
```

### 5.5 一键部署

```bash
# 创建命名空间
kubectl apply -f namespace.yaml

# 部署 MySQL
kubectl apply -f mysql/

# 部署后端
kubectl apply -f backend/

# 部署前端
kubectl apply -f frontend/

# 查看所有资源
kubectl get all -n my-app
```

---

## 六、常见问题排查

### 6.1 Pod 一直 Pending

**可能原因：**
- 资源不足（CPU/内存）
- 没有可用节点

**排查命令：**
```bash
kubectl describe pod <pod名称>
# 查看 Events 部分
```

### 6.2 Pod 一直 CrashLoopBackOff

**可能原因：**
- 应用启动失败
- 配置错误

**排查命令：**
```bash
# 查看日志
kubectl logs <pod名称>

# 查看上一次崩溃的日志
kubectl logs <pod名称> --previous
```

### 6.3 Service 无法访问

**可能原因：**
- selector 标签不匹配
- 端口配置错误

**排查命令：**
```bash
# 检查 Service 的 Endpoints
kubectl get endpoints <service名称>

# 如果 Endpoints 为空，说明没有匹配到 Pod
```

### 6.4 镜像拉取失败

**可能原因：**
- 镜像名称错误
- 私有仓库需要认证

**解决方案：**
```bash
# 创建 docker registry secret
kubectl create secret docker-registry my-registry \
  --docker-server=xxx \
  --docker-username=xxx \
  --docker-password=xxx
```

---

## 七、进阶学习路线

### 7.1 下一步学什么？

1. **ConfigMap** - 配置管理
2. **PersistentVolume** - 持久化存储
3. **Ingress** - HTTP 路由
4. **HPA** - 自动扩缩容
5. **Helm** - K8s 包管理器

### 7.2 推荐资源

- 📚 官方文档：https://kubernetes.io/zh/docs/
- 🎮 在线练习：https://killercoda.com/
- 📖 《Kubernetes in Action》

---

## 八、总结

### 核心概念回顾

| 概念 | 作用 | 比喻 |
|------|------|------|
| Pod | 运行容器的最小单位 | 房间 |
| Deployment | 管理 Pod 的副本数量 | 包工头 |
| Service | 提供稳定的访问入口 | 前台 |
| Namespace | 资源隔离 | 楼层 |

### 常用命令回顾

```bash
kubectl get pods          # 查看 Pod
kubectl describe pod xxx  # 查看详情
kubectl logs xxx          # 查看日志
kubectl apply -f xxx.yaml # 应用配置
kubectl delete -f xxx.yaml # 删除资源
kubectl scale deployment xxx --replicas=N  # 扩缩容
```

---

**🎉 恭喜你！现在你已经掌握了 K8s 的基础知识，可以开始动手实践了！**

有问题欢迎留言讨论～
