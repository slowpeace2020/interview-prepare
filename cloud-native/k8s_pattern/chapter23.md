# Chapter 23. Process Containment
>describes the ways to contain a
process to the least privileges it is entitled to.helps make applications more secure by
limiting the attack surface and creating a line of defense. It also prevents
any rogue process from running out of its designated boundary.

## Problem
Many techniques can help improve code security. For
example, static code analysis tools can check the source code for security
flaws. Dynamic scanning tools can simulate malicious attackers with the
goal of breaking into the system through well-known service attacks such as
SQL injection (SQLi), cross-site request forgery (CSRF), and cross-site
scripting (XSS). Then there are tools for regularly scanning the
application’s dependencies for security vulnerabilities. As part of the image
build process, the containers are scanned for known vulnerabilities. This is
usually done by checking the base image and all its packages against a
database that tracks vulnerable packages. These are only a few of the steps
involved in creating secure applications and protecting against malicious
actors, compromised users, unsafe container images, or dependencies with
vulnerabilities.Without runtime process-level
security controls in place, a malicious actor can breach the application code
and attempt to take control of the host or the entire Kubernetes cluster. The
mechanisms we will explore in this chapter demonstrate how to limit a
container only to the permissions it needs to run and apply the least-
privilege principle.

## Solution
When the container is managed by
Kubernetes, the security configurations that will be applied to a container
are controlled by Kubernetes and exposed to the user through the security
context configurations of the Pod and the container specs. The Pod-level
configurations apply to the Pod’s volumes and all containers in the Pod,
whereas container-level configurations apply to a single container. When
the same configurations are set at both Pod and container levels, the values
in the container spec take precedence.

As a developer creating cloud native applications, you typically should not
need to deal with many fine-grained security configurations but instead
have them validated and enforced as global policy. Fine-grained tuning is
usually required when creating specialized infrastructure containers such as
build systems and other plugins that need broader access to the underlying
nodes. Therefore, we will review only the common security configurations
that would be useful for running typical cloud native applications on
Kubernetes.

### Running Containers with a Non-Root User
Container images have a user, and can optionally have a group, to run the
container process. These users and groups are used to control access to files,
directories, and volume mounts. With some other containers, no user is
created and the container image runs as root by default. In others, a user is
created in the container image, but it is not set as the default user to run.
These situations can be rectified by overriding the user at runtime using
securityContext.

example yaml:


### Restricting Container Capabilities
In essence, a container is a process that runs on a node, and it can have the
same privileges a process can have. If the process requires a kernel-level
call, it needs to have the privileges to do so in order to succeed. You can do
this either by running the container as root, which grants all privileges to
the container, or by assigning specific capabilities required for the
application to function.

To make containers more secure, you should provide them with the least
amount of privileges needed to run. The container runtime assigns a set of
default privileges (capabilities) to the container. Contrary to what you might
expect, if the .spec.containers[].securityContext.capabilities
section is left empty, the default set of capabilities defined by the container
runtime are far more generous than most processes need, opening them up
to exploits. A good security practice for locking down the container attack
surface is to drop all privileges and add only the ones you need.

### Avoiding a Mutable Container Filesystem
In general, containerized applications should not be able to write to the
container filesystem because containers are ephemeral and any state will be
lost upon restart.Logs should be written to stdout or forward to a remote log
collector. Such an application can limit the attack surface of the container
further by having a read-only container filesystem. A read-only filesystem
will prevent any rogue user from tampering with the application
configuration or installing additional executables on the disk that can be
used for further exploits.

### Enforcing Security Policies
So far, we’ve explored setting security parameters of the container runtime
using the securityContext definition as part of the Pod and container
specifications. These specifications are created individually per Pod and
usually indirectly through higher abstractions such as Deployments, Jobs,
and CronJobs. But how can a cluster administrator or a security expert
ensure that a collection of Pods follows certain security standards? The
answer is in the Kubernetes Pod Security Standards (PSS) and Pod Security
Admission (PSA) controller. PSS defines a common understanding and
consistent language around security policies, and PSA helps enforce them.
This way, the policies are independent of the underlying enforcement
mechanism and can be applied through PSS or other third-party tools.These policies are grouped in three security profiles that are cumulative,
from highly permissive to highly restrictive, as follows:Privileged,Baseline,Restricted.

The security standards are
applied to a Kubernetes namespace using labels that define the standard
level as described earlier and one or more actions to take when a potential
violation is detected. Following are the actions you can take:
Warn,Audit,Enforce.


example creates a new namespace and configures the security
standards to apply to all Pods that will be created in this namespace. It is
also possible to update the configuration of a namespace or apply the policy
to one or all existing namespaces. 

## Discussion
One of the common security challenges with Kubernetes is running legacy
applications that are not implemented or containerized with Kubernetes
security controls in mind. Running a privileged container can be a challenge
on Kubernetes distributions or environments with strict security policies.It is important to
realize that a container is not only a packaging format and not only a
resource isolation mechanism, but when configured properly, it is also a
security fence.


