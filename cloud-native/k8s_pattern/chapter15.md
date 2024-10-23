# Chapter15. Init Container
> introduces a lifecycle for initialization-
related tasks, decoupled from the main application responsibilities.

> The Init Container pattern enables separation of concerns by providing a
separate lifecycle for initialization-related tasks distinct from the main
application containers. In this chapter, we look closely at this fundamental
Kubernetes concept that is used in many other patterns when initialization
logic is required.

## Problem
if you have one or more containers in a Pod that represent
your main application, these containers may have prerequisites before
starting up. These may include special permissions setup on the filesystem,
database schema setup, or application seed data installation. Also, this
initializing logic may require tools and libraries that cannot be included in
the application image.For security reasons, the application image may not
have permissions to perform the initializing activities. Alternatively, you
may want to delay the startup of your application until an external
dependency is satisfied. For all these kinds of use cases, Kubernetes uses
init containers as implementation of this pattern, which allow separation of
initializing activities from the main application duties.

## Solution
Init containers in Kubernetes are part of the Pod definition, and they
separate all containers in a Pod into two groups: init containers and
application containers. If an init container fails, the
whole Pod is restarted (unless it is marked with RestartNever), causing all
init containers to run again. Thus, to prevent any side effects, making init
containers idempotent is a good practice.

On one hand, init containers have all of the same capabilities as application
containers: all of the containers are part of the same Pod, so they share
resource limits, volumes, and security settings and end up placed on the
same node. On the other hand, they have slightly different lifecycle, health-
checking, and resource-handling semantics.

first, init containers run a sequence,
then all application containers run in parallel。

example： illustrates a typical usage pattern of an init container sharing a
volume with the main container

### debug技巧
For debugging the outcome of init containers, it helps if the command of
the application container is replaced temporarily with a dummy sleep
command so that you have time to examine the situation. This trick is
particularly useful if your init container fails to start up and your
application fails to start because the configuration is missing or broken. The
following command within the Pod declaration gives you an hour to debug
the volumes mounted by entering the Pod with kubectl exec -it <pod>
sh:
   command:
   - /bin/sh
   - "-c"
   - "sleep 3600"

这个命令是在 application container 中使用的，而不是在 init container 中。

A similar effect can be achieved by using a sidecar, as described next in
Chapter 16, “Sidecar”, where the HTTP server container and the Git
container are running side by side as application containers. But with the
sidecar approach, there is no way of knowing which container will run first,
and sidecar is meant to be used when containers run side by side
continuously. We could also use a sidecar and init container together if both
a guaranteed initialization and a constant update of the data are required.

### MORE INITIALIZATION TECHNIQUES
As you have seen, an init container is a Pod-level construct that gets
activated after a Pod has been started. A few other related techniques
used to initialize Kubernetes resources are different from init containers
and are worth listing here for completeness

#### Admission controllers
#### Admission webhooks

Nowadays other projects such as Metacontroller and
Kyverno use admission webhooks or the Operator pattern to mutate
Kubernetes resources and intervene in the initialization process. These
techniques differ from init containers because they validate and mutate
resources at creation time.

In contrast, the Init Container pattern discussed in this chapter is
something that activates and performs its responsibilities during startup
of the Pod. You could use admission webhooks, for example, to inject
an init container into any Pod that doesn’t have one already. 

For
example, Istio, which is a popular service mesh project, uses a
combination of techniques discussed in this chapter to inject its proxies
into application Pods. Istio uses Kubernetes mutating admission
webhooks for automatic sidecar and init container injection into the Pod
definition at Pod definition creation time. When such a Pod is starting
up, Istio’s init container configures the Pod environment to redirect
inbound and outbound traffic from the application to the Envoy proxy
sidecar. The init container runs before any other container and
configures iptable rules to insert the Envoy proxy in the request path of
the application before any traffic reaches the application. This
separation of containers is good for lifecycle management and also
because the init container in this case requires elevated permissions to
configure traffic redirection, which can pose a security threat. This is an
example of how many initialization activities can be performed before
an application container starts up.
In the end, the most significant difference is that init containers can be
used by developers deploying on Kubernetes, whereas admission
webhooks help administrators and various frameworks control and alter
the container initialization process.

## Discussion
why separate containers in a Pod into two groups? Why not just use an
application container with a bit of scripting in a Pod for initialization if
required? The answer is that these two groups of containers have different
lifecycles, purposes, and even authors in some cases.
Having init containers run before application containers, and more
importantly, having init containers run in stages that progress only when the
current init container completes successfully, means you can be sure at
every step of the initialization that the previous step has completed
successfully, and you can progress to the next stage. Application containers,
in contrast, run in parallel and do not provide similar guarantees as init
containers. With this distinction in hand, we can create containers focused
on initialization or application-focused tasks, and reuse them in different
contexts by organizing them in Pods with predictable guarantees.

