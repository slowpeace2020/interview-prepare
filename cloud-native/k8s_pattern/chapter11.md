# 第11章：无状态服务

## 引言
“无状态服务”描述了用于管理相同应用实例的构建模块。无状态服务模式介绍了如何创建和操作由相同、短暂的副本组成的应用程序。这类应用非常适合动态的云环境，在这些环境中，它们可以迅速扩展并保持高可用性。

### 问题定义
无状态服务不会在实例内部存储跨服务交互的状态。在本文的上下文中，如果容器不在其内部存储（如内存或临时文件系统）中保存任何用于未来请求的关键信息，则该容器被认为是无状态的。无状态进程没有关于过去请求的知识或引用，因此每个请求都被当作全新请求处理。如果进程需要存储此类信息，它应将其存储在外部存储中，如数据库、消息队列、挂载的文件系统或其他可被其他实例访问的数据存储中。

无状态服务由相同、可替换的实例组成，这些实例通常将状态转移到外部的永久存储系统，并通过负载均衡器在它们之间分配传入请求。本章将具体介绍 Kubernetes 中哪些抽象帮助操作此类无状态应用程序。

## 解决方案
用于管理无状态 Pod 的控制器是 **ReplicaSet**，但它是由 **Deployment** 管理的较低级别的内部控制结构。Deployment 是推荐的用户操作无状态应用的抽象，负责创建和管理后台的 ReplicaSet。当 Deployment 提供的更新策略不适用，或者需要自定义机制时，可以直接使用 ReplicaSet。Deployment 允许控制副本的升级和回滚，而 ReplicaSet 负责维持指定数量的 Pod 副本。

### 实例管理
ReplicaSet 的主要目的是确保指定数量的相同 Pod 副本始终在运行。ReplicaSet 的定义包括三个重要部分：
1. **副本数**：指定 ReplicaSet 应维护的 Pod 数量。
2. **选择器**：指定如何识别其管理的 Pod。
3. **Pod 模板**：用于创建新 Pod 副本。

无论是通过 ReplicaSet 还是 Deployment 创建，最终都会保证有指定数量的相同 Pod 副本在运行。使用 Deployment 的额外好处是可以控制副本的升级和回滚。ReplicaSet 会自动处理 Pod 的重启以及随着副本数量的增加或减少进行的横向扩展或缩减。

**示例：**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
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
      - name: my-container
        image: my-image
```
在这个例子中，`my-deployment` 部署了 3 个 `my-app` 的副本，这些副本通过 `app: my-app` 标签选择器识别。

### 网络管理
ReplicaSet 会创建新的 Pod，这些 Pod 会有新的名称、主机名和 IP 地址。对于无状态的应用程序，每个 Pod 可以独立处理新请求。然而，由于 Pod 的 IP 地址在每次 Pod 重启时都会改变，建议使用基于 Kubernetes `Service` 的永久 IP 地址。`Service` 提供固定的 Cluster IP，确保客户端请求始终负载均衡到健康且准备好接受请求的 Pod 上。

**示例：**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  clusterIP: 10.0.0.1
```
在这个例子中，`my-service` 将请求路由到 `app: my-app` 标签匹配的所有 Pod 上。即使 Pod 的 IP 地址发生变化，`Service` 提供的固定 `clusterIP`（如 10.0.0.1）仍然是应用的入口。

### 存储管理
虽然无状态服务不保存内部状态，但它们通常需要外部存储来保存数据。Kubernetes 提供了 `PersistentVolume`（PV）和 `PersistentVolumeClaim`（PVC）来支持持久化存储。Pod 通过 PVC 请求并绑定到 PV，这种间接连接使 Pod 的生命周期与存储的生命周期分离。

**PersistentVolumeClaim 示例：**
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
      storage: 1Gi
```
这个 PVC 请求 1GiB 的存储空间，并以 `ReadWriteOnce` 模式访问。它可以在 Pod 模板中引用：

```yaml
volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

**不同的访问模式：**
- **ReadWriteOnce**：只能由一个节点挂载，但该节点上的多个 Pod 可以读写。
- **ReadOnlyMany**：可以被多个节点挂载，但只允许读操作。
- **ReadWriteMany**：可以被多个节点挂载，并允许读写操作。
- **ReadWriteOncePod**：仅允许一个 Pod 读写，保证数据一致性。

## 总结
无状态服务由相同、可替换的短暂实例组成，适合处理短生命周期的请求，并能够快速横向扩展或缩减。ReplicaSet 负责确保 Pod 的数量和状态，Deployment 则提供升级和回滚的控制。Service 作为入口，提供负载均衡和固定 IP 地址。对于需要持久化数据的无状态应用，PVC 提供了存储分离的机制。

开发者在使用 Kubernetes 构建无状态应用时，应该理解这些组件的协同工作方式，确保应用的可扩展性和高可用性。