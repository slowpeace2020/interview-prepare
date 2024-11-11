Chapter 30. Image Builder
moves the aspect of building application
images onto the cluster itself.

Kubernetes is a general-purpose orchestration engine, suitable not only for
running applications but also for building container images. The Image
Builder pattern explains why it makes sense to build the container images
within the cluster and what techniques exist today for creating images
within Kubernetes.


## Problem
what about building the application
itself? The classic approach is to build container images outside the cluster,
push them to a registry, and refer to them in the Kubernetes Deployment
descriptors. However, building within the cluster has several advantages.
If your company policies allow, having only one cluster for everything is
advantageous. Building and running applications in one place can
considerably reduce maintenance costs. It also simplifies capacity planning
and reduces platform resource overhead.
Typically, continuous integration (CI) systems like Jenkins are used to build
images. Building with a CI system is a scheduling problem for efficiently
finding free computing resources for build jobs. At the heart of Kubernetes
is a highly sophisticated scheduler that is a perfect fit for this kind of
scheduling challenge.

Once we move to continuous delivery (CD), where we transition from
building images to running containers, if the build happens within the same
cluster, both phases share the same infrastructure and ease transition.

When implementing this Image Builder pattern, the cluster knows both—
the build of an image and its deployment—and can automatically do a
redeployment if a base image changes.

Having seen the benefits of building images on the platform, let’s look at
what techniques exist for creating images in a Kubernetes cluster.

## Solution
Each of these tools has a unique
focus, but for in-cluster builds, we can identify these high-level categories:
Container image builder,Build orchestration

### Container Image Builder
One of the essential prerequisites for building images from within a cluster
is creating images without having privileged access to the node host.

#### Dockerfile-Based builders

#### Multilanguage builders
#### Specialized builders
