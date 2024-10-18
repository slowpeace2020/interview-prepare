# Chapter 12. “Stateful Service”

>is all about how to create and manage
distributed stateful applications with Kubernetes.

>Distributed stateful applications require features such as persistent identity,
networking, storage, and ordinality. The Stateful Service pattern describes
the StatefulSet primitive that provides these building blocks with strong
guarantees ideal for the management of stateful applications.

## Problem
It is a significant boost to have a platform taking care of the placement,
resiliency, and scaling of stateless applications, but there is still a large part
of the workload to consider: stateful applications in which every instance is
unique and has long-lived characteristics.

In the real world, behind every highly scalable stateless service is a stateful
service, typically in the shape of a data store. While that is mostly true for a single-instance stateful application, it is not
entirely true, as a ReplicaSet does not guarantee At-Most-One semantics,
and the number of replicas can vary temporarily. Such a situation can be
disastrous and lead to data loss for distributed stateful applications.

### Storage
How do we define the
storage requirements in such a case? Typically, a distributed stateful
application such as those mentioned previously would require dedicated,
persistent storage for every instance. ReplicaSet with replicas=3 and a
PVC definition would result in all three Pods attached to the same PV.A workaround is for the application instances to share storage and have an
in-app mechanism to split the storage into subfolders and use it without
conflicts. While doable, this approach creates a single point of failure with
the single storage. Also, it is error-prone as the number of Pods changes
during scaling, and it may cause severe challenges around preventing data
corruption or loss during scaling。

Another workaround is to have a separate ReplicaSet (with replicas=1)
for every instance of the distributed stateful application. In this scenario,
every ReplicaSet would get its PVC and dedicated storage. The downside
of this approach is that it is intensive in manual labor: scaling up requires
creating a new set of ReplicaSet, PVC, or Service definitions. This
approach lacks a single abstraction for managing all instances of the stateful
application as one.

### Networking
Similar to the storage requirements, a distributed stateful application
requires a stable network identity. In addition to storing application-specific
data into the storage space, stateful applications also store configuration
details such as hostname and connection details of their peers. That means
every instance should be reachable in a predictable address that should not
change dynamically, as is the case with Pod IP addresses in a ReplicaSet.
Here we could address this requirement again through a workaround: create
a Service per ReplicaSet and have replicas=1. However, managing such a
setup is manual work, and the application itself cannot rely on a stable
hostname because it changes after every restart and is also not aware of the
Service name it is accessed from

### Identity
As you can see from the preceding requirements, clustered stateful
applications depend heavily on every instance having a hold of its long-
lived storage and network identity. That is because in a stateful application,
every instance is unique and knows its own identity, and the main
ingredients of that identity are the long-lived storage and the networking
coordinates. To this list, we could also add the identity/name of the instance
(some stateful applications require unique persistent names), which in
Kubernetes would be the Pod name. A Pod created with ReplicaSet would
have a random name and would not preserve that identity across a restart.

### Ordinality
In addition to a unique and long-lived identity, the instances of clustered
stateful applications have a fixed position in the collection of instances.
This ordering typically impacts the sequence in which the instances are
scaled up and down. However, it can also be used for data distribution or
access and in-cluster behavior positioning such as locks, singletons, or
leaders.

### Other Requirements
Stable and long-lived storage, networking, identity, and ordinality are
among the collective needs of clustered stateful applications. Managing
stateful applications also carries many other specific requirements that vary
case by case. For example, some applications have the notion of a quorum
and require a minimum number of instances to always be available; some
are sensitive to ordinality, and some are fine with parallel Deployments; and
some tolerate duplicate instances, and some don’t. Planning for all these
one-off cases and providing generic mechanisms is an impossible task, and
that’s why Kubernetes also allows you to create CustomResourceDefinitions (CRDs) and Operators for managing
applications with bespoke requirements.

## Solution
In many
ways, StatefulSet is for managing pets, and ReplicaSet is for managing
cattle. Pets versus cattle is a famous (but also a controversial) analogy in the
DevOps world: identical and replaceable servers are referred to as cattle,
and nonfungible unique servers that require individual care are referred to
as pets. 
Similarly, StatefulSet (initially inspired by the analogy and named
PetSet) is designed for managing nonfungible Pods, as opposed to
ReplicaSet, which is for managing identical replaceable Pods.

example： our random-generator service as a
StatefulSet

### Storage
The
way to request and associate persistent storage with a Pod in Kubernetes is
through PVs and PVCs. To create PVCs the same way it creates Pods,
StatefulSet uses a volumeClaimTemplates element. This extra property is
one of the main differences between a StatefulSet and a ReplicaSet, which
has a persistentVolumeClaim element.

Rather than referring to a predefined PVC, StatefulSets create PVCs by
using volumeClaimTemplates on the fly during Pod creation. This
mechanism allows every Pod to get its own dedicated PVC during initial
creation as well as during scaling up by changing the replicas count of the
StatefulSets.

As you probably realize, we said PVCs are created and associated with the
Pods, but we didn’t say anything about PVs. That is because StatefulSets do
not manage PVs in any way. The storage for the Pods must be provisioned
in advance by an admin or provisioned on demand by a PV provisioner
based on the requested storage class and ready for consumption by the
stateful Pods.

Note the asymmetric behavior here: scaling up a StatefulSet (increasing the
replicas count) creates new Pods and associated PVCs. Scaling down
deletes the Pods, but it does not delete any PVCs (or PVs), which means the
PVs cannot be recycled or deleted, and Kubernetes cannot free the storage.
This behavior is by design and driven by the presumption that the storage of
stateful applications is critical and that an accidental scale-down should not
cause data loss.

### Networking
Each Pod created by a StatefulSet has a stable identity generated by the
StatefulSet’s name and an ordinal index (starting from 0).

we define a headless Service. In a headless Service,
clusterIP is set to None, which means we don’t want a kube-proxy to
handle the Service, and we don’t want a cluster IP allocation or load
balancing. Then why do we need a Service?

Stateless Pods created through a ReplicaSet are assumed to be identical, and
it doesn’t matter on which one a request lands (hence the load balancing
with a regular Service). But stateful Pods differ from one another, and we
may need to reach a specific Pod by its coordinates.

A headless Service with selectors (notice .selector.app == random-
generator) enables exactly this. Such a Service creates endpoint records in
the API Server and creates DNS entries to return A records (addresses) that
point directly to the Pods backing the Service. Long story short, each Pod
gets a DNS entry where clients can directly reach out to it in a predictable
way. For example, if our random-generator Service belongs to the
default namespace, we can reach our rg-0 Pod through its fully qualified
domain name: rg-0.random-generator.default.svc.cluster.local,
where the Pod’s name is prepended to the Service name. This mapping
allows other members of the clustered application or other clients to reach
specific Pods if they wish to.

We can also perform DNS lookup for Service (SRV) records (e.g., through
dig SRV random-generator.default.svc.cluster.local) and
discover all running Pods registered with the StatefulSet’s governing
Service. This mechanism allows dynamic cluster member discovery if any
client application needs to do so. The association between the headless
Service and the StatefulSet is not only based on the selectors, but the
StatefulSet should also link back to the Service by its name as
serviceName。

Having dedicated storage defined through volumeClaimTemplates is not
mandatory, but linking to a Service through serviceName field is. The
governing Service must exist before the StatefulSet is created and is
responsible for the network identity of the set. You can always create other
types of Services that also load balance across your stateful Pods if that is
what you want.

### Identity
Identity is the meta building block all other StatefulSet guarantees are built
upon. A predictable Pod name and identity is generated based on
StatefulSet’s name. 
We then use that identity to name PVCs, reach out to
specific Pods through headless Services, and more. You can predict the
identity of every Pod before creating it and use that knowledge in the
application itself if needed.

### Ordinality
In addition to their uniqueness,
instances may also be related to one another based on their instantiation
order/position, and this is where the ordinality requirement comes in.

From a StatefulSet point of view, the only place where ordinality comes
into play is during scaling. Pods have names that have an ordinal suffix
(starting from 0), and that Pod creation order also defines the order in which
Pods are scaled up and down (in reverse order, from n – 1 to 0).

To allow proper data synchronization during scale-up and -down,
StatefulSet by default performs sequential startup and shutdown. That
means Pods start from the first one (with index 0), and only when that Pod
has successfully started is the next one scheduled (with index 1), and the
sequence continues. During scaling down, the order reverses—first shutting
down the Pod with the highest index, and only when it has shut down
successfully is the Pod with the next lower index stopped. This sequence
continues until the Pod with index 0 is terminated.

### Other Features
StatefulSets have other aspects that are customizable to suit the needs of
stateful applications. Each stateful application is unique and requires careful
consideration while trying to fit it into the StatefulSet model. Let’s see a
few more Kubernetes features that may turn out to be useful while taming
stateful applications：

#### Partitioned updates
StatefulSets allow
phased rollout (such as a canary release), which guarantees a certain
number of instances to remain intact while applying updates to the rest
of the instances.By using the default rolling update strategy, you can partition instances
by specifying a .spec.updateStrategy.rollingUpdate.partition
number. The parameter (with a default value of 0) indicates the ordinal
at which the StatefulSet should be partitioned for updates.If the
parameter is specified, all Pods with an ordinal index greater than or
equal to the partition are updated, while all Pods with an ordinal less
than that are not updated. That is true even if the Pods are deleted;
Kubernetes recreates them at the previous version. This feature can
enable partial updates to clustered stateful applications (ensuring the
quorum is preserved, for example) and then roll out the changes to the
rest of the cluster by setting the partition back to 0.

#### Parallel deployments
When we set .spec.podManagementPolicy to Parallel, the
StatefulSet launches or terminates all Pods in parallel and does not wait
for Pods to run and become ready or completely terminated before
moving to the next one. If sequential processing is not a requirement for
your stateful application, this option can speed up operational
procedures。

#### At-Most-One Guarantee
Uniqueness is among the fundamental attributes of stateful application
instances, and Kubernetes guarantees that uniqueness by making sure
no two Pods of a StatefulSet have the same identity or are bound to the
same PV. In contrast, ReplicaSet offers the At-Least-X-Guarantee for its
instances. For example, a ReplicaSet with two replicas tries to keep at
least two instances up and running at all times. 

A StatefulSet controller, on the other hand, makes every possible check
to ensure there are no duplicate Pods—hence the At-Most-One
Guarantee. It does not start a Pod again unless the old instance is
confirmed to be shut down completely.

## Discussion
We
discovered that handling a single-instance stateful application is relatively
easy, but handling distributed state is a multidimensional challenge. While
we typically associate the notion of “state” with “storage,” here we have
seen multiple facets of state and how it requires different guarantees from
different stateful applications. In this space, StatefulSets is an excellent
primitive for implementing distributed stateful applications generically. It
addresses the need for persistent storage, networking (through Services),
identity, ordinality, and a few other aspects. It provides a good set of
building blocks for managing stateful applications in an automated fashion,
making them first-class citizens in the cloud native world.
StatefulSets are a good start and a step forward, but the world of stateful
applications is unique and complex. In addition to the stateful applications
designed for a cloud native world that can fit into a StatefulSet, a ton of
legacy stateful applications exist that have not been designed for cloud
native platforms and have even more needs. Luckily Kubernetes has an
answer for that too. The Kubernetes community has realized that rather than
modeling different workloads through Kubernetes resources and
implementing their behavior through generic controllers, it should allow
users to implement their custom controllers and even go one step further
and allow modeling application resources through custom resource
definitions and behavior through operators.






