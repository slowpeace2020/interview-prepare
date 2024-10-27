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