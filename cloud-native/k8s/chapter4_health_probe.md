# Chapter 4. Health Probe

>dictates that every container should implement specific APIs to help the platform observe and maintain the application healthily.

## Problem
Kubernetes regularly checks the container process status and restarts it if
issues are detected. However, from practice, we know that checking the
process status is not sufficient to determine the health of an application. Alternatively, an application may freeze
because it runs into an infinite loop, deadlock, or some thrashing (cache,
heap, process). To detect these kinds of situations, Kubernetes needs a
reliable way to check the health of applications—that is, not to understand
how an application works internally, but to check whether the application is
functioning as expected and capable of serving consumers.

## Solution

### Process Health Checks
It is not enough, and
other types of health checks are also necessary

### Liveness Probes
If your application runs into a deadlock, it is still considered healthy from
the process health check’s point of view. To detect this kind of issue and
any other types of failure according to your application business logic,
Kubernetes has liveness probes—regular checks performed by the Kubelet
agent that asks your container to confirm it is still healthy.

it offers more flexibility regarding which methods to
use for checking the application health, as follows:
HTTP probe
Performs an HTTP GET request to the container IP address and expects
a successful HTTP response code between 200 and 399.
TCP Socket probe
Assumes a successful TCP connection.
Exec probe
Executes an arbitrary command in the container’s user and kernel
namespace and expects a successful exit code (0).
gRPC probe
Leverages gRPC’s intrinsic support for health checks.

the health check behavior can be influenced
with the following parameters:
initialDelaySeconds
periodSeconds
timeoutSeconds
failureThreshold


the result of not passing a health check is that your container will restart. If restarting your container does not help, there is no benefit to having a failing health check
as Kubernetes restarts your container without fixing the underlying issue.

### Readiness Probes
