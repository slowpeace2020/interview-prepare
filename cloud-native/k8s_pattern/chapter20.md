Chapter 20 Configuration Resource
>uses Kubernetes resources like
ConfigMaps or Secrets to store configuration information.

Kubernetes provides native configuration resources for regular and
confidential data, which allows you to decouple the configuration lifecycle
from the application lifecycle. The Configuration Resource pattern explains
the concepts of ConfigMap and Secret resources and how we can use them,
as well as their limitations.


## Problem
it is better to keep all the configuration data in a single place and not
scattered around in various resource definition files. But it does not make
sense to put the content of a whole configuration file into an environment
variable. So some extra indirection would allow more flexibility, which is
what Kubernetes configuration resources offer.

## Solution
When we are describing ConfigMaps, the same can be
applied most of the time to Secrets too. Besides the actual data encoding
(which is Base64 for Secrets), there is no technical difference for the use of
ConfigMaps and Secrets.

Once a ConfigMap is created and holding data, we can use the keys of a
ConfigMap in two ways:
As a reference for environment variables, where the key is the name of
the environment variable.
As files that are mapped to a volume mounted in a Pod. The key is
used as the filename.The file in a mounted ConfigMap volume is updated when the ConfigMap
is updated via the Kubernetes API. So, if an application supports hot reload
of configuration files, it can immediately benefit from such an update.
However, with ConfigMap entries used as environment variables, updates
are not reflected because environment variables can’t be changed after a
process has been started.
In addition to ConfigMap and Secret, another alternative is to store
configuration directly in external volumes that are then mounted.

Secrets, as with ConfigMaps, can also be consumed as environment
variables, either per entry or for all entries. When a ConfigMap is used as a volume, its complete content is projected
into this volume, with the keys used as filenames.

The mapping of configuration data can be fine-tuned more granularly by
adding additional properties to the volume declaration. Rather than
mapping all entries as files, you can also individually select every key that
should be exposed, the filename, and permissions under which it should be
available.

Besides preventing unwanted updates, using
immutable ConfigMaps and Secrets considerably improves a cluster’s
performance as the Kubernetes API server does not need to monitor
changes on those immutable objects.

## Discussion
ConfigMaps and Secrets allow you to store configuration information in
dedicated resource objects that are easy to manage with the Kubernetes
API. The most significant advantage of using ConfigMaps and Secrets is
that they decouple the definition of configuration data from its usage. This
decoupling allows us to manage the objects that use the configuration
independently of the configuration definition. Another benefit of
ConfigMaps and Secrets is that they are intrinsic features of the platform。However, these configuration resources also have their restrictions: with a 1
MB size limit for Secrets, they can’t store arbitrarily large data and are not
well suited for nonconfiguration application data. You can also store binary
data in Secrets, but since they have to be Base64 encoded, you can use only
around 700 KB data for it. Real-world Kubernetes clusters also put an
individual quota on the number of ConfigMaps that can be used per
namespace or project, so ConfigMap is not a golden hammer。




