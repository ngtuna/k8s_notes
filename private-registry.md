# Setup docker private registry for k8s

## Prerequisites

For this setup, you must have an ingress controller running for:

- exposing the web portal
- uploading image(s) to the registry

Alternatively, refer to [this doc](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types) for publishing service(s) in k8s.

## Deploy the registry

```
$ kubectl create -f manifests/docker-registry.yaml
```

**Explain the registry**

The image data is stored at /var/lib/registry and mounted to a NFS server. Alternatively you can use a suitable volume setup in your k8s environment for data persistence.

```
...
  volumeMounts:
  - mountPath: /var/lib/registry
    name: data
volumes:
- name: data
  nfs:
    path: /var/nfs/data/docker-registry
    server: nfs.example.com
```

There is no authentication included in this setup. You can add it following [the official guide](https://docs.docker.com/registry/deploying/#native-basic-auth)

The registry is exposed via an ingress route, mostly for the case of uploading image.

## Deploy the web portal

```
$ kubectl create -f manifests/docker-frontend.yaml
```

**Explain the web portal**

The web portal is exposed via an ingress route, mostly for the case of accessing externally.

## Uploading image(s)

```
$ docker tag <image_name>:<tag> docker.example.com/<image_name>:<tag>
$ docker push docker.example.com/<image_name>:<tag>
```

## Deploy to K8S

Deploy as usual, only change is made at the `image` field:
```
image: docker.example.com/<image_name>:<tag>
```
