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

