# Declarative Deployment

>describes the different
application deployment strategies that can be expressed in a
declarative way. This abstraction encapsulates the upgrade and
rollback processes of a group of containers and makes its execution a
repeatable and automated activity.

## Problem
We can provision isolated environments as namespaces in a self-service
manner and place the applications in these environments with minimal
human intervention through the scheduler. But with a growing number of
microservices, continually updating and replacing them with newer versions
becomes an increasing burden too.


## Solution
At the very core of a Deployment is the ability to start and stop a set of Pods predictably. For this
to work as expected, the containers themselves usually listen and honor lifecycle events


DEPLOYMENT UPDATES WITH KUBECTL ROLLOUT
rollout相关的命令

### Rolling Deployment

The declarative way of updating applications in Kubernetes is through the
concept of Deployment. Behind the scenes, the Deployment creates a
ReplicaSet that supports set-based label selectors. Also, the Deployment
abstraction allows you to shape the update process behavior with strategies
such as RollingUpdate (default) and Recreate.

RollingUpdate的各项参数配置含义

RollingUpdate strategy behavior ensures there is no downtime during the
update process. Behind the scenes, the Deployment implementation
performs similar moves by creating new ReplicaSets and replacing old
containers with new ones. One enhancement here is that with Deployment,
it is possible to control the rate of a new container rollout. The Deployment
object allows you to control the range of available and excess Pods through
maxSurge and maxUnavailable fields.

These two fields can be either absolute numbers of Pods or relative
percentages that are applied to the configured number of replicas for the
Deployment and are rounded up (maxSurge) or down (maxUnavailable) to
the next integer value. By default, maxSurge and maxUnavailable are both
set to 25%.

Another important parameter that influences the rollout behavior is
minReadySeconds. This field specifies the duration in seconds that the
readiness probes of a Pod need to be successful until the Pod itself is
considered to be available in a rollout.

[滚动升级的小demo，example](https://github.com/k8spatterns/examples/tree/main/foundational/DeclarativeDeployment)


Deployment的好处：
Deployment is a Kubernetes resource object whose status is entirely
managed by Kubernetes internally. The whole update process is
performed on the server side without client interaction.
The declarative nature of Deployment specifies how the deployed state
should look rather than the steps necessary to get there.
The Deployment definition is an executable object and more than just
documentation. It can be tried and tested on multiple environments
before reaching production.
The update process is also wholly recorded and versioned with options
to pause, continue, and roll back to previous versions

### Fixed Deployment
A RollingUpdate strategy is useful for ensuring zero downtime during the
update process. However, the side effect of this approach is that during the
update process, two versions of the container are running at the same time.
That may cause issues for the service consumers, especially when the
update process has introduced backward-incompatible changes in the
service APIs and the client is not capable of dealing with them. 在这种情况下，你就可以用Recreate的策略。

The Recreate strategy has the effect of setting maxUnavailable to the
number of declared replicas. This means it first kills all containers from the
current version and then starts all new containers simultaneously when the
old containers are evicted. The result of this sequence is that downtime
occurs while all containers with old versions are stopped, and no new
containers are ready to handle incoming requests. On the positive side, two
different versions of the containers won’t be running at the same time, so
service consumers can connect only one version at a time.


### Blue-Green Release
minimizing downtime and
reducing risk.

We can use the Deployment
primitive as a building block, together with other Kubernetes primitives, to
implement this more advanced release strategy。

什么是蓝绿部署
A Blue-Green deployment needs to be done manually if no extensions like
a service mesh or Knative are used, though. Technically, it works by
creating a second Deployment, with the latest version of the containers
(let’s call it green) not serving any requests yet. At this stage, the old Pod
replicas from the original Deployment (called blue) are still running and
serving live requests.Once we are confident that the new version of the Pods is healthy and ready
to handle live requests, we switch the traffic from old Pod replicas to the
new replicas. 

优点是一个时间的服务版本是一样的，服务的消费者不需要同时处理几个版本，降低复杂性。缺点是因为两个版本都在运行，需要双倍的application capacity. Also, significant complications
can occur with long-running processes and database state drifts during the
transitions.

### Canary Release
Canary release is a way to softly deploy a new version of an application
into production by replacing only a small subset of old instances with new
ones. This technique reduces the risk of introducing a new version into
production by letting only some of the consumers reach the updated
version. When we’re happy with the new version of our service and how it
performed with a small sample of users, we can replace all the old instances
with the new version in an additional step after this canary release.

In Kubernetes, this technique can be implemented by creating a new
Deployment with a small replica count that can be used as the canary
instance. At this stage, the Service should direct some of the consumers to
the updated Pod instances. After the canary release and once we are
confident that everything with the new ReplicaSet works as expected, we
scale the new ReplicaSet up, and the old ReplicaSet down to zero. In a way,
we’re performing a controlled and user-tested incremental rollout.

## Discussion
The out-of-the-box deployment strategies
(rolling and recreate) control the replacement of old containers by new
ones, and the advanced release strategies (Blue-Green and canary) control
how the new version becomes available to service consumers. The latter
two release strategies are based on a human decision for the transition
trigger and as a consequence are not fully automated by Kubernetes but
require human interaction.


Pre and Post hooks would allow the execution of custom commands before and after Kubernetes has executed a deployment strategy. 

Deployment does not support
more sophisticated strategies like canary or Blue-Green deployments
directly. There are higher-level abstractions that enhance Kubernetes by
introducing new resource types, enabling the declaration of more
flexible deployment strategies. Those extensions all leverage the
Operator pattern

the most prominent platforms that support higher-level
Deployments: Flagger,Argo Rollouts,Knative


it is essential for
Kubernetes to know when your application Pods are up and running to
perform the required sequence of steps to reach the defined target
deployment state. 