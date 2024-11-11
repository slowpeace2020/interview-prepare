# Chapter 27. Controller
>is essential to Kubernetes itself and shows
how custom controllers can extend the platform.

>A controller actively monitors and maintains a set of Kubernetes resources
in a desired state. The heart of Kubernetes itself consists of a fleet of
controllers that regularly watch and reconcile the current state of
applications with the declared target state.

## Problem
The main questions that arise here are how to extend Kubernetes without
changing and breaking it and how to use its capabilities for custom use
cases.

As opposed to an imperative approach,
a declarative approach does not tell Kubernetes how it should act but
instead describes how the target state should look.

The whole process is also known as state reconciliation, where a target state
(the number of desired replicas) differs from the current state (the actual
running instances), and it is the task of a controller to reconcile and reach
the desired target state again. When looked at from this angle, Kubernetes
essentially represents a distributed state manager. You give it the desired
state for a component instance, and it attempts to maintain that state should
anything change.

How can we now hook into this reconciliation process without modifying
Kubernetes code and create a controller customized for our specific needs?

## Solution
Kubernetes comes with a collection of built-in controllers that manage
standard Kubernetes resources like ReplicaSets, DaemonSets, StatefulSets,
Deployments, or Services. These controllers run as part of the controller
manager, which is deployed (as a standalone process or a Pod) on the
control plane node. These controllers are not aware of one another. They
run in an endless reconciliation loop, to monitor their resources for the
actual and desired state and to act accordingly to get the actual state closer
to the desired state.
However, in addition to these out-of-the-box controllers, the Kubernetes
event-driven architecture allows us to natively plug in other custom
controllers. Custom controllers can add extra functionality to the behavior
by reacting to state-changing events, the same way that internal controllers
do. A common characteristic of controllers is that they are reactive and
react to events in the system to perform their specific actions. At a high
level, this reconciliation process consists of the following main steps:
Observe,Analyze,Act

From an evolutionary and
complexity point of view, we can classify the active reconciliation
components into two groups:

Controllers
A simple reconciliation process that monitors and acts on standard
Kubernetes resources. More often, these controllers enhance platform
behavior and add new platform features.

Operators
A sophisticated reconciliation process that interacts with
CustomResourceDefinitions (CRDs), which are at the heart of the
Operator pattern. Typically, these operators encapsulate complex
application domain logic and manage the full application lifecycle.

To avoid having multiple controllers acting on the same resources
simultaneously, controllers use the Singleton Service pattern.

Most controllers are deployed just as Deployments but with one
replica, as Kubernetes uses optimistic locking at the resource level to
prevent concurrency issues when changing resource objects. In the end, a
controller is nothing more than an application that runs permanently in the
background.

The following are a few considerations to keep in mind when
choosing where to store controller data:

Labels
Labels as part of a resource’s metadata can be watched by any
controller. They are indexed in the backend database and can be
efficiently searched for in queries. We should use labels when a
selector-like functionality is required

Annotations
Annotations are an excellent alternative to labels. They have to be used
instead of labels if the values do not conform to the syntax restrictions
of label values. Annotations are not indexed, so we use annotations for
nonidentifying information not used as keys in controller queries.
Preferring annotations over labels for arbitrary metadata also has the
advantage that it does not negatively impact the internal Kubernetes
performance.

label和annotation的区别在哪里
Labels: 用于选择和过滤，是对象的主要标识工具。
annotations 不会在标准 Kubernetes 核心内直接引发操作变化，但在扩展工具或自定义控制器中，确实会通过解析这些 annotation 产生操作影响。

ConfigMaps
Sometimes controllers need additional information that does not fit well
into labels or annotations. In this case, ConfigMaps can be used to hold
the target state definition. 

Here are a few reasonably simple example controllers you can study as a
sample implementation of this pattern:
jenkins-x/exposecontroller：watches Service definitions, and if it detects an
annotation named expose in the metadata, the controller automatically
exposes an Ingress object for external access of the Service. It also
removes the Ingress object when someone removes the Service. This
project is now archived but still serves as a good example of
implementing a simple controller

stakater/Reloader
This is a controller that watches ConfigMap and Secret objects for
changes and performs rolling upgrades of their associated workloads,
which can be Deployment, DaemonSet, StatefulSet and other workload
resources. We can use this controller with applications that are not
capable of watching the ConfigMap and updating themselves with new
configurations dynamically. That is particularly true when a Pod
consumes this ConfigMap as environment variables or when your
application cannot quickly and reliably update itself on the fly without a
restart.


Flatcar Linux Update Operator
This is a controller that reboots a Kubernetes node running on Flatcar
Container Linux when it detects a particular annotation on the Node
resource object.

Our controller shell script now evaluates this ConfigMap. You can find the
source in its full glory in our Git repository. In short, the controller starts a
hanging GET HTTP request for opening an endless HTTP response stream
to observe the lifecycle events pushed by the API Server to us. These events
are in the form of plain JSON objects, which are then analyzed to detect
whether a changed ConfigMap carries our annotation. As events arrive, the
controller acts by deleting all Pods matching the selector provided as the
value of the annotation. Let’s have a closer look at how the controller
works.
The main part of this controller is the reconciliation loop, which listens on
ConfigMap lifecycle events。Watching events this way is not very robust, of course. The connection can
stop anytime, so there should be a way to restart the loop. Also, one could
miss events, so production-grade controllers should not only watch on
events but from time to time should also query the API Server for the entire
current state and use that as the new base.

Watching events this way is not very robust, of course. The connection can
stop anytime, so there should be a way to restart the loop. Also, one could
miss events, so production-grade controllers should not only watch on
events but from time to time should also query the API Server for the entire
current state and use that as the new base.


Extracting the ConfigMap’s k8spatterns.io/podDeleteSelector
annotation value and converting it to a Pod selector is performed with
jq. This is an excellent JSON command-line tool.


We use a Deployment to create a Pod for our controller with two containers:
One Kubernetes API ambassador container that exposes the
Kubernetes API on localhost on port 8001. 

The main container that executes the script contained in the just-
created ConfigMap.

we mount the config-watcher-controller-script from
the ConfigMap we created previously and directly use it as the startup
command for the primary container.

Also, we need a
ServiceAccount config-watcher-controller that is allowed to monitor
ConfigMaps. 


## Discussion
To sum up, a controller is an active reconciliation process that monitors
objects of interest for the world’s desired state and the world’s actual state.
Then, it sends instructions to try to change the world’s current state to be
more like the desired state. Kubernetes uses this mechanism with its
internal controllers, and you can also reuse the same mechanism with
custom controllers. We demonstrated what is involved in writing a custom
controller and how it functions and extends the Kubernetes platform. However, one issue with the
asynchronous nature of controllers is that they are often hard to debug
because the flow of events is not always straightforward. As a consequence,
you can’t easily set breakpoints in your controller to stop everything to
examine a specific situation.