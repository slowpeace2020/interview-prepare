# Chapter 29. Elastic Scale
>The Elastic Scale pattern covers application scaling in multiple dimensions:
horizontal scaling by adapting the number of Pod replicas, vertical scaling
by adapting resource requirements for Pods, and scaling the cluster itself by
changing the number of cluster nodes. While all of these actions can be
performed manually, in this chapter we explore how Kubernetes can
perform scaling based on load automatically.

## Problem
 with the
seasonal nature of many workloads that often change over time, it is not an
easy task to figure out how the desired state should look. Accurately
identifying how many resources a container will require and how many
replicas a service will need at a given time to meet service-level agreements
takes time and effort. Luckily, Kubernetes makes it easy to alter the
resources of a container, the desired replicas for a service, or the number of
nodes in the cluster. Such changes can happen either manually, or given
specific rules, can be performed in a fully automated manner.
Kubernetes not only can preserve a fixed Pod and cluster setup but can also
monitor external load and capacity-related events, analyze the current state,
and scale itself for the desired performance. This kind of observation is a
way for Kubernetes to adapt and gain antifragile traits based on actual
usage metrics rather than anticipated factors. Let’s explore the different
ways we can achieve such behavior and how to combine the various scaling
methods for an even greater experience.

## Solution

### Manual Horizontal Scaling
Both manual scaling styles (imperative and declarative) expect a human to
observe or anticipate a change in the application load, make a decision on
how much to scale, and apply it to the cluster. They have the same effect,
but they are not suitable for dynamic workload patterns that change often
and require continuous adaptation. Next, let’s see how we can automate
scaling decisions themselves.



### Horizontal Pod Autoscaling
cloud native technologies
such as Kubernetes enable you to create applications that adapt to changing
loads. Autoscaling in Kubernetes allows us to define a varying application
capacity that is not fixed but instead ensures just enough capacity to handle
a different load. The most straightforward approach to achieving such
behavior is by using a HorizontalPodAutoscaler (HPA) to horizontally scale
the number of Pods. HPA is an intrinsic part of Kubernetes and does not
require any extra installation steps. One important limitation of the HPA is
that it can’t scale down to zero Pods so that no resources are consumed at
all if nobody is using the deployed workload. Luckily, Kubernetes add-ons
offer scale-to-zero and transform Kubernetes into a true serverless platform.

#### Kubernetes HorizontalPodAutoscaler
 Deployments create new
ReplicaSets during updates but without copying over any HPA definitions.
If you apply an HPA to a ReplicaSet managed by a Deployment, it is not
copied over to new ReplicaSets and will be lost. A better technique is to
apply the HPA to the higher-level Deployment abstraction, which preserves
and applies the HPA to the new ReplicaSet versions.

So choose a metric that is directly (preferably linearly)
correlated to the number of Pods.

introduce the two most prominent Kubernetes-
based add-ons for enabling scale-to-zero: Knative and KEDA. It is essential
to understand that Knative and KEDA are not alternative but
complementary solutions. Both projects cover different use cases and can
ideally be used together. As we will see, Knative specializes in stateless
HTTP applications and offers an autoscaling algorithm that goes beyond the
capabilities of the HPA. On the other hand, KEDA is a pull-based approach
that can be triggered by many different sources, like messages in a Kafka
topic or IBM MQ queue.

#### Knative
Historically, Knative used to be implemented as a custom metric adapter for
the HPA in Kubernetes. However, it later developed its own implementation
in order to have more flexibility in influencing the scaling algorithm and to
avoid the bottleneck of being able to register only a single custom metric
adapter in a Kubernetes cluster.


#### KEDA
Kubernetes Event-Driven Autoscaling (KEDA) is the other important
Kubernetes-based autoscaling platform that supports scale-to-zero but has a
different scope than Knative. While Knative supports autoscaling based on
HTTP traffic, KEDA is a pull-based approach that scales based on external
metrics from different systems. Knative and KEDA play very well together,
and there is only a little overlap,3 so nothing prevents you from using both
add-ons together.

Now that we have seen all the possibilities for scaling horizontally with
HPA, Knative, and KEDA, let’s look at a completely different kind of
scaling that does not alter the number of parallel-running replicas but lets
your application grow and shrink.


### Vertical Pod Autoscaling
Horizontal scaling is preferred over vertical scaling because it is less
disruptive, especially for stateless services. That is not the case for stateful
services, where vertical scaling may be preferred. Other scenarios where
vertical scaling is useful include tuning the resource needs of a service
based on actual load patterns. We’ve discussed why identifying the correct number of Pod replicas might be difficult and even impossible when the
load changes over time. Vertical scaling also has these kinds of challenges
in identifying the correct requests and limits for a container. The
Kubernetes Vertical Pod Autoscaler (VPA) aims to address these challenges
by automating the process of adjusting and allocating resources based on
real-world usage feedback.

Kubernetes is designed to manage immutable containers with immutable
Pod spec definitions, as seen in Figure 29-4. While this simplifies
horizontal scaling, it introduces challenges for vertical scaling, such as
requiring Pod deletion and recreation, which can impact scheduling and
cause service disruptions. This is true even when the Pod is scaling down
and wants to release already-allocated resources with no disruption.
Another concern is the coexistence of VPA and HPA because these
autoscalers are not currently aware of each other, which can lead to
unwanted behavior. 

### Cluster Autoscaling
One of the tenets of cloud computing is pay-as-you-go resource
consumption. We can consume cloud services when needed, and only as
much as needed. CA can interact with cloud providers where Kubernetes is
running and request additional nodes during peak times or shut down idle
nodes during other times, reducing infrastructure costs. While the HPA and
VPA perform Pod-level scaling and ensure service-capacity elasticity within
a cluster, the CA provides node scalability to ensure cluster-capacity
elasticity.

A CA primarily performs two operations: it add new nodes to a cluster or
removes nodes from a cluster. 

### Scaling Levels
To enable large-scale distributed system
management, automating repetitive activities is a must. The preferred
approach is to automate scaling and enable human operators to focus on
tasks that a Kubernetes Operator cannot automate yet.

Application tuning,Vertical Pod autoscaling,Horizontal Pod autoscaling

## Discussion
While a resilient and reliable system is good
enough for many applications today, Kubernetes goes a step further. A
small but properly configured Kubernetes system would not break under a
heavy load but instead would scale the Pods and nodes. So in the face of
these external stressors, the system would get bigger and stronger rather
than weaker and more brittle, giving Kubernetes antifragile capabilities.






