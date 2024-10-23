# 第13章：服务发现（Service Discovery）

## 解释
本章介绍了在 Kubernetes 集群中，客户端服务如何发现并消费提供服务的实例。服务发现解决了一个重要问题，即如何在动态变化的环境中（如 Pod 的启动、停止、扩展）找到可用的服务实例。

## 问题
如果我们需要手动跟踪、注册和发现动态 Kubernetes Pods 的端点，将是一项巨大的挑战。Kubernetes 通过多种机制实现了服务发现模式，简化了这一过程。本章详细探讨了这些机制。

## 解决方案
在“**Kubernetes 之前的时代**”，最常见的服务发现机制是**客户端发现**。在这种架构中，服务消费者必须依赖一个发现代理来从注册表中查找服务实例，然后选择一个进行调用。  
在“**Kubernetes 之后的时代**”，分布式系统的许多非功能性责任（如服务的部署、健康检查、自愈等）已经转移到平台上，服务发现和负载均衡也因此成为平台内置的功能。

虽然服务发现模式看起来简单，但根据服务消费者和提供者是否在同一集群内，使用的机制会有所不同。

### 内部服务发现（Internal Service Discovery）

Kubernetes 的 `Service` 资源解决了服务发现中的这一挑战。它为一组提供相同功能的 Pods 提供了一个稳定的入口点。通过 `kubectl expose` 命令，可以为 Deployment 或 ReplicaSet 中的 Pods 创建一个 `Service`。这会生成一个虚拟 IP（`clusterIP`），并使用 Pods 的选择器和端口号来定义服务。

一旦 `Service` 创建完毕，分配的 `clusterIP` 只在 Kubernetes 集群内可访问，并且在 `Service` 定义存在期间不会更改。然而，如何让集群内的其他应用得知这个动态分配的 `clusterIP` 呢？有两种方式：

#### 1. 通过环境变量进行发现
当创建服务时，Kubernetes 会自动将服务的相关信息（如 `clusterIP`）注入到启动的 Pods 环境变量中。  
**问题**：这种方法有个时间依赖性问题，环境变量只能注入到在 `Service` 创建之后启动的 Pods 中。如果先启动了 Pods，再创建 `Service`，需要重启 Pods 来获取服务信息。

#### 2. 通过 DNS 查询进行发现
Kubernetes 中每个 Pod 都会自动配置使用集群的 DNS 服务器。当创建新的 `Service` 时，它会在 DNS 中自动注册一个条目，所有 Pod 可以通过查询该 DNS 条目访问 `Service`。  
例如，客户端可以通过完全合格域名（FQDN）如 `random-generator.default.svc.cluster.local` 访问 `Service`。这避免了环境变量的时间依赖问题。

**解释**：DNS 服务发现不受环境变量注入的限制，任何时间创建的 Pods 都可以通过 DNS 查询到服务 IP。不过，如果使用非标准端口，可能仍然需要环境变量来获取端口号。

**例子**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: random-generator
spec:
  selector:
    app: random-generator
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
客户端可以通过 `random-generator.default.svc.cluster.local` 查找到服务。

### ClusterIP 服务的其他特性

#### 1. 多端口支持
一个 `Service` 可以支持多个源端口和目标端口。

#### 2. 会话亲和性
默认情况下，Kubernetes 服务是随机选择一个 Pod 处理请求。但可以通过 `sessionAffinity: ClientIP` 选项使同一客户端 IP 的请求总是转发到同一个 Pod。

#### 3. 就绪探针（Readiness Probes）
如果一个 Pod 的就绪检查失败，即使它的标签匹配选择器，它也会从服务端点列表中移除。

#### 4. 虚拟 IP
当创建类型为 `ClusterIP` 的服务时，它会分配一个虚拟 IP，虽然这个 IP 并不存在于实际的网络接口中，但 kube-proxy 会在每个节点上更新 iptables 规则，将发往该虚拟 IP 的网络包转发给实际的 Pod IP。

#### 5. 自定义 ClusterIP
在创建 `Service` 时，可以通过 `.spec.clusterIP` 字段指定一个特定的 IP 地址。这通常用于兼容需要使用固定 IP 的遗留应用程序。

**示例**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: custom-ip-service
spec:
  clusterIP: 10.0.0.50
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### 手动服务发现
可以通过手动管理服务的端点（endpoints）来实现外部资源的服务发现。例如，将流量转发到集群外部的数据库或 API 时，通常通过跳过选择器并手动创建端点资源来实现。

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: external-db
subsets:
  - addresses:
      - ip: 192.168.1.100
    ports:
      - port: 3306
```

这样，服务的 DNS 名称依然可以用于访问外部资源。

### 集群外的服务发现
Kubernetes 的 `ClusterIP` 类型服务只能在集群内部使用，而 `NodePort` 和 `LoadBalancer` 类型的服务可以将集群内的服务暴露给外部客户端。

#### 1. `NodePort` 类型服务
`NodePort` 会在集群中每个节点上打开一个端口，使外部客户端可以通过该端口访问服务。客户端可以连接到任何节点，但需要确保节点的健康状况。

#### 2. `LoadBalancer` 类型服务
在云提供商的基础设施上，`LoadBalancer` 服务会在集群外部创建一个负载均衡器，允许外部流量访问服务。

**示例**：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### 应用层服务发现
`Ingress` 是一种高级的服务发现机制，它位于服务之前，提供 HTTP 路由、负载均衡、TLS 终止等功能。它允许多个服务共享同一个外部负载均衡器和 IP，从而降低基础设施成本。

**示例**：
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - host: my-app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

### 讨论
集群内的服务发现通常通过 `Service` 资源实现，不同选项可适应不同场景。而集群外的服务发现则基于 `NodePort` 或 `LoadBalancer`。 `Ingress` 提供了更灵活的方式，将多个服务暴露给外部。

这使得 Kubernetes 可以通过平台集成服务发现功能，而无需额外的外部工具。