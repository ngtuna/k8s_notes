# Replication Controller

## What is a replication controller?
A replication controller ensures that a specified number of pod “replicas” are running at any one time, which means always up and available.

## Working with replication controller
### Deleting just a RC
Using `kubectl delete`, specify the `--cascade=false` option
### Isolating pods from a RC
Pods may be removed from a replication controller’s target set by changing their labels. This technique may be used to remove pods from service for debugging, data recovery, etc. Pods that are removed in this way will be replaced automatically (assuming that the number of replicas is not also changed).

## Common usage patterns
### Rescheduling
As mentioned above, whether you have 1 pod you want to keep running, or 1000, a replication controller will ensure that the specified number of pods exists, even in the event of node failure or pod termination (e.g., due to an action by another control agent).
### Scaling
The replication controller makes it easy to scale the number of replicas up or down, either manually or by an auto-scaling control agent, by simply updating the replicas field.
### Rolling updates
The replication controller is designed to facilitate rolling updates to a service by replacing pods one-by-one. The recommended approach is to create a new replication controller with 1 replica, scale the new (+1) and old (-1) controllers one by one, and then delete the old controller after it reaches 0 replicas. This predictably updates the set of pods regardless of unexpected failures.
*Examples*
```
# Update pods of frontend-v1 using new replication controller data in frontend-v2.json.
kubectl rolling-update frontend-v1 -f frontend-v2.json

# Update pods of frontend-v1 using JSON data passed into stdin.
cat frontend-v2.json | kubectl rolling-update frontend-v1 -f -

# Update the pods of frontend-v1 to frontend-v2 by just changing the image, and switching the name of the replication controller.
kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2

# Update the pods of frontend by just changing the image, and keeping the old name.
kubectl rolling-update frontend --image=image:v2

# Abort and reverse an existing rollout in progress (from frontend-v1 to frontend-v2).
kubectl rolling-update frontend-v1 frontend-v2 --rollback
```
## Using Replication Controllers with Services
Multiple replication controllers can sit behind a single service, so that, for example, some traffic goes to the old version, and some goes to the new version.

A replication controller will never terminate on its own, but it isn’t expected to be as long-lived as services.
## Writing programs for Replication
Pods created by a replication controller are intended to be fungible and semantically identical. This is an obvious fit for replicated stateless servers, but replication controllers can also be used to maintain availability of master-elected, sharded, and worker-pool applications. Such applications should use dynamic work assignment mechanisms, such as the [etcd lock module](https://coreos.com/docs/distributed-configuration/etcd-modules/) or [RabbitMQ work queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html).

##Alternatives to Replication Controller
###Bare Pods
###Job
Use a [Job](http://kubernetes.io/docs/user-guide/jobs/) instead of a replication controller for pods that are expected to terminate on their own (i.e. batch jobs)
###DaemonSet
Use a [DaemonSet](http://kubernetes.io/docs/admin/daemons/) instead of a replication controller for pods that provide a machine-level function, such as machine monitoring or machine logging. These pods have a lifetime is tied to machine lifetime: the pod needs to be running on the machine before other pods start, and are safe to terminate when the machine is otherwise ready to be rebooted/shutdown.
