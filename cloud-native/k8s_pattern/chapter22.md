Chapter 22. Configuration Template
>is useful when large configuration files need to be managed for multiple environments that
differ only slightly.  The generated
configuration is specific to the target runtime environment as reflected by
the parameters used in processing the configuration template.

## Problem
Large configuration files typically differ only slightly for the different
execution environments. This similarity leads to a lot of duplication and
redundancy in the ConfigMaps because each environment has mostly the
same data. The Configuration Template pattern we explore in this chapter
addresses these specific use-case concerns.

## Solution
To reduce duplication, it makes sense to store only the differing
configuration values like database connection parameters in a ConfigMap or
even directly in environment variables. During startup of the container,
these values are processed with configuration templates to create the full
configuration file.

Before the application is started, the fully processed configuration file is put
into a location where it can be directly used like any other configuration
file.
There are two techniques for how such live processing can happen during
runtime:
We can add the template processor as part of the ENTRYPOINT to a
Dockerfile so the template processing becomes directly part of the
container image. The entry point here is typically a script that first
performs the template processing and then starts the application. The
parameters for the template come from environment variables.
With Kubernetes, a better way to perform initialization is with an init
container of a Pod in which the template processor runs and creates the
configuration for the application containers in the Pod. 

For Kubernetes, the init container approach is the most appealing because
we can use ConfigMaps directly for the template parameters.

## Discussion
The Configuration Template pattern builds on top of the Configuration
Resource pattern and is especially suited when we need to operate
applications in different environments with similar complex configurations.
However, the setup with configuration templates is more complicated and
has more moving parts that can go wrong. Use it only if your application
requires huge configuration data. Such applications often require a
considerable amount of configuration data from which only a small fraction
is dependent on the environment. Even when copying over the whole
configuration directly into the environment-specific ConfigMap works
initially, it puts a burden on the maintenance of that configuration because it
is doomed to diverge over time. For such a situation, this template approach
is perfect。

