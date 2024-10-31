# Chapter 24. Network Segmentation
>applies network controls to
limit the traffic a Pod is allowed to participate in.

## Problem
Namespaces are a crucial part of Kubernetes, allowing you to group your
workloads together. However, they only provide a grouping concept,
imposing isolation constraints on the containers associated with specific
namespaces. In Kubernetes, every Pod can talk to every other Pod,
regardless of their namespace. This default behavior has security
implications, particularly when multiple independent applications operated
by different teams run in the same cluster.Restricting network access to and from Pods is essential for enhancing the
security of your application because not everyone may be allowed to access
your application via an ingress. Outgoing egress network traffic for Pods
should also be limited to what is necessary to minimize the blast radius of a
security breach.Network segmentation plays a vital role in multitenancy setups where
multiple parties share the same cluster. what does defining and establishing a network segmentation look like in
a Kubernetes world?

## Solution
The essence of this Network Segmentation pattern is how we, as developers,
can define the network segmentation for our applications by creating
“application firewalls.”

There are two ways to implement this feature that are complementary and
can be applied together. The first is through the use of core Kubernetes
features that operate on the L3/L4 networking layers.1 By defining
resources of the type NetworkPolicy, developers can create ingress and
egress firewall rules for workload Pods.
The other method involves the use of a service mesh and targets the L7
protocol layer, specifically HTTP-based communication. This allows for
filtering based on HTTP verbs and other L7 protocol parameters.

### Network Policies
NetworkPolicy is a Kubernetes resource type that allows users to define
rules for inbound and outbound network connections for Pods. These rules
act like a custom firewall and determine which Pods can be accessed and
which destinations they can connect to. The user-defined rules are picked up
by the Container Network Interface (CNI) add-on used by Kubernetes for
its internal networking.

The NetworkPolicy definition consists of a selector for Pods and lists of
inbound (ingress) or outbound (egress) rules.
The Pod selector is used to match the Pods to which the NetworkPolicy
should be applied. This selection is done by using labels, which are
metadata attached to Pods.

NetworkPolicy objects are namespace-scoped and match only Pods from
within the NetworkPolicy’s namespace. Unfortunately, there is no way to
define cluster-wide defaults for all namespaces. However, some CNI
plugins like Calico support customer extensions for defining cluster-wide
behavior.

There are two common ways to consistently label workloads:
a. Using workload-unique labels, you can directly model the dependency
graph between application components such as other microservices or
a database.

b. In a more loosely coupled approach, you can define specific role or
permissions labels that need to be attached to every workload that
plays a certain role.

#### ingress
we have seen how to individually configure the
allowed incoming connections for a selected set of Pods. This setup works
fine as long as you don’t forget to configure one Pod, since the default
mode, when NetworkPolicy is not configured in the namespace, does not
restrict incoming and outgoing traffic (allow-all). Also, for Pods that we
might create in the future, it is problematic that it might be necessary to
remember to add the respective NetworkPolicy.

Therefore, it is highly recommended to start with a deny-all policy。


Besides selecting Pods, you have additional options to configure the ingress
rules. We already saw the from field for an ingress rule that can contain a
podSelector for selecting all Pods that pass this rule. In addition, a
namespaceSelector can be given to choose the namespaces in which the
podSelector should be applied to identify the Pods that can send traffic.

#### Egress
Egress rules are configured precisely
with the same options as ingress rules. And as with ingress rules, starting
with a very restrictive policy is recommended. However, denying all
outgoing traffic is not practical. Every Pod needs interaction with Pods from
the system namespace for DNS lookups. Also, if we use ingress rules to
restrict incoming traffic, we would have to add mirrored egress rules for the
source Pods.

First, it is
essential to always allow access to the DNS server in the kube-system
namespace. This configuration is best done by allowing access to port 53
for UDP and TCP to all ports in the system namespace.

For operators and controllers, the Kubernetes API server needs to be
accessible. Unfortunately, no unique label would select the API server in
the kube-system namespace, so the filtering should happen on the API
server’s IP address. The IP address can best be fetched from the
kubernetes endpoints in the default namespace with kubectl get
endpoints -n default kubernetes.

#### Tooling
Setting up the network topology with NetworkPolicies gets complex
quickly since it involves creating many NetworkPolicy resources. It is best
to start with some simple use cases that you can adapt to your specific
needs. Kubernetes Network Policy Recipes is a good starting point.

policy advisor tools can be beneficial.
They work by recording the network activity when playing through typical
use cases. A comprehensive integration test suite with good test coverage
pays off to catch all corner cases involving network connections. As of
2023, several tools can help you audit network traffic to create network
policies.

Inspektor Gadget is a great tool suite for debugging and inspecting
Kubernetes resources.

It is entirely based on eBPF programs that enable
kernel-level observability and provides a bridge from kernel features to
high-level Kubernetes resources. One of Inspektor Gadget’s features is to
monitor network activity and record all UDP and TCP traffic for generating
Kubernetes network policies. This technique works well but depends on the
quality and depth of covered use cases。eBPF is a Linux technology that can run sandboxed programs in kernel
space.2 This technique extends the kernel’s capabilities safely and
allows for much faster innovation on top of this interface.

Now that you have seen how we can model the network boundaries for our
application on the TCP/UDP and IP levels, let’s move up some levels in the
OSI stack.

### Authentication Policies
it is sometimes beneficial to base the
network restrictions on filtering on higher-level protocol parameters. This
advanced network control requires knowledge of higher-level protocols like
HTTP and the ability to inspect incoming and outgoing traffic. Kubernetes
does not support this out of the box. Luckily, a whole family of add-ons
extends Kubernetes to provide this functionality: service meshes.

Some operational requirements like security, observability, or reliability
affect all your applications. A service mesh takes care of these aspects
in a generic way so that applications can focus on their business logic.Authorization in Istio is
configured with the AuthorizationPolicy resource. While
AuthorizationPolicy is only one component in Istio’s security model, it can
be used alone and allows for partitioning the network space based on HTTP.

The schema of AuthorizationPolicy is very similar to NetworkPolicy but is
more flexible and includes HTTP-specific filters. NetworkPolicy and
AuthorizationPolicy should be used together. This can lead to a tricky
debugging setup when two configurations must be checked and verified in
parallel. Traffic will pass through to a Pod only if the two user-defined
firewalls spanned by NetworkPolicy and AuthorizationPolicy definition will
allow it.

An AuthorizationPolicy is a namespaced resource and contains a set of
rules that control whether or not traffic is allowed or denied to a particular
set of Pods in a Kubernetes cluster. 

## Discussion
In the early days of computing, network topologies were defined by
physical wiring and devices like switches. This approach is secure but not
very flexible. With the advent of virtualization, these devices were replaced
by software-backed constructs to provide network security. Software-
defined networking (SDN) is a type of computer networking architecture
that allows network administrators to manage network services through
abstraction of lower-level functionality. This abstraction is typically
achieved by separating the control plane, which makes decisions about how
data should be transmitted, from the data plane, which actually sends the
data. Even with the use of SDN, administrators are still needed to set up and
rearrange networking boundaries to effectively manage the network.

Kubernetes has the ability to overlay its flat cluster-internal network with
network segments defined by users through the Kubernetes API. This is the
next step in the evolution of network user interfaces. It shifts the
responsibility to developers who understand the security requirements of
their applications. This shift-left approach is beneficial in a world of
microservices with many distributed dependencies and a complex network
of connections. NetworkPolicies for L3/L4 network segmentation and
AuthenticationPolicies for more granular control of network boundaries are
essential for implementing this Network Segmentation pattern.

With the advent of eBPF-based platforms on top of Kubernetes, there is
additional support for finding suitable network models. Cilium is an
example of a platform that combines L3/L4 and L7 firewalling into a single
API, making it easier to implement the pattern described in this chapter in
future versions of Kubernetes.

