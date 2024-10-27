# Chapter 17. Adapter

>takes a heterogeneous system and makes it
conform to a consistent unified interface that can be consumed by the
outside world.The Adapter
pattern inherits all its characteristics from the Sidecar pattern but has the
single purpose of providing adapted access to the application.

## Problem
Today, it is common to see
multiple teams using different technologies and creating distributed systems
composed of heterogeneous components. This heterogeneity can cause
difficulties when all components have to be treated in a unified way by
other systems. The Adapter pattern offers a solution by hiding the
complexity of a system and providing unified access to it.

A
major prerequisite for successfully running and supporting distributed
systems is providing detailed monitoring and alerting. Moreover, if we have
a distributed system composed of multiple services we want to monitor, we
may use an external monitoring tool to poll metrics from every service and
record them.
However, services written in different languages may not have the same
capabilities and may not expose metrics in the same format expected by the
monitoring tool. This diversity creates a challenge for monitoring such a
heterogeneous application from a single monitoring solution that expects a
unified view of the whole system.

## Solution
With the Adapter pattern, it is possible to
provide a unified monitoring interface by exporting metrics from various containers into one standard format and protocol.

With this approach, every service represented by a Pod, in addition to the
main application container, would have another container that knows how
to read the custom application-specific metrics and expose them in a generic
format understandable by the monitoring tool. We could have one adapter
container that knows how to export Java-based metrics over HTTP and
another adapter container in a different Pod that exposes Python-based
metrics over HTTP. For the monitoring tool, all metrics would be available
over HTTP and in a common, normalized format.

For this use case, an adapter is a perfect fit: a sidecar container starts a small
HTTP server and on every request, reads the custom log file and transforms
it into a Prometheus-understandable format.

Another use of this pattern is logging. Different containers may log
information in different formats and levels of detail. An adapter can
normalize that information, clean it up, enrich it with contextual
information by using the Self Awareness pattern and then make it available for pickup by the centralized log aggregator.

## Discussion
The Adapter is a specialization of the Sidecar pattern. It acts as a reverse proxy to a heterogeneous system by hiding
its complexity behind a unified interface. Using a distinct name different
from the generic Sidecar pattern allows us to more precisely communicate
the purpose of this pattern.





