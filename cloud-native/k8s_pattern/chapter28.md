Chapter 28. Operator
>combines a controller with custom domain-
specific resources to encapsulate operational knowledge in an
automated form.

>An operator is a controller that uses a CRD to encapsulate operational
knowledge for a specific application in an algorithmic and automated form.
The Operator pattern allows us to extend the Controller pattern from the
preceding chapter for more flexibility and greater expressiveness.

## Problem
for extended use cases,
plain custom controllers are not powerful enough, as they are limited to
watching and managing Kubernetes intrinsic resources only. Moreover,
sometimes we want to add new concepts to the Kubernetes platform, which
requires additional domain objects. For example, let’s say we chose
Prometheus as our monitoring solution and want to add it as a monitoring
facility to Kubernetes in a well-defined way. Wouldn’t it be wonderful to
have a Prometheus resource describing our monitoring setup and all the
deployment details, similar to how we define other Kubernetes resources?
Moreover, could we have resources relating to services we have to monitor?

These situations are precisely the kind of use cases where
CustomResourceDefinition (CRD) resources are very helpful. They allow
extensions of the Kubernetes API, by adding custom resources to your
Kubernetes cluster and using them as if they were native resources. Custom
resources, together with a controller acting on these resources, form the
Operator pattern.

An operator is a Kubernetes controller that understands two domains:
Kubernetes and something else. By combining knowledge of both areas,
it can automate tasks that usually require a human operator that
understands both domains.

## Solution

### Custom Resource Definitions
With a CRD, we can extend Kubernetes to manage our domain concepts on
the Kubernetes platform. Custom resources are managed like any other
resource, through the Kubernetes API, and are eventually stored in the
backend store etcd.

An OpenAPI V3 schema can also be specified to allow Kubernetes to
validate a custom resource. For simple use cases, this schema can be
omitted, but for production-grade CRDs, the schema should be provided so
that configuration errors can be detected early.

Additionally, Kubernetes allows us to specify two possible subresources for
our CRD via the spec field subresources:

scale
With this property, a CRD can specify how it manages its replica count.
This field can be used to declare the JSON path, where the number of
desired replicas of this custom resource is specified: the path to the
property that holds the actual number of running replicas and an
optional path to a label selector that can be used to find copies of
custom resource instances. 

status
When this property is set, a new API call becomes available that allows
you to update only the status field of a resource. This API call can be
secured individually and allows the operator to reflect the actual status
of the resource, which might differ from the declared state in the spec
field. When a custom resource is updated as a whole, any sent status
section is ignored, as is the case with standard Kubernetes resources.

### Controller and Operator Classification
Based on the
operator’s action, broadly, the classifications are as follows:


#### Installation CRDs
Meant for installing and operating applications on the Kubernetes
platform. Typical examples are the Prometheus CRDs, which we can
use for installing and managing Prometheus itself.

#### Application CRDs
In contrast, these are used to represent an application-specific domain
concept. This kind of CRD allows applications deep integration with
Kubernetes, which involves combining Kubernetes with an application-
specific domain behavior. For example, the ServiceMonitor CRD is
used by the Prometheus operator to register specific Kubernetes
Services to be scraped by a Prometheus server. The Prometheus
operator takes care of adapting the Prometheus server configuration
accordingly.

The boundary between these two categories of
CRDs is blurry.

An advantage of using a plain ConfigMap is that you don’t need to have the
cluster-admin rights you need when registering a CRD. In certain cluster
setups, it is just not possible for you to register such a CRD.

you can still use the concept of Observe-Analyze-Act when you
replace a CRD with a plain ConfigMap that you use as your domain-
specific configuration. The drawback is that you don’t get essential tool
support like kubectl get for CRDs; you have no validation on the API
Server level and no support for API versioning. Also, you don’t have much
influence on how you model the status field of a ConfigMap, whereas for
a CRD, you are free to define your status model as you wish.

Another advantage of CRDs is that you have a fine-grained permission
model based on the kind of CRD, which you can tune individually.

This kind of RBAC security is
not possible when all your domain configuration is encapsulated in
ConfigMaps, as all ConfigMaps in a namespace share the same permission
setup.

For the CRD case, we don’t have the type information out of the
box, and we can either use a schemaless approach for managing CRD
resources or define the custom types on our own, possibly based on an
OpenAPI schema contained in the CRD definition. Support for typed CRDs
varies by client library and framework used.

For operators, there is even a more advanced Kubernetes extension hook
option. When Kubernetes-managed CRDs are not sufficient to represent a
problem domain, you can extend the Kubernetes API with its own
aggregation layer. We can add a custom-implemented APIService resource
as a new URL path to the Kubernetes API.

### Operator Development and Deployment

#### Kubebuilder
is a framework and
library for creating Kubernetes APIs via CustomResourceDefinitions.

#### Operator framework
The Operator Framework provides extensive support for developing
operators.

#### Metacontroller
Metacontroller is very different from the other two operator building
frameworks as it extends Kubernetes with APIs that encapsulate the
common parts of writing custom controllers. It acts similarly to Kubernetes
Controller Manager by running multiple controllers that are not hardcoded
but are defined dynamically through Metacontroller-specific CRDs. In other
words, it’s a delegating controller that calls out to the service providing the
actual controller logic.

This delegation allows us to write functions in any language that can
understand HTTP and JSON and that do not have any dependency on the
Kubernetes API or its client libraries. The functions can be hosted on
Kubernetes, or externally on a Functions-as-a-Service provider, or
somewhere else.

if your use case involves
extending and customizing Kubernetes with simple automation or
orchestration, and you don’t need any extra functionality, you should have a
look at Metacontroller, especially when you want to implement your
business logic in a language other than Go. 

if your use case involves
extending and customizing Kubernetes with simple automation or
orchestration, and you don’t need any extra functionality, you should have a
look at Metacontroller, especially when you want to implement your
business logic in a language other than Go. 

## Discussion
In many cases, a plain controller working with standard resources is good
enough. This approach has the advantage that it doesn’t need any cluster-
admin permission to register a CRD, but it has its limitations when it comes
to security and validation.
An operator is a good fit for modeling a custom domain logic that fits nicely
with the declarative Kubernetes way of handling resources with reactive
controllers.

More specifically, consider using an operator with CRDs for your
application domain for any of the following situations:
You want tight integration into the already-existing Kubernetes tooling
like kubectl.
You are working on a greenfield project where you can design the
application from the ground up.
You benefit from Kubernetes concepts like resource paths, API groups,
API versioning, and especially namespaces.
You want to have good client support for accessing the API with
watches, authentication, role-based authorization, and selectors for
metadata.

If your custom use case fits these criteria, but you need more flexibility in
how custom resources can be implemented and persisted, consider using a
custom API Server. 

If your use case is not declarative, if the data to manage does not fit into the
Kubernetes resource model, or you don’t need a tight integration into the
platform, you are probably better off writing your standalone API and
exposing it with a classical Service or Ingress object.
The Kubernetes documentation itself also has a chapter for suggestions on
when to use a controller, operator, API aggregation, or custom API
implementation.

