#Logging Into a Kubernetes Cluster With Kubectl

##View current configs

```
kubectl config view
```

##Add a new cluster

```
kubectl config set-credentials kubeuser/foo.kubernetes.com --username=kubeuser --password=kubepassword
```

##Point to the cluster

```
kubectl config set-cluster foo.kubernetes.com --insecure-skip-tls-verify=true --server=https://foo.kubernetes.com
```

##Create a context. 
This context basically points to the cluster with a specific user. Using (and properly organizing) our different contexts, we can quickly switch across multiple clusters:

```
kubectl config set-context default/foo.kubernetes.com/kubeuser --user=kubeuser/foo.kubernetes.com --namespace=default --cluster=foo.kubernetes.com
```

##Now to use this context

```
kubectl config use-context default/foo.kubernetes.com/kubeuser
```
