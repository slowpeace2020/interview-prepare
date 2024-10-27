# Chapter 16. Sidecar
>describes how to extend and enhance the
functionality of a preexisting container without changing it.

## Problem
Today, to make an HTTP call, we don’t have to write a client library but can
use an existing one. In the same way, to serve a website, we don’t have to
create a container for a web server but can use an existing one. This
approach allows developers to avoid reinventing the wheel and create an
ecosystem with a smaller number of better-quality containers to maintain.
However, having single-purpose reusable containers requires ways of
extending the functionality of a container and a means for collaboration
among containers. The sidecar pattern describes this kind of collaboration,
where a container enhances the functionality of another preexisting
container.

# Solution
Sidecar (sometimes also called Sidekick) is used to describe the
scenario of a container being put into a Pod to extend and enhance another
container’s behavior.
A typical example demonstrating this pattern is of an HTTP server and a
Git synchronizer. The HTTP server container is focused only on serving
files over HTTP and does not know how or where the files are coming from.
Similarly, the Git synchronizer container’s only goal is to sync data from a
Git server to the local filesystem. It does not care what happens once synced
—its only concern is keeping the local folder in sync with the remote Git
server.

in a Sidecar
pattern, there is a main container and a helper container that enhance the
collective behavior. Typically, the main container is the first one listed in the
containers list, and it represents the default container。allows runtime collaboration
of containers and at the same time enables separation of concerns for both
containers, which might be owned by separate teams, using different
programming languages, with different release cycles, etc. It also promotes
replaceability and reuse of containers 。

example: 

## Discussion
With the composition approach, you have multiple containers (processes)
running, health checked, restarted, and consuming resources, as the main
application container does. Modern sidecar containers are small and
consume minimal resources, but you have to decide whether it is worth
running a separate process or whether it is better to merge it into the main
container.

We see two dominating approaches for using sidecars: transparent sidecars
that are invisible to the application, and explicit sidecars that the main
application interacts with over well-defined APIs. Envoy proxy is an
example of a transparent sidecar that runs alongside the main container and
abstracts the network by providing common features such as Transport
Layer Security (TLS), load balancing, automatic retries, circuit breaking,
global rate limiting, observability of L7 traffic, distributed tracing, and
more. All of these features become available to the application by
transparently attaching the sidecar container and intercepting all the
incoming and outgoing traffic to the main container. This is similar to
aspect-oriented programming, in that with additional containers, we
introduce orthogonal capabilities to the Pod without touching the main
container.

An example of an explicit proxy that uses the sidecar architecture is Dapr.
A Dapr sidecar container is injected into a Pod and offers features such as
reliable service invocation, publish-subscribe, bindings to external systems,
state abstraction, observability, distributed tracing, and more. The primary
difference between Dapr and Envoy proxy is that Dapr does not intercept all
the networking traffic going in and out of the application. Rather, Dapr
features are exposed over HTTP and gRPC APIs, which the application
invokes or subscribes to.

envoy和Dapr各给一个YAML example说明

