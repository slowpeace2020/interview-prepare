# Chapter 11. Stateless Service
>“Stateless Service”, describes the building blocks used for
managing identical application instances.
>The Stateless Service pattern describes how to create and operate
applications that are composed of identical ephemeral replicas. These
applications are best suited for dynamic cloud environments where they can
be rapidly scaled and made highly available.

## Problem
A stateless service does not maintain any state internally within the instance
across service interactions. In our context, it means a container is stateless if
it does not hold any information from requests in its internal storage
(memory or temporary filesystem) that is critical for serving future requests.
A stateless process has no stored knowledge of or reference to past requests,
so each request is made as if from scratch.
Instead, if the process needs to
store such information, it should store it in an external storage such as a
database, message queue, mounted filesystem, or some other data store that
can be accessed by other instances. 

Stateless services are made of identical, replaceable instances that often
offload state to external permanent storage systems and use load-balancers
for distributing incoming requests among themselves. In this chapter, we
will see specifically which Kubernetes abstractions can help operate such
stateless applications.

## Solution
The controller that is used for managing
stateless Pods is ReplicaSet, but that is a lower-level internal control
structure used by a Deployment. Deployment is the recommended user-
facing abstraction for creating and updating stateless applications, which
creates and manages the ReplicaSets behind the scene. A ReplicaSet should
be used when the update strategies provided by Deployment are not
suitable, or a custom mechanism is required, or no control over the update
process is needed at all.

### Instances
The primary purpose of a ReplicaSet is to ensure a specified number of
identical Pod replicas running at any given time. The main sections of a
ReplicaSet definition include the number of replicas indicating how many
Pods it should maintain, a selector that specifies how to identify the Pods it
manages, and a Pod template for creating new Pod replicas.

Regardless of whether you create a ReplicaSet directly or through a
Deployment, the end result will be that the desired number of identical Pod
replicas are created and maintained. The added benefit of using Deployment
is that we can control how the replicas are upgraded and rolled back.

 The ReplicaSet’s job is to restart the
containers if needed and scale out or in when the number of replicas is
increased or decreased, respectively. With this behavior, Deployment and
ReplicaSet can automate the lifecycle management of stateless applications.

### Networking
the ReplicaSet will
create a new Pod that will have a new name, hostname, and IP address. If
the application is stateless, as we’ve defined earlier in the chapter, new
requests should be handled from the newly created Pod the same way as by
any other Pod.
more
often, stateless services are contacted by other services over synchronous
request/response-driven protocols such as HTTP and gRPC. Since the Pod
IP address changes with every Pod restart, it is better to use a permanent IP
address based on a Kubernetes Service that service consumers can use. A
Kubernetes Service has a fixed IP address that doesn’t change during the
lifetime of the Service, and it ensures the client requests are always load-
balanced across instances and routed to the healthy and ready-to-accept-
requests Pods.

Once a Service is created, it is assigned a clusterIP that is accessible only
from within the Kubernetes cluster, and that IP remains unchanged as long
as the Service definition exists. This acts as a permanent entrypoint to all
matching Pods that are ephemeral and have changing IP addresses.

Notice that Deployment and the resulting ReplicaSet are only responsible
for maintaining the desired number of stateless Pods that match the label
selector. They are unaware of any Kubernetes Service that might be
directing traffic to the same set of Pods or a different combination of Pods.

### Storage
Most stateless services require
state, but they are stateless because they offload the state to some other
stateful system or data store, such as a filesystem.

In this section, we’ll look at the persistentVolumeClaim volume
type, which allows you to use manually or dynamically provisioned
persistent storage.

A PersistentVolume (PV) represents a storage resource abstraction in a
Kubernetes cluster that has a lifecycle independent of any Pod lifecycle that
is using it. A Pod cannot directly refer to a PV; however, a Pod uses
PersistentVolumeClaim (PVC) to request and bind to the PV, which points
to the actual durable storage. This indirect connection allows for a
separation of concerns and Pod lifecycle decoupling from PV. A cluster
administrator can configure storage provisioning and define PVs. The
developer creating Pod definitions can use PVC to use the storage. With this
indirection, even if the Pod is deleted, the ownership of the PV remains
attached to the PVC and continues to exist.

Once a PVC is defined, it can be referenced from a Pod template through
the persistentVolumeClaim field. One of the interesting fields of
PersistentVolumeClaim is accessModes. It controls how the storage is
mounted to the nodes and consumed by the Pods.

different
accessModes offered by Kubernetes:
ReadWriteOnce
This represents a volume that can be mounted to a single node at a time.
In this mode, one or multiple Pods running on the node could carry out
read and write operations.

ReadOnlyMany
The volume can be mounted to multiple nodes, but it allows read-only
operations to all Pods.

ReadWriteMany
In this mode, the volume can be mounted by many nodes and allows
both read and write operations.

ReadWriteOncePod
Notice that all of the access modes described so far offer per-node
granularity. Even ReadWriteOnce allows multiple Pods on the same
node to read from and write to the same volume simultaneously. Only
ReadWriteOncePod access mode guarantees that only a single Pod has
access to a volume.This is invaluable in scenarios where at most one
writer application is allowed to access data for data-consistency
guarantees. Use this mode with caution as it will turn your services into
a singleton and prevent scaling out.

In a ReplicaSet, all Pods are identical; they share the same PVC and refer to
the same PV. This is in contrast to StatefulSets covered in the next chapter,
where PVCs are created dynamically for each stateful Pod replica. This is
one of the major differences between how stateless and stateful workloads
are handled in Kubernetes.

## Discussion
Stateless services are composed
of identical, swappable, ephemeral, and replaceable instances. They are
ideal for handling short-lived requests and can scale up and down rapidly
without having any dependencies among the instances. 

At the lowest level, the Pod abstraction ensures that one or more containers
are observed with liveness checks and are always up and running. Building
on that, the ReplicaSet also ensures that the desired number of stateless
Pods are always running on the healthy nodes. Deployments automate the
upgrade and rollback mechanism of Pod replicas. When there is incoming
traffic, the Service abstraction discovers and distributes traffic to healthy
Pod instances with passing readiness probes. When a persistent file storage
is required, PVCs can request and mount storage.

You have to understand how liveness
checks and ReplicaSet control Pods’ lifecycles, and how they relate to
readiness probes and Service definitions controlling how the traffic is
directed to the Pods. You should also understand how PVCs and
accessMode control where the storage is mounted and how it is accessed.








