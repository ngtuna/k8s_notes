# Pods

Pods are the smallest deployable units of computing that can be created and managed in Kubernetes.

## What is a pod?
A `pod` is a group of one or more containers, the shared storage for those containers, and option about how to run the containers. A `pod` models one or more application containers which are relatively tightly coupled.

Containers within a pod share an IP address and port space, and can find each other via localhost. They can also communicate with each other using standard inter-process communications like `SystemV semaphores` or `POSIX` shared memory.

Application within a pod also have access to shared volumes, which are made available to be mounted into each application filesystem.

Pods are considered to be relatively ephemeral entities. If a node dies, the pods scheduled to that node are deleted. A given pod is not "rescheduled" to a new node; instead, it can be replaced by an identical pod, with even the same name if desired, but with a new UID. Beside, if that pod is deleted for any reason, the volume is also destroyed.

## Motivation for pods
### Management
Pods are a model of the pattern of multiple cooperating processes which form a cohesive unit of service. Pods serve as unit of deployment, horizontal scaling, and replication. They simplify application deployment and management by providing a higher-level abstraction than the set of their constituent applications.

### Resource sharing and communication
Pods enable data sharing and communication among their constituents.

The applications in a pod all use the same network namespace (same IP and port space), and can thus “find” each other and communicate using localhost. Because of this, applications in a pod must coordinate their usage of ports. Each pod has an IP address in a flat shared networking space that has full communication with other physical computers and pods across the network.

The hostname is set to the pod’s Name for the application containers within the pod

### Uses of pods
Pods can be used to host vertically integrated application stacks (ex LAMP), but their primary motivation is to support co-located, co-managed helper programs, such as:
- content management systems, file and data loaders, local cache managers, etc
- log and checkpoint backup, compression, rotation, snapshotting, etc
- data change watchers, log tailers, logging and monitoring adapters, event publishers, etc
- proxies, bridges, and adapters
- controllers, managers, configurators, and updaters.

In general, individual pods are not intended to run multiple instances of the same application.

***Some great examples for using `pod`: Patterns for Composite Containers***
*Example 1: Sidecar containers*
Sidecar containers extend and enhance the "main" container, they take existing containers and make them better. As an example, consider a container that runs the Nginx web server. Add a different container that syncs the file system with a git repository, share the file system between the containers and you have built Git push-to-deploy. But you’ve done it in a modular manner where the git synchronizer can be built by a different team, and can be reused across many different web servers (Apache, Python, Tomcat, etc). Because of this modularity, you only have to write and test your git synchronizer once and reuse it across numerous apps. And if someone else writes it, you don’t even need to do that.

![sidecar containers]
(https://lh3.googleusercontent.com/kwkhauSbZrxJFToohmvA8phCeXTm7Z3-84aKfKB5EeCnfWIT72RB0fZE9heCokFM3C2UHWfacvZfJNrVLfpwq_zk4E09pPFg3xFbH2bcXVmm89wh_QVf_qFvWw-OgtQEuCliUiM)

*Example 2: Ambassador containers*
Ambassador containers proxy a local connection to the world. As an example, consider a Redis cluster with read-replicas and a single write master. You can create a Pod that groups your main application with a Redis ambassador container. Because these two containers share a network namespace, they share an IP address and then your application can open a connection to `localhost` and find the proxy without any service discovery. It is simply connecting to Redis servers on localhost.

This is powerful, not just because of separation of concerns and the fact that different teams can easily own the components, but also because in the development environment, you can simply skip the proxy and connect directly to a Redis server that is running on localhost.

![ambassador containers]
(https://lh5.googleusercontent.com/UwzEbkSGSQPDU97n4qfVPrMC43FHRLYTepMOAG-4d5vdlGpdTpIupOUNKa3BqU0d-ORbcVYEUErjZ-pkJ8TaeRwerg5agQwiMRONEoRGqkRA4Yv7ecyvaAZnB6pec70bLW2T-RU)

*Example 3: Adapter containers*
Adapter containers standardize and normalize output. Consider the task of monitoring N different applications. Each application has a different way of exporting monitoring data. However, monitoring system expects a consistent and uniform data model for the monitoring data it collects. By using the `adapter` pattern of composite containers, you can transform the heterogeneous monitoring data from different systems into a single unified representation. gain because these Pods share namespaces and file systems, the coordination of these two containers is simple and straightforward.

![adapter containers]
(https://lh4.googleusercontent.com/n42wAo-Df7KOwnbTn1eL2vlB_1XmCsMk7RgmtYkkG2o-sRuGx9Y3pz0h-_EkCnDfPDSdgGW8j0zDEgg6GaBhDMoC4o1CfH_BBL7Gv5Z9rY76K3RKGAdVfNg2JdfjY1CtBDf9Pso)

*Conclusion*
In all of these cases, we've used the container boundary as an encapsulation/abstraction boundary that allows us to build modular, reusable components that we combine to build out applications. This reuse enables us to more effectively share containers between different developers, reuse our code across multiple applications, and generally build more reliable, robust distributed systems more quickly. I hope you’ve seen how Pods and composite container patterns can enable you to build robust distributed systems more quickly, and achieve container code re-use.

### Privileged mode for pod
From kubernetes v1.1, any container in a pod can enable privileged mode, using the `privileged` flag on the `SecurityContext` of the container spec. This is useful for containers that want to use linux capabilities like manipulating the network stack and accessing devices. Processes within the container get almost the same privileges that are available to processes outside a container. With privileged mode, it should be easier to write network and volume plugins as separate pods that don’t need to be compiled into the kubelet.
