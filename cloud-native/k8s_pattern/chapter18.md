# Chapter 18. Ambassador

>describes a proxy that decouples access to
external services. acts as a proxy to the outside world.

## Problem
The difficulty
in accessing other services may be due to dynamic and changing addresses,
the need for load balancing of clustered service instances, an unreliable
protocol, or difficult data formats. Ideally, containers should be single-
purposed and reusable in different contexts. But if we have a container that
provides some business functionality and consumes an external service in a
specialized way, the container will have more than one responsibility.

Consuming the external service may require a special service discovery
library that we do not want to put in our container. Or we may want to swap
different kinds of services by using different kinds of service-discovery
libraries and methods. This technique of abstracting and isolating the logic
for accessing other services in the outside world is the goal of this
Ambassador pattern.

## Solution
Accessing a local cache in the development environment may be a simple
configuration, but in the production environment, we may need a client
configuration that can connect to the different shards of the cache.
Another
example is consuming a service by looking it up in a registry and
performing client-side service discovery. A third example is consuming a
service over a nonreliable protocol such as HTTP, so to protect our
application, we have to use circuit-breaker logic, configure timeouts,
perform retries, and more.In all of these cases, we can use an ambassador container that hides the
complexity of accessing the external services and provides a simplified
view and access to the main application container over localhost.For development purposes, this ambassador container can be easily
exchanged with a locally running in-memory key-value store like
memcached.

## Discussion
At a higher level, the Ambassador pattern is a Sidecar pattern. The main
difference between ambassador and sidecar is that an ambassador does not
enhance the main application with additional capability. Instead, it acts
merely as a smart proxy to the outside world (this pattern is sometimes
referred to as the Proxy pattern). This pattern can be useful for legacy
applications that are difficult to modify and extend with modern networking
concepts such as monitoring, logging, routing, and resiliency patterns.

The benefits of the Ambassador pattern are similar to those of the Sidecar
pattern—both allow you to keep containers single-purposed and reusable.
With such a pattern, our application container can focus on its business
logic and delegate the responsibility and specifics of consuming the external
service to another specialized container. This also allows you to create
specialized and reusable ambassador containers that can be combined with
other application containers.



