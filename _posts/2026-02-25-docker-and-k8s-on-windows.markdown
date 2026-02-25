---
title: "Windows 下 Docker 和 Kubernetes 入门指南"
date: 2026-02-25 12:00:00 +0800
categories: [docker, kubernetes]
---

这篇文章面向刚接触容器技术的后端开发者，帮你在 Windows 上快速跑起来 Docker 和 Kubernetes。

## 前置准备

- Windows 10/11 专业版或企业版（需要 Hyper-V 支持）
- 至少 8GB 内存，建议 16GB
- 开启 BIOS 中的虚拟化（VT-x / AMD-V）

## 安装 Docker Desktop

1. 从 [Docker 官网](https://www.docker.com/products/docker-desktop/) 下载 Docker Desktop
2. 安装时选择 WSL 2 后端（推荐）
3. 安装完成后重启电脑

打开终端验证：

```bash
docker --version
docker run hello-world
```

看到 `Hello from Docker!` 就说明安装成功了。

## Docker 基本用法

### 跑一个容器

```bash
# 拉取 nginx 镜像并启动
docker run -d -p 8080:80 --name my-nginx nginx
```

浏览器打开 `http://localhost:8080`，能看到 nginx 欢迎页。

### 常用命令

```bash
docker ps              # 查看运行中的容器
docker ps -a           # 查看所有容器（包括已停止的）
docker stop my-nginx   # 停止容器
docker rm my-nginx     # 删除容器
docker images          # 查看本地镜像
docker logs my-nginx   # 查看容器日志
```

### 用 Dockerfile 构建镜像

假设你有一个 Spring Boot 项目，在项目根目录创建 `Dockerfile`：

```dockerfile
FROM openjdk:17-jdk-slim
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

构建并运行：

```bash
docker build -t my-app .
docker run -d -p 8080:8080 my-app
```

### Docker Compose

多个服务一起跑时用 Compose。创建 `docker-compose.yml`：

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: mydb
    ports:
      - "3306:3306"
```

```bash
docker compose up -d    # 启动所有服务
docker compose down     # 停止并清理
docker compose logs -f  # 查看日志
```

## 启用 Kubernetes

Docker Desktop 自带 Kubernetes，只需要开启：

1. 打开 Docker Desktop → Settings → Kubernetes
2. 勾选 Enable Kubernetes
3. 点 Apply & Restart，等几分钟拉取镜像

验证：

```bash
kubectl version
kubectl get nodes
```

看到一个 `Ready` 状态的节点就行了。

## Kubernetes 基本用法

### 核心概念

- Pod：最小部署单元，里面跑一个或多个容器
- Deployment：管理 Pod 的副本数量和更新策略
- Service：给 Pod 提供稳定的访问入口

### 部署一个应用

创建 `deployment.yaml`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
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
          image: my-app:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080
```

`imagePullPolicy: Never` 表示用本地 Docker 构建的镜像，不从远程拉取。

```bash
kubectl apply -f deployment.yaml
kubectl get pods
kubectl get svc
```

浏览器访问 `http://localhost:30080`。

### 常用 kubectl 命令

```bash
kubectl get pods                    # 查看 Pod
kubectl get svc                     # 查看 Service
kubectl logs <pod-name>             # 查看日志
kubectl describe pod <pod-name>     # 查看 Pod 详情
kubectl exec -it <pod-name> -- sh  # 进入容器
kubectl delete -f deployment.yaml   # 删除部署的资源
kubectl scale deployment my-app --replicas=3  # 扩容到 3 个副本
```

## 开发建议

- 本地开发先用 Docker Compose，够用且简单
- 需要模拟生产环境或学习编排时再上 Kubernetes
- 镜像尽量用 slim 或 alpine 版本，体积小启动快
- 敏感信息不要写在镜像里，用环境变量或 K8s Secret 管理
