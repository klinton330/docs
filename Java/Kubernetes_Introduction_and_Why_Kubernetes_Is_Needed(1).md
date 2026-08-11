# Kubernetes --- Introduction and Why Kubernetes Is Needed

## 1. Overview

Kubernetes is a **container orchestration platform**. To understand why
Kubernetes is required, we first need to understand the limitations of
running containers directly with Docker.

> **Key idea:** Docker helps us run and manage containers. Kubernetes
> helps us manage containers at scale across multiple machines.

------------------------------------------------------------------------

## 2. Docker vs Kubernetes

  -----------------------------------------------------------------------
  Docker                              Kubernetes
  ----------------------------------- -----------------------------------
  Container platform                  Container orchestration platform

  Runs containers                     Manages containers at scale

  Primarily operates within a host    Manages a cluster of nodes

  Container lifecycle management      Application/container orchestration

  Limited built-in scaling/healing    Provides scaling and self-healing
  capabilities                        mechanisms

  Suitable for running containers     Designed for distributed production
                                      workloads
  -----------------------------------------------------------------------

The important question is not just the textbook definition:

> **What problems become difficult when managing containers directly?**

------------------------------------------------------------------------

## 3. Why Do We Need Kubernetes?

A small number of containers can be managed directly with Docker.
Production environments, however, can involve:

-   Hundreds or thousands of containers
-   Multiple servers
-   Large numbers of users
-   Container failures
-   Changing traffic levels
-   High availability requirements
-   Load balancing
-   Automatic scaling
-   Automatic recovery

The four major problems discussed are:

1.  Single-host limitation
2.  Lack of automatic healing
3.  Lack of automatic scaling
4.  Enterprise-level capabilities

------------------------------------------------------------------------

## 4. Problem 1 --- Single Host

With a basic Docker setup, containers may run on the same host and share
the host's resources.

``` text
EC2 Instance
     |
   Docker
     |
+----+----+----+----+
| C1 | C2 | C3 | C4 |
+----+----+----+----+
```

If one container consumes excessive resources, other workloads on the
same host can be affected.

### Kubernetes Solution

Kubernetes uses a **cluster** consisting of multiple nodes. Workloads
can be scheduled across available nodes rather than being tied to one
machine.

``` text
Kubernetes Cluster
        |
   +----+----+
   |         |
 Node 1    Node 2
   |         |
 Pods       Pods
```

------------------------------------------------------------------------

## 5. What Is a Kubernetes Cluster?

A cluster is a group of nodes managed together.

``` text
Kubernetes Cluster
|
├── Control Plane
├── Worker Node 1
│   ├── Pod
│   └── Pod
├── Worker Node 2
│   ├── Pod
│   └── Pod
└── Worker Node 3
    ├── Pod
    └── Pod
```

Kubernetes is generally deployed as a cluster in production. A
single-node installation can also be used for development and learning.

> **Terminology note:** Modern Kubernetes documentation generally uses
> **control plane** rather than the older term **master node**.

------------------------------------------------------------------------

## 6. Problem 2 --- Auto Healing

Containers are **ephemeral**, meaning they can be short-lived and can
stop or disappear.

``` text
Application
    |
 Container
    |
    X
Container stopped
```

In a basic setup, someone may need to detect failures and restart
workloads. This becomes impractical when an environment contains
thousands of containers.

### Kubernetes Solution

Kubernetes continuously works toward the **desired state**.

For example, if the desired number of replicas is three:

``` text
Desired:  C1  C2  C3
Actual:   C1  C2
               ↓
        Kubernetes restores
               ↓
Desired:  C1  C2  C3
```

Kubernetes works to maintain the desired number of running instances.

------------------------------------------------------------------------

## 7. Problem 3 --- Auto Scaling

Application traffic can increase significantly during events,
promotions, festivals, or popular releases.

``` text
10,000 users
     ↓
100,000 users
     ↓
1,000,000 users
```

A single application instance may not be able to handle the increased
demand.

Instead of manually creating additional application instances,
Kubernetes can increase the number of replicas based on configured
scaling policies.

------------------------------------------------------------------------

## 8. Horizontal Scaling

Kubernetes supports **Horizontal Pod Autoscaling (HPA)**.

For example:

``` text
Minimum replicas = 2
Maximum replicas = 10
CPU threshold = 80%

Traffic increases
      ↓
CPU reaches threshold
      ↓
More replicas
      ↓
Application capacity increases
```

HPA can increase or decrease the number of application replicas based on
resource utilization or configured metrics.

------------------------------------------------------------------------

## 9. Load Balancing

When multiple instances of an application exist, users should normally
access a single application endpoint rather than individual instance
addresses.

``` text
                 Users
                   |
                   ↓
             Load Balancer
              /    |    \
             /     |     \
           Pod1   Pod2   Pod3
```

The load-balancing layer distributes incoming traffic among available
application instances.

------------------------------------------------------------------------

## 10. Problem 4 --- Enterprise-Level Capabilities

Production applications typically require more than simply running
containers.

Common requirements include:

-   Load balancing
-   Security controls
-   Networking
-   Scaling
-   Self-healing
-   API gateway capabilities
-   Traffic management
-   Access control
-   High availability

Kubernetes provides an orchestration foundation, while additional
components and controllers can provide specialized capabilities.

------------------------------------------------------------------------

## 11. Kubernetes Is Extensible

Kubernetes provides an extensible architecture.

Important mechanisms include:

-   Custom Resources
-   Custom Resource Definitions (CRDs)
-   Controllers
-   Operators
-   Ingress Controllers

Conceptually:

``` text
              Kubernetes
                  |
        +---------+---------+
        |                   |
     Core APIs        Custom Resources
                            |
                         Controller
                            |
                     External System
```

Ingress Controllers are one example of extending Kubernetes networking
capabilities.

------------------------------------------------------------------------

## 12. Kubernetes and YAML

Kubernetes resources are commonly defined using YAML manifests.

The YAML describes the **desired state** of a resource.

Example:

``` yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-app

spec:
  replicas: 3
```

The basic flow is:

``` text
YAML
  ↓
Desired State
  ↓
Kubernetes
  ↓
Managed Resources
```

------------------------------------------------------------------------

## 13. Pods vs Containers

With Docker, we commonly talk about containers.

In Kubernetes, the basic deployable unit is a **Pod**.

A Pod can contain one or more containers.

``` text
Kubernetes
    |
   Pod
    |
Container(s)
```

A useful mental model is:

> Docker works directly with containers, while Kubernetes manages
> workloads through Kubernetes resources such as Pods and Deployments.

------------------------------------------------------------------------

## 14. Docker → Kubernetes Mental Model

### Docker

``` text
Docker
|
├── Build image
├── Run container
├── Stop container
├── Start container
└── Manage container lifecycle
```

### Kubernetes

``` text
Kubernetes
|
├── Manage Pods
├── Schedule workloads
├── Maintain desired state
├── Scale workloads
├── Recover failed workloads
├── Distribute workloads across nodes
├── Provide service discovery/networking
└── Integrate with additional infrastructure
```

Kubernetes therefore operates at a higher orchestration level than
simply running individual containers.

------------------------------------------------------------------------

## 15. Complete Example

Consider an e-commerce application.

Initially:

``` text
Users
  |
  ↓
Load Balancer
  |
  ↓
Pod 1
```

As traffic increases:

``` text
Users
  |
  ↓
Load Balancer
  |
  +---- Pod 1
  |
  +---- Pod 2
  |
  +---- Pod 3
```

If a Pod fails, Kubernetes works to restore the desired number of
replicas.

If a node becomes unavailable, Kubernetes can schedule replacement
workloads onto other available nodes, depending on cluster configuration
and available resources.

This demonstrates:

-   Scaling
-   Self-healing
-   Scheduling
-   Multi-node operation
-   Load distribution

------------------------------------------------------------------------

## 16. The Four Problems --- Quick Summary

  -----------------------------------------------------------------------
  Problem                 Basic Docker Challenge  Kubernetes Concept
  ----------------------- ----------------------- -----------------------
  **Single Host**         Workloads may depend on Cluster + scheduling
                          one host                

  **No Auto Healing**     Failed workloads may    Controllers + desired
                          need intervention       state

  **No Auto Scaling**     Manual creation of      HPA / replica
                          additional instances    management

  **Enterprise            Additional              Kubernetes ecosystem +
  Requirements**          infrastructure/tools    extensibility
                          are needed              
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 17. Why Kubernetes Is Important

The growth of microservices and containers creates a need for
orchestration.

The progression can be viewed as:

``` text
Microservices
     ↓
Containers
     ↓
Many containers
     ↓
Multiple machines
     ↓
Need orchestration
     ↓
Kubernetes
```

As the number of containers, hosts, users, and operational requirements
increases, manually managing containers becomes increasingly difficult.

Kubernetes provides an orchestration layer to automate this management.

------------------------------------------------------------------------

## 18. Kubernetes Learning Path

A logical learning sequence is:

``` text
1. Containers
      ↓
2. Docker fundamentals
      ↓
3. Why orchestration is required
      ↓
4. Kubernetes architecture
      ↓
5. Pods
      ↓
6. Deployments
      ↓
7. Services
      ↓
8. Ingress
      ↓
9. Scaling
      ↓
10. Controllers
      ↓
11. Admission Controllers
      ↓
12. Advanced Kubernetes
```

------------------------------------------------------------------------

## 19. Interview-Ready Explanation

### What is the difference between Docker and Kubernetes?

> **Docker is a container platform used to build, package, and run
> containers, whereas Kubernetes is a container orchestration platform
> used to manage containerized applications across a cluster of
> machines. Kubernetes provides capabilities such as workload
> scheduling, scaling, self-healing, and service management, making it
> suitable for managing large-scale containerized applications.**

### Why do we need Kubernetes?

> **When the number of containers and hosts increases, manually managing
> containers becomes difficult. We need mechanisms for scheduling
> workloads, maintaining the desired number of application instances,
> automatically recovering failed workloads, scaling applications based
> on demand, and managing workloads across multiple nodes. Kubernetes
> provides these orchestration capabilities.**

------------------------------------------------------------------------

## 20. Key Takeaways

``` text
Docker
  ↓
Runs containers

Kubernetes
  ↓
Orchestrates containerized workloads

Kubernetes Cluster
  ↓
Multiple nodes

Pods
  ↓
Basic deployable unit

Deployment
  ↓
Manages application replicas

HPA
  ↓
Automatic horizontal scaling

Controllers
  ↓
Continuously work toward desired state

Kubernetes ecosystem
  ↓
Extends Kubernetes with additional capabilities
```

### Big Picture

**Docker answers:**

> "How do I run this container?"

**Kubernetes answers:**

> "How do I reliably run and manage containerized applications across a
> cluster while handling failures, scaling, scheduling, and networking?"
