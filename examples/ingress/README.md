# This is an example for ingress rule and ingress controller

**Setup ingress controller first**

```console
$ kubectl create -f nginx-ingress-controller.yaml
```

**Create backend service and deployment**

```console
$ kubectl create -f backend.yaml
```

**Finally, create an Ingress rule for it**

```console
$ kubectl create -f ingress-rule.yaml
```
