# Chapter 26. Access Control
>allows users and application workloads
to authenticate and interact with the Kubernetes API server.

In
2022, security researchers made a troubling discovery: nearly one million
Kubernetes instances were left exposed on the internet due to
misconfigurations.1 Using specialized security scanners, researchers were
able to easily access these vulnerable nodes, highlighting the need for
stringent access-control measures to protect the Kubernetes control plane.access control on the Kubernetes
platform becomes critical. In this chapter, we delve into the Access Control
pattern and explore the concepts of Kubernetes authorization. With the
potential risks and consequences at stake, it’s never been more important to
ensure the security of your Kubernetes deployment.

## Problem
Security is a crucial concern when it comes to operating applications. At the
core of security are two essential concepts: authentication and
authorization.

Authentication focuses on identifying the subject, or who, of an operation
and preventing access by unauthorized actors. Authorization, on the other
hand, involves determining the permissions for what actions are allowed on
resources.

On the other hand, developers are
typically more concerned with authorization, such as who can perform
which operations in the cluster and access specific parts of an application.
To secure access to their applications running on top of Kubernetes,
developers must consider a range of security strategies, from simple web-
based authentication to sophisticated single-sign-on scenarios involving
external providers for identity and access management. At the same time,
access control to the Kubernetes API server is also an essential concern for
applications running on Kubernetes.

it crucial to
have fine-grained access management and restrictions in place to limit the
impact of any potential security breaches.

## Solution
Every request to the Kubernetes API server has to pass through three stages
—Authentication, Authorization, and Admission Control.

### Authentication
we won’t go into too much detail about authentication
because it is mainly an administration concern. But it’s good to know which
options are available, so let’s have a look at the pluggable authentication
strategies Kubernetes has to offer that an administrator can configure: Bearer Tokens,Client certificates,Authenticating Proxy,Static Token files,Webhook Token Authentication.

Kubernetes allows you to use multiple authentication plugins
simultaneously, such as Bearer Tokens and Client certificates. If the Bearer
Token strategy authenticates a request, Kubernetes won’t check the Client
certificates, and vice versa. Unfortunately, the order in which these
strategies are evaluated is not fixed, so it’s impossible to know which one
will be checked first.

### Authorization
Kubernetes provides RBAC as a standard way to manage access to the
system. 

### Admission Controllers
Admission controllers are a feature of the Kubernetes API server that
allows you to intercept requests to the API server and take additional
actions based on those requests. For example, you can use them to enforce
policies, perform validations, and modify incoming resources.
Kubernetes uses Admission controller plugins for implementing various
functions. The functionality ranges from setting default values on specific
resources (like the default storage class on persistent volumes), to
validations (like the allowed resource limits for Pods), by calling external
web hooks.we will focus on the authorization
aspect and how we can configure a fine-grained permission model for
securing access to the Kubernetes API server.
As mentioned, authentication has two fundamental parts and authorization:
the who, represented by a subject that can be either a human person or a
workload identity, and the what, representing the actions those subjects can
trigger at the Kubernetes API server. 

### Subject
A subject is all about the who, the identity associated with a request to the
Kubernetes API server. In Kubernetes, there are two kinds of subjects, human users and service accounts that represent the
workload identity of Pods.

Human users and ServiceAccounts can be separately grouped in user
groups and service account groups, respectively. Those groups can act as a
single subject in which all members of the group share the same permission
model. 

#### Users
Unlike many other entities in Kubernetes, human users are not defined as
explicit resources in the Kubernetes API. This design decision implies that
you can’t manage users via an API call. The authentication and mapping to
a user subject happens outside the usual Kubernetes API machinery by
external user management.

#### Service accounts
Service accounts in Kubernetes represent nonhuman actors within the
cluster and are used as workload identities. They are associated with Pods
and allow running processes inside a Pod to communicate with the
Kubernetes API Server. Service accounts in Kubernetes are authenticated by the API server.

Thanks to the new
projected volume type, the token is available only to the Pod and is not
exposed as an additional resource, which reduces the attack surface. You
can still create a Secret manually to contain a ServiceAccount’s token.

Image pull secrets: Kubernetes provides a shortcut
by allowing you to attach a pull secret directly to a ServiceAccount in
the top-level field imagePullSecrets. Every Pod associated with the
ServiceAccount will automatically have the pull secrets injected into its
specification when it is created. This automation eliminates the need to
manually include the image pull secrets in the Pod specification every
time a new Pod is created in the namespace, reducing the manual effort
required.

Mountable secrets: The secrets field in the ServiceAccount resource allows you to specify
which secrets a Pod associated with the ServiceAccount can mount. You
can enable this restriction by adding the kubernetes.io/enforce-
mountable-secrets annotation to the ServiceAccount. If this
annotation is set to true, only the Secrets listed will be allowed to be
mounted by Pods associated with the ServiceAccount.

#### Groups
Both user and service accounts in Kubernetes can belong to one or more
groups. Groups are attached to requests by the authentication system and are used to grant permissions to all group members.

As mentioned earlier, groups can be freely defined and managed by the
identity provider to create groups of subjects with the same permission
model. A set of predefined groups in Kubernetes are also implicitly defined
and have a system: prefix in their name.

### Role-Based Access Control
In Kubernetes, Roles define the specific actions that a subject can perform
on particular resources. You can then assign these Roles to subjects, such as
users or service accounts, as described in “Subject”, through the use of
RoleBindings. Roles and RoleBindings are Kubernetes resources that can
be created and managed like any other resource. They are tied to a specific
namespace and apply to its resources.
In Kubernetes RBAC, it is important to understand that there is a many-to-
many relationship between subjects and Roles. This means that a single
subject can have multiple Roles, and a single Role can be applied to
multiple subjects.




