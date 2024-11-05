2024.11.02

# Ingress2gateway

If I had to define a tool such as "ingress2gateway" succinctly, I would say that it's a tool designed to translate Kubernetes Ingress resources to Gateway API resources.

It ought to be able take any Ingress resource, and produce the Gateway + HttpRoute equivalent.

Here is an example of a valid and working Ingress object applied to a cluster with Istio installed (yes, Istio supports the Ingress resource, see [here](https://istio.io/latest/docs/tasks/traffic-management/ingress/kubernetes-ingress/) for more):


```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: istio
  name: httpbin-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: httpbin
            port:
              number: 8000
```

At the most basic level, one ought to be able to run the following command in order for it to produce the transformed yaml to stdout (standard output):

```shell
i2gw < ingress.yaml
```

The reality is different, in the following ways:

1. The name of the tool is ingress2gateway, so:

    ```shell
    ingress2gateway < ingress.yaml
    ```

1. The input does not come from stdin, it's specified with the flag `--input-file`, so:

    ```shell
    ingress2gateway --input-file ingress.yaml
    ```

   It might be worth noting that the tool provides the alternative ability to read ingress objects directly from a Kubernetes cluster.

1. One must use the `print` subcommand, so:

    ```shell
    ingress2gateway print --input-file ingress.yaml
    ```

So far, the above differences are perfectly reasonable and logical.

Here is where things get problematic..

1. One must specify a provider.  Here is the list of providers currently supported:  

    ```
    ingress-nginx
    istio
    kong
    gce
    openapi3
    apisix
    ```

The `istio` provider does not translate Ingress objects: it translates Istio's Gateway+VirtualService resources to the K8s Gateway API. That's a nice touch, but not what I'm after at the moment.

Presumably each provider implementation goes beyond supporting just a generic Ingress object, and is able to understand implementation-specific annotations (annotations were a way for implementers to support features that were not part of the Ingress resource specification).

The glaring problem is that there is no way to get the tool to process a generic Ingress resource!  One must specify an implementation in the form of a provider.

I cannot get my Ingress resource translated without modifying it.

Here is one solution:

1. Revise the `ingress.class` annotation from "istio" to "nginx", and specify the `ingress-nginx` provider:

    ```shell
    ingress2gateway print --input-file ingress.yaml --providers ingress-nginx
    ```

2. The above still won't work.  For some reason I must also explicitly specify the namespace in the metadata section of my resource, or supply a `--all-namespaces` flag.  Only then will the tool do my bidding:

    ```shell
    ingress2gateway print \
      --input-file ingress.yaml \
      --providers ingress-nginx \
      --all-namespaces
    ```

In my opinion, the tool should, out of the box, be able to take a plain Ingress resource and translate it, without fuss.  If there are custom annotations supported by a specific implementation that I also wish the tool to be able to translate, then the notion of a provider plugin implementation is a logical way to address this.
