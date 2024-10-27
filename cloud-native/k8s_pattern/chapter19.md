# Introduction

Every application needs to be configured, and the easiest way to do this is
by storing configurations in the source code. However, this approach has
the side effect of code and configuration living and dying together. We need
the flexibility to adapt configurations without modifying the application and
recreating its container image. In fact, mixing code and configuration is an
antipattern for a continuous delivery approach, where the application is
created once and then moves unaltered through the various stages of the
deployment pipeline until it reaches production. The way to achieve this
separation of code and configuration is by using external configuration data,
which is different for each environment. The patterns in the following
chapters are all about customizing and adapting applications with external
configurations for various environments.

# Chapter 19. EnvVar Configuration
> uses environment variables to
store configuration data.For small sets of configuration values, the easiest
way to externalize configuration is by putting them into universally
supported environment variables. We’ll see different ways of declaring
environment variables in Kubernetes but also the limitations of using
environment variables for complex configurations

## Problem
it is a bad thing to hardcode
configurations within the application. Instead, the configuration should be
externalized so that we can change it even after the application has been
built. That provides even more value for containerized applications that
enable and promote sharing of immutable application artifacts. But how can
this be done best in a containerized world?

## Solution
The twelve-factor app manifesto recommends using environment variables
for storing application configurations. This approach is simple and works
for any environment and platform. Every operating system knows how to
define environment variables and how to propagate them to applications,
and every programming language also allows easy access to these
environment variables. It is fair to claim that environment variables are
universally applicable. When using environment variables, a typical usage
pattern is to define hardcoded default values during build time, which we
can then overwrite at runtime. Let’s see some concrete examples of how
this works in Docker and Kubernetes.

For Docker images, environment variables can be defined directly in
Dockerfiles with the ENV directive. 

Directly running such an image will use the default hardcoded values. But
in most cases, you want to override these parameters from outside the
image.
When running such an image directly with Docker, environment variables
can be set from the command line by calling Docker.

For Kubernetes, these types of environment variables can be set directly in
the Pod specification of a controller like Deployment or ReplicaSet

In such a Pod template, you not only can attach values directly to
environment variables (as for LOG_FILE), but also can use a delegation to
Kubernetes Secrets and ConfigMaps. The advantage of ConfigMap and
Secret indirection is that the environment variables can be managed
independently from the Pod definition.

environment variables are not secure. Putting sensitive, readable
information into environment variables makes this information easy to read,
and it may even leak into logs.

### DEFAULT VALUES
Default values make life easier, as they take away the burden of
selecting a value for a configuration parameter you might not even
know exists. They also play a significant role in the convention over
configuration paradigm.

However, defaults are not always a good idea.
Sometimes they might even be an antipattern for an evolving
application.
This is because changing default values retrospectively is a difficult
task. First, changing default values means replacing them within the
code, which requires a rebuild. Second, people relying on defaults
(either by convention or consciously) will always be surprised when a
default value changes. We have to communicate the change, and the
user of such an application probably has to modify the calling code as
well.

Changes in default values, however, often make sense, because it is
hard to get default values right from the very beginning. It’s essential
that we consider a change in a default value as a major change, and if
semantic versioning is in use, such a modification justifies a bump in
the major version number. If unsatisfied with a given default value, it is
often better to remove the default altogether and throw an error if the
user does not provide a configuration value. This will at least break the
application early and prominently instead of it doing something
different and unexpected silently.

Considering all these issues, it is often the best solution to avoid default
values from the very beginning if you cannot be 90% sure that a
reasonable default will last for a long time. Passwords or database
connection parameters are good candidates for not providing default
values, as they depend highly on the environment and often cannot be
reliably predicted. Also, if we do not use default values, the
configuration information has to be provided explicitly, which serves as
documentation too.

Two other valuable features that can be used with environment variables are
the downward API and dependent variables.

With a $(...) notation, you can reference environment variables defined
earlier in the env list or coming from an envFrom import. Kubernetes will
resolve those references during the startup of the container.

if you reference a variable defined later in the list, it
will not be resolved, and the $(...) reference will be taken over literally.
In addition, you can also reference environment variables with this syntax
for Pod commands

## Discussion
Environment variables are easy to use, and everybody knows about them.
This concept maps smoothly to containers, and every runtime platform
supports environment variables. But environment variables are not secure,
and are good only for a decent number of configuration values. And when
there are a lot of different parameters to configure, the management of all
these environment variables becomes unwieldy.

Environment variables are universally applicable, and because of that, we
can set them at various levels. This option leads to fragmentation of the
configuration definitions and makes it hard to track for a given environment
variable where it is set. When there is no central place where all
environments variables are defined, it is hard to debug configuration issues.
Another disadvantage of environment variables is that they can be set only
before an application starts, and we cannot change them later. On the one
hand, it’s a drawback that you can’t change configuration “hot” during
runtime to tune the application. However, many see this as an advantage, as
it promotes immutability even to the configuration. Immutability here
means you throw away the running application container and start a new
copy with a modified configuration, very likely with a smooth Deployment
strategy like rolling updates. That way, you are always in a defined and
well-known configuration state.







