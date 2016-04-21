#Services
A `service` is an abstraction which defines a logical set of `pods` and a policy by which to access them - sometimes called a *micro-service*. The set of `pods` targeted by a `service` is (usually) determined by a `label selector`.

The `service` abstraction enables application decoupling, while, for ex, frontends do not care which backend they use. The frontends should not need to be aware of that or keep track of the list of backends themselves.

##Defining a service
Suppose you have a set of `pods` that each expose port `9376` and carry a label "app=MyApp".
```
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            "app": "MyApp"
        },
        "ports": [
            {
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9376
            }
        ]
    }
}
```
This specification will create a new `service` object named “my-service” which targets TCP port 9376 on any `pod` with the "app=MyApp" label.

Note that a `service` can map an incoming `port` to any `targetPort`. By default the `targetPort` will be set to the same value as the `port` field. It's more interesting that `targetPort` can be a string, referring to the name of a `port` in the backend pods. The actual port number assigned to that name can be different in each backend pod. This offers a lot of flexibility for deploying and evolving your `service`. For example, you can change the port number that pods expose in the next version of your backend software, without breaking clients.

##Service without Selectors
`Services` generally abstract access to Kubernetes `Pods`, but they can also abstract other kinds of backends.
- You want to have an external database cluster in production, but in test you use your own databases.
- You want to point your service to a service in another `Namespace` or on another cluster.
- You are migrating your workload to Kubernetes and some of your backends run outside of Kubernetes.

In any of these scenarios you can define a `service` without a `selector`:
```
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "ports": [
            {
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9376
            }
        ]
    }
}
```
Because this `service` has no `selector`, the corresponding ***Endpoints*** object will not be created. You can manually map the service to your own specific endpoints:
```
{
    "kind": "Endpoints",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "subsets": [
        {
            "addresses": [
                { "ip": "1.2.3.4" }
            ],
            "ports": [
                { "port": 9376 }
            ]
        }
    ]
}
```
Accessing a `Service` without a `selector` works the same as if it had selector. The traffic will be routed to ***endpoints*** defined by the user (1.2.3.4:9376 in this example)
##Virtual IPs and service proxies
Every node in a Kubernetes cluster runs a `kube-proxy`. This application is responsible for implementing a form of virtual IP for `Services`. In Kubernetes v1.0 the proxy was purely in userspace. In Kubernetes v1.1 an iptables proxy was added, but was not the default operating mode. In Kubernetes v1.2 we expect the iptables proxy to be the default.
As of Kubernetes v1.0, `Services` are a “layer 3” (TCP/UDP over IP) construct. In Kubernetes v1.1 the Ingress API was added (beta) to represent “layer 7” (HTTP) services.

Any traffic bound for the Service’s IP:Port is proxied (by *kube-proxy*) to an appropriate backend pod without the clients knowing anything about Kubernetes or Services or Pods. By default, the choice of backend is round robin.
##Multi-Port Services
Many Services need to expose more than one port. For this case, Kubernetes supports multiple port definitions on a Service object. When using multiple ports you must give all of your ports names, so that endpoints can be disambiguated
```
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            "app": "MyApp"
        },
        "ports": [
            {
                "name": "http",
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9376
            },
            {
                "name": "https",
                "protocol": "TCP",
                "port": 443,
                "targetPort": 9377
            }
        ]
    }
}
```
##Choosing your own IP address
You can specify your own cluster IP address as part of a `Service` creation request. To do this, set the **spec.clusterIP** field

##Discovering services
Kubernetes supports 2 primary modes of finding a `Service` - environment variables and DNS.
###Environment variables
When a `Pod` is run on a Node, the kubelet adds a set of environment variables for each active `Service`. It supports both Docker links compatible variables (see makeLinkVariables) and simpler {SVCNAME}_SERVICE_HOST and {SVCNAME}_SERVICE_PORT variables.

For example, the Service "redis-master" which exposes TCP port 6379 and has been allocated cluster IP address 10.0.0.11 produces the following environment variables:
```
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```
*Ordering requirement:* any `Service` that a `Pod` wants to access must be created before the `Pod` itself, or else the environment variables will not be populated. DNS does not have this restriction.
###DNS
An optional (though strongly recommended) cluster add-on is a DNS server. The DNS server watches the Kubernetes API for new `Services` and creates a set of DNS records for each. If DNS has been enabled throughout the cluster then all `Pods` should be able to do name resolution of `Services` automatically.

For example, if you have a `Service` called "my-service" in Kubernetes `Namespace` "my-ns" a DNS record for "my-service.my-ns" is created. `Pods` which exist in the "my-ns" `Namespace` should be able to find it by simply doing a name lookup for "my-service". `Pods` which exist in other `Namespaces` must qualify the name as "my-service.my-ns". The result of these name lookups is the `cluster IP`.
##Headless services
Sometimes you don’t want load-balancing and a single service IP. In this case, you can create “headless” services by specifying "None" for the cluster IP (spec.clusterIP).

For such `Services`, a `cluster IP` is not allocated. DNS is configured to return **multiple A records** (addresses) for the `Service` name, **which point directly to the Pods backing the Service**. Additionally, the `kube-proxy` does not handle these `services` and there is no load balancing or proxying done by the platform for them. The endpoints controller will still create Endpoints records in the API.
##Publishing services - service types
Service could be exposed onto an external or internal network. Kubernetes `ServiceTypes` allow you to specify what kind of service you want.

Valid values for the **ServiceType** field are:
- **ClusterIP**: use a cluster-internal IP only - this is the default and is discussed above. Choosing this value means that you want this service to be reachable only from inside of the cluster.
- **NodePort**: on top of having a cluster-internal IP, expose the service on a port on each node of the cluster (the same port on each node). You’ll be able to contact the service on any **NodeIP:NodePort** address.
- **LoadBalancer**: on top of having a cluster-internal IP and exposing service on a NodePort also, ask the cloud provider for a load balancer which forwards to the `Service` exposed as a **NodeIP:NodePort** for each Node.

###Type NodePort
Kubernetes master will allocate a port from a flag-configured range (default: 30000-32767), and each Node will proxy that port (the same port number on every Node) into your `Service`

This gives developers the freedom to set up their own load balancers, to configure cloud environments that are not fully supported by Kubernetes, or even to just expose one or more nodes’ IPs directly.

###Type LoadBalancer
On cloud providers which support external load balancers, setting the type field to **"LoadBalancer"** will provision a load balancer for your `Service`

```
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            "app": "MyApp"
        },
        "ports": [
            {
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9376,
                "nodePort": 30061
            }
        ],
        "clusterIP": "10.0.171.239",
        "loadBalancerIP": "78.11.24.19",
        "type": "LoadBalancer"
    },
    "status": {
        "loadBalancer": {
            "ingress": [
                {
                    "ip": "146.148.47.155"
                }
            ]
        }
    }
}
```
##External IPs
If there are external IPs that route to one or more cluster nodes, Kubernetes services can be exposed on those `externalIPs`. `externalIPs` are not managed by Kubernetes and are the responsibility of the cluster administrator. In the example below, my-service can be accessed by clients on 80.11.12.10:80 (externalIP:port)
```
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            "app": "MyApp"
        },
        "ports": [
            {
                "name": "http",
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9376
            }
        ],
        "externalIPs" : [
            "80.11.12.10"
        ]
    }
}
```
##Future works
In the future we envision that the proxy policy can become more nuanced than simple round robin balancing, for example master-elected or sharded.
