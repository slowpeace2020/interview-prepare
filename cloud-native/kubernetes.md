##[k8s基础概念](https://www.cnblogs.com/ZhuChangwu/p/16441181.html)

什么是k8s？
open source container orchestration tool
developed by google
helps you manage containerized applications in different environments

它解决什么问题？orchestration tool的任务是什么
现在的趋势是从monolith转到microservice，增加了container的使用
container有数百甚至数千个，就需要使用脚本和自制工具管理跨环境的容器负载，这很困难甚至不可能
所以就需要container orchestration tool.

它能提供：high availability or no downtime
scalability or high performance
disaster recovery, backup and restore

基本架构是什么？
a master node and a couple of slave nodes(has a kublet process)
kublet process: 节点之间相互通信，执行任务，比如运行应用程序
slave node也就是worker node, 部署了不同application的docker container，取决于工作负载的分布情况，在worker node上运行不同数量的docker容器
Master node运行了几个kubernetes process：
API Server:entrypoint to k8s cluster, 也是container，比如UI，API，Client都是和它交互
Controller manager: keeps track whats happening in the cluster,比如什么需要修复，container的状态怎么样，是否存活，是否需要重启
scheduler：根据每个节点的工作负载和资源调度安排container
etcd: key-value storage, 保存kubernetes cluster的状态，它所有的配置数据、每个节点和该节点每个容器的所有状态数据，备份和恢复实际上是由etcd快照创建的，可以通过它恢复整个集群的状态
virtual network：它把集群内的所有节点变成一台强大的机器，拥有所有节点的资源总和。
一般kubernetes cluster至少要2个Master nodes

pod: smallest unit as kubernetes user will configure and interact.它其实是wrapper of container, 一个worker node有多个pod，一个pod有多个container。通常而言，一个应用程序有一个pod，在这个pod放置多个container，是因为这个application需要一些helper containers.
每一个pod有自己的ip地址，pod之间的通信用这个ip。pod挂掉之后重新起一个ip会变
service背后是pod，但是它有permanent ip address，有两种类型一个是external，一个是internal, 它也是一个load balancer
external也叫Ingress，通常是app-name:port的方式访问而非ip:port
Ingress: handles every incoming request

ConfigMap: external configuration of your application,和一般的配置文件有什么不同呢？
Secret：used to store secret data, base64 encoded

Volumes：storage，可以是local machine，也可以Remote，不属于k8s cluster的一部分，因为k8s doesn't manage data persistance

Deployment：blueprint/abstraction for application Pods, for stateless Apps
DB can't be replicated via Deployment, and DB are oftn hosted outside of the K8s cluster
StatefulSet for stateful Apps or Databases

worker node: each node has multiple Pods on it, 3 processes must be installed on every node
container runtime: 比如docker
kubelet: interacts with both the container runtime and node, starts the pod with a container inside 
kube proxy: forwards the request

master node: 4 process run on every master node
API server: cluster gateway, acts as a gatekeeper for authentication
Scheduler: decides on which node new Pod should be scheduled
Controller Manager: detects cluster state changes, 如果有新的pod需要起，就告诉scheduler去安排
etcd: cluster brain, cluster changes get stored in the key values store, 但是application data 不存在这里

k8s怎么实现high availability和scalability
请求从外界进来，Ingress处理它，然后到各个service，every component is replicated, load balanced, no bottleneck that slows down the response

disaster recovery, backup and restore，通过master节点的controller manager和etcd实现


什么是minikube？
Master and worker node processes run on one machine
minikube creates virtual box on your laptop, node runs in that virtual box
1 node k8s cluster
for testing purpose

master processes: 也就是那四个，API Server: enable interaction with cluster


什么是kubectl? 
command line tool for k8s cluster


set-up minikube cluster
安装minikube
#启动cluster
minikube start --vm-driver=hyperkit
#得到节点信息
kubectl get nodes
#删除cluster
minikube delete
#以debug模式启动cluster
minikube start --vm-driver=hyperkit --v=7 --alsologtostderr

kubectl get pod
#创建pod
kubectl create deployment NAME --image=image [--dry-run] [options]

kubectl get deployment
kubectl get replicaset

#编辑deployment文件
kubectl edit deployment NAME

你可以使用以下命令查看与 nginx-depl-5dfbd7dbbd 相关的 Pod：
kubectl get pods --selector=app=nginx-depl-5dfbd7dbbd
#查看日志
kubectl logs <pod-name>

#进入pod的bash
kubectl exec -it <pod-name> -- bin/bash

#配置文件的方式创建deployment
kubectl apply -f config-file.yaml

CRUD commands
kubectl create/edit/delete deployment NAME

status of k8s components
kubectl get nodes/pod/services/replicaset/deployment

Debuging pods
log to console: kubectl logs [pod name]
get interactive terminal: kubectl exec -it [pod name] --bin/bash
get info about pod: kubectl describe pod [pod name] 

use configuration file for CRUD
Apply a configuration file:   kubectl apply -f [file name]
Delete with configuration file: kubectl delete -f [file name]

YAML configuration file in kubernetes
3个部分
metadata
specification
status有Desired和Actual，从etcd获取

pods get the label through the template blueprint
template上面的是Service，这个部分的selector和label是相互映证的


connecting Deployments to Services to Pods

Deployment manages a 
ReplicaSet manages a
Pod is abstraction of
Container



kubectl describe service [service name]
kubectl get deployment nginx-deployment -o yaml
kubectl get deployment nginx-deployment -o yaml > statusinfo.yaml
kubectl get pod -o wide
kubectl get deployment nginx-deployment -o yaml
kubectl get all | grep mongodb

Deployment 是用来管理和维护 Pods 的，它确保应用程序按照所定义的状态运行，并支持更新和扩展。
Service 则是用来暴露 Pods，提供稳定的网络访问，并实现负载均衡。它抽象了 Pod 的 IP 地址和端口，提供一个稳定的访问入口。
简而言之，Deployment 负责应用的运行状态，而 Service 负责应用的网络访问。













kubernetes configuration格式yaml或json
kind有哪些？
Deployment: a template for creating pods
Declarative
Is==Should
controller manager check desired==actual



Secret Configuration File
kind: Secret
metadata/name: a random name
type: "Opaque" - default for arbitrary key-value pairs
data: the actual contents in key-value pairs


Service Configuration File
kind: Service
meatadat/name: a random name
selector: to connect to Pod through label
ports:
  port: Service port
  targetPort: containerPort of Deployment

How to make it an external service?
把service的type设为LoadBalancer
作用是assign service an external IP address and so accepts external requests

nodePort: must between 30000-32767

inter service or Cluster IP is DEFAULT

ConfigMap用来做什么？
- external configuration
- centralized
- other components can use it

kind: ConfigMap
metadata/name: a random name
data: the actual contents - in key-value pairs


Namespace
在cluster内部的virtual cluster
cluster内部有四个默认的namespace
不要动kube-system, 它是System processes
kube-public:
- 存储publicly accessible data
- a configmap, which contains cluster information

kube-node-lease
- heartbeat of nodes
- each node has associated lease object in namespace
- determine the availability of a node

default
- resources you create are located here
- 


kubectl create namespace my-namespace
kubectl get namespace

create a namespace with a configuration file
example config content:

为什么要用namespace?
- resources grouped in namespace，资源分类
- conflicts: many teams, same application, 避免团队冲突，命名冲突，资源覆盖
- resources sharing: Staging and Development，蓝绿部署不同版本的应用，资源共享，re-use components in both environment
- Access and Resource Limits on Namespaces, each team has own, isolated environment,限制每个Namespace的CPU,RAM, Storage使用

Namespace的特点
you can't access most resources from another Namespace
each NS must define own ConfigMap
Access Service in another Namespace: namespace名字.service名字

怎么在Namespace创建component?
kubectl apply -f mysql-configmap.yaml --namespace=my-namespace

kubectl get configmap -n my-namespace

# change active namespace
先安装命令行支持：brew install kubectx
kubens [namespace]


External Service vs. Ingress
External Service example YAML file:
kind: Service
nodePort: 35010 这个application对外的端口号，也就是外界访问host:port的端口号

Ingress example YAML file:
kind: Ingress
有routing rules, forward request to the internal service
host, paths 对应host后面的路径转发规则

Ingress对应Internal Service，没有nodePort
Host: 
- valid domain address
- map domain name to Node's IP address, which is the entrypoint

implementation for Ingress 就是 Ingress Controller
- evaluate all the rules
- manages redirections
- entrypoint to cluster
- many third-party implementations
- k8s nginx Ingress Controller

# install Ingress Controller in Minikube
minikube addons enable ingress
#create ingress rules
yaml example

mulitple paths for same host
multiple sub-domains or domains

configuring TLS certification example

# Helm
- package manager for kubernetes: to package YAML files and distribute them in public and private repositories
- create your own Helm Charts with Helm
- push them to helm repository
- sharing Helm Charts: helm search <keyword>
- Templating Engine: 方便创建新的服务，大体相同的配置，practical for CI/CD, same applications across different environment

Helm chart structure
Directory structure

value injection into template files
helm install --values=my-values.yaml <chartanme>
helm install --set version=2.0.0

release management

Helm version 2 comes in 2 parts:
Client(helm cli), Server(Tiller)
helm install <chartname>
helm upgrade <chartname>
helm rollback <chartname>

Why is a Pod abstraction useful?

Container vs Pods

- "pause" container in each pod
- also called "sandbox" container
- reserves and holds network namespace(netns)
- enables communication between containers


When are multiple containers necessary

# 这条命令表示你要进入 nginx Pod 中的 sidecar 容器。
kubectl exec -it nginx -c sidecar -- /bash/sh

kubernetes Volumes
the need for Volumes
Persistence does not rely on pod lifecycle

Storage Requirements
1. Storage doesn't depend on the pod lifecycle
2. Storage must be available on all nodes
3. Storage needs to survive even if cluster crashes

Persistent Volume
What type of storage do you need?
you need to create and manage them by yourself
YAML configuration example, 挂载配置

pod claims storage via PVC
PVC requests storage from SC
SC creates PV that meets the needs of the Claim

在 Kubernetes 中，持久存储（Persistent Storage）通过以下几个组件协同工作：Pod、PersistentVolumeClaim (PVC)、StorageClass (SC)、和 PersistentVolume (PV)。下面是这个过程的详细解释：

### 1. **Pod 声明存储：**

Pod 需要持久化存储时，会通过 `PersistentVolumeClaim`（PVC）来请求所需的存储资源。在 Pod 的 YAML 配置文件中，您可以将 PVC 作为一个卷（volume）挂载到 Pod 中。这个 PVC 会指定所需的存储大小、访问模式（如读写模式）等。

**示例 Pod 配置：**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

在这个示例中，`my-pod` 通过 `my-pvc` 这个 PVC 请求存储，并将其挂载到容器内的 `/usr/share/nginx/html` 目录。

### 2. **PVC 请求存储：**

`PersistentVolumeClaim` 是一个资源对象，它声明了一个 Pod 需要的存储。PVC 描述了存储的大小、访问模式以及可能的存储类别（通过 StorageClass）。Kubernetes 会根据 PVC 的要求找到一个合适的 `PersistentVolume`（PV）来绑定。

**示例 PVC 配置：**
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
  storageClassName: my-storage-class
```

在这个示例中，`my-pvc` 请求了 1GiB 的存储，并指定了 `my-storage-class` 作为存储类别。

### 3. **StorageClass 处理 PVC 请求：**

`StorageClass` 定义了如何动态地创建 `PersistentVolume`（PV）。它指定了存储的供应商（如 AWS EBS、GCE PD、NFS 等），以及特定存储类型的配置参数。PVC 请求存储时，Kubernetes 会根据 PVC 中指定的 `storageClassName`，使用对应的 StorageClass 来创建 PV。

**示例 StorageClass 配置：**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage-class
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
```

在这个示例中，`my-storage-class` 定义了一个 AWS EBS 的存储类型，使用 `gp2` 类型的卷，并且文件系统类型为 `ext4`。

### 4. **StorageClass 创建 PV 来满足 PVC 的需求：**

当一个 PVC 发出请求，且指定了一个 `StorageClass`，Kubernetes 会根据这个 `StorageClass` 的配置动态地创建一个 `PersistentVolume`（PV）。这个 PV 将符合 PVC 的需求，并且会绑定到 PVC。

**示例 PV（动态创建的）配置：**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-aws-ebs
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: my-storage-class
  awsElasticBlockStore:
    volumeID: vol-12345678
    fsType: ext4
```

在这个例子中，`pv-aws-ebs` 是一个动态创建的 PV，符合 PVC 的要求，并且使用了 AWS EBS 作为底层存储。

### 总结流程：
1. **Pod** 通过 **PVC** 请求存储。
2. **PVC** 向 **StorageClass** 请求分配存储。
3. **StorageClass** 根据配置动态创建满足 **PVC** 需求的 **PV**。
4. **PV** 被绑定到 **PVC**，并且 Pod 可以使用该持久存储。

这种方式使得 Kubernetes 中的存储管理更加灵活和自动化，简化了存储资源的分配和使用。

ConfigMap and Secret for mounting files
要在 Kubernetes 中使用 ConfigMap 和 Secret 来挂载文件，你可以创建一个 ConfigMap 和一个 Secret，并将它们作为卷挂载到 Pod 中。这种方法适用于需要将配置文件和敏感信息（如证书、密码）注入到容器中的情况。

### 1. **ConfigMap 示例**

假设你有一个名为 `mosquitto-config-file` 的配置文件，它的内容如下：

```ini
# mosquitto.conf
listener 1883
allow_anonymous true
```

你可以通过 ConfigMap 将这个配置文件注入到 Pod 中。

**创建 ConfigMap 的 YAML 文件：**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mosquitto-config
data:
  mosquitto.conf: |
    listener 1883
    allow_anonymous true
```

### 2. **Secret 示例**

假设你有一个名为 `secret.file` 的敏感文件，例如包含认证信息或证书：

```plaintext
# secret.file
username: myUser
password: myPassword
```

你可以通过 Secret 将这个文件注入到 Pod 中。

**创建 Secret 的 YAML 文件：**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mosquitto-secret
type: Opaque
data:
  secret.file: bXlVc2VyOm15UGFzc3dvcmQ=  # "username: myUser\npassword: myPassword" 的 base64 编码
```

这里需要注意的是，Secret 中的内容必须是 Base64 编码的。因此，在实际使用中，你需要先将 `secret.file` 的内容进行 Base64 编码。

### 3. **将 ConfigMap 和 Secret 挂载到 Pod 中**

下面是一个 Pod 的 YAML 示例，它将 ConfigMap 和 Secret 挂载为文件。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mosquitto-pod
spec:
  containers:
  - name: mosquitto
    image: eclipse-mosquitto
    volumeMounts:
    - name: config-volume
      mountPath: /mosquitto/config/mosquitto.conf
      subPath: mosquitto.conf
    - name: secret-volume
      mountPath: /mosquitto/config/secret.file
      subPath: secret.file
  volumes:
  - name: config-volume
    configMap:
      name: mosquitto-config
  - name: secret-volume
    secret:
      secretName: mosquitto-secret
```

### 解释：

- **ConfigMap 挂载**：
  - `mosquitto-config` ConfigMap 被挂载为 `config-volume`。
  - `volumeMounts` 中的 `subPath` 表示将 ConfigMap 中的 `mosquitto.conf` 文件挂载到容器内的 `/mosquitto/config/mosquitto.conf` 路径。

- **Secret 挂载**：
  - `mosquitto-secret` Secret 被挂载为 `secret-volume`。
  - `volumeMounts` 中的 `subPath` 表示将 Secret 中的 `secret.file` 文件挂载到容器内的 `/mosquitto/config/secret.file` 路径。

这样，`mosquitto.conf` 和 `secret.file` 会以文件的形式被挂载到容器内的指定路径，并且在容器启动时可以直接使用这些文件。

## pull image from private docker registry in kubernetes cluster
kubectl create secret generic my registry-key \
> --from-file=.dockerconfigjson=.docker/config.json \
> --type=kubernetes.io/dockerconfigjson
example:

Secret must be in the same namespace with Deployment

What is StatefulSet?
stateless applications deployed using Deployment
- identical and interchangeable
- created in random order
- one service that load balances to any pod
stateful applications using StatefulSet
- can't be created/deleted at same time
- can't be randomly addressed
- replica Pods are not identical(Pod Identity)

Pod Identity
- sticky identity for each pod
- created from same specification, but not interchangeable
- persistent identifier across any re-scheduling
  用Persistent Volume, lifecycle isn't tied to other component's lifecycle

pod endpoints
{pod name}.{governing service domain}
 2 characteristics
1. predictable pod name
2. fixed individual DNS name

How to create a StatefulSet?
StatefulSet configuration file

Setup Prometheus Monitoring on kubernetes using Helm and Prometheus Operator

Kubernetes Operator是什么？
怎么用它部署stateless和stateful application
HOW to deploy the app
how to create cluster of replicas?
how to recover?

stateful applications with operator
only 1 standard automated process

how does this work?
control loop mechanism,watch for changes
make use of CRD's(custom resource definitions,custom k8s components, extend k8s API)

k8s manage complete lifecycle of stateless apps
k8s can't automate the process natively for stateful apps

stateful apps 有它们各自对应的kubernetes operator，可以去网上找到对应的仓库

steps to monitor mongoDB metrics using Prometheus exporter

what is an exporter?
target:mongodb app
1. fetches metrics
2. converts to correct format
3. expose/metrics
4. 
what is a serviceMonitor?

what is a kubernetes service and when we need it?
each pod has its own IP address, pods are ephemeral, destroyed frequently

service has stable IP address, loadbalancing, lose coupling, within & outside cluster


different service types explained: 
clisterIP services: default type, IP address from Node's range,每个node有一个IP范围( kubectl get pod -o wide)
流量请求先经由Ingress,到sevices(internal service)再转发到相应的pod，
example: 
microservice app deployed, 
side car container: collects microservice logs

service communication: selector
how the service know which pods to forward the requests to? which port to forward it to?
pods are identified via selectors, it's key value pairs, label of pods, service的targetPort参数决定转发到哪个端口
必须匹配所有的label，register as endpoints



headless services, NodePort Services, LoadBalancer Services








using an operator manage of all Prometheus components
- find Prometeus operator
- deploy in k8s cluster

using Helm chart to deploy operator



Why StatefulSet is used?


How StatefulSet works and how it's different from Deployment







1 Docker 和虛拟机有哪些不同？
2简述Kubernetes 和 Docker 的关系？
3简述kube-proxy ipvs 和 iptables 的异同？
4 简述kube-proxy 切换为ipvs负载？
5简述微服务部署中的蓝绿发布？
6简述蓝绿发布的优缺点
6简述 Kubernetes 静态 Pod?
7简述 Kubernetes 数据持久化的方式有哪些？
8简述dockerfile中copy和add指令有什么区别？
