Chapter 21. Immutable Configuration
>brings immutability to large
configuration sets by putting them into containers linked to the
application at runtime.

With this pattern, we can not
only use immutable and versioned configuration data, but also overcome the
size limitation of configuration data stored in environment variables or
ConfigMaps.

## Problem
Immutability here means that we can’t change the configuration after the
application has started, in order to ensure that we always have a well-
defined state for our configuration data. In addition, immutable
configuration can be put under version control and follow a change control
process.

There are several options to address the concern of configuration
immutability. The simplest and preferred option is to use ConfigMaps or
Secrets that are marked as immutable in their declaration.

ConfigMaps should be the first
choice if your configuration fits into a ConfigMap and is reasonably easy to
maintain. In real-world scenarios, however, the amount of configuration
data can increase quickly. Although a WildFly application server
configuration might still fit in a ConfigMap, it is quite huge. It becomes
really ugly when you have to nest XML or YAML within YAML—i.e.,
when the content of your configuration is also YAML and you embed this
as within the ConfigMaps YAML section. 


## Solution

To address the concern of complex configuration data, we can put all
environment-specific configuration data into a single, passive data image
that we can distribute as a regular container image. During runtime, the
application and the data image are linked together so that the application
can extract the configuration from the data image. With this approach, it is
easy to craft different configuration data images for various environments.
These images then combine all configuration information for specific
environments and can be versioned like any other container image.
Creating such a data image is trivial, as it is a simple container image that
contains only data. The challenge is the linking step during startup. We can
use various approaches, depending on the platform.

### Docker Volumes
Before looking at Kubernetes, let’s go one step back and consider the
vanilla Docker case. In Docker, it is possible for a container to expose a
volume with data from the container. With a VOLUME directive in a
Dockerfile, you can specify a directory that can be shared later. During
startup, the content of this directory within the container is copied over to
this shared directory. 

docker build -t k8spatterns/config-dev-image:1.0.1 -f Dockerfile-
config . 使用当前目录中的 Dockerfile-config 文件构建一个名为 k8spatterns/config-dev-image、版本为 1.0.1 的 Docker 镜像。

docker run --volumes-from config-dev k8spatterns/welcome-servlet:1.0

该命令将基于 k8spatterns/welcome-servlet:1.0 镜像启动一个新的容器，并且从名为 config-dev 的容器继承数据卷。这意味着 config-dev 容器中的卷将被挂载到新启动的容器中，使得两个容器可以共享存储数据。

When you
move this application from the development environment to the production
environment, all you have to do is change the startup command. There is no
need to alter the application image itself. Instead, you simply volume-link
the application container with the production configuration container

### Kubernetes Init Containers
In Kubernetes, volume sharing within a Pod is perfectly suited for this kind
of linking of configuration and application containers. However, if we want
to transfer this technique of Docker volume linking to the Kubernetes
world, we will find that there is currently no support for container volumes
in Kubernetes. Considering the age of the discussion and the complexity of
implementing this feature versus its limited benefits, it’s likely that
container volumes will not arrive anytime soon.
So containers can share (external) volumes, but they cannot yet directly
share directories located within the containers. To use immutable
configuration containers in Kubernetes, we can use the Init Containers
pattern from Chapter 15 that can initialize an empty shared volume during
startup.

But for Kubernetes init containers, we need help from the
base image to copy over the configuration data to a shared Pod volume. A
good choice for this is busybox, which is still small but allows us to use a
plain Unix cp command for this task.

### OpenShift Templates 选择性了解


### Discussion
Using data containers for the Immutable Configuration pattern is admittedly
a bit involved. Use these only if immutable ConfigMaps and Secret are not
suitable for your use case.
Data containers have some unique advantages:
Environment-specific configuration is sealed within a container.
Therefore, it can be versioned like any other container image.
Configuration created this way can be distributed over a container
registry. The configuration can be examined even without accessing
the cluster.
The configuration is immutable, as is the container image holding the
configuration: a change in the configuration requires a version update
and a new container image.
Configuration data images are useful when the configuration data is
too complex to put into environment variables or ConfigMaps, since it
can hold arbitrarily large configuration data.

As expected, the Immutable Configuration pattern also has certain
drawbacks:
It has higher complexity, because extra container images need to be
built and distributed via registries.
It does not address any of the security concerns around sensitive
configuration data.

Since no image volume support is actually available for Kubernetes
workloads, the technique described here is still limited for use cases
where the overhead of copying over data from init containers to a local
volume is acceptable. We hope that eventually mounting container
images directly as volumes will be possible in the future, but as of
2023, only experimental CSI support is available.
Extra init container processing is required in the Kubernetes case, and
hence we need to manage different Deployment objects for different
environments.

