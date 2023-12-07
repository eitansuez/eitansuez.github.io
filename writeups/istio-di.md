2023.09.07

# Rediscovering past patterns in modern systems architectures

We can make an analogy, a comparison between the actors in a monolithic application and the actors in a cloud-native application, as follows:

1. Workloads in the cloud-native world are akin to objects in a monolith.
2. Communication between workloads in a Kubernetes cluster are akin to method calls between objects inside a monolith.

That analogy can be taken further, for example, a stack trace in a monolithic application is akin to a distributed trace in a cloud-native application.

Of course it's important to remember that workloads are nothing like objects, and network calls are nothing like method calls.  Each is its own animal, so to speak, with its own set of characteristics, and issues.

But it's useful to think of a cloud-native application as some kind of "bigger box" that can allow systems to run at scale, by magnifying each "object" in the monolith into a new creature, the microservice.

In a similar fashion, Istio in a cloud-native app is like dependency injection (DI) in the monolith.

If Istio is like dependency injection (DI), then let's compare how DI works both in Istio and in a monolith with say, one of the most popular DI frameworks of all, the [Spring framework](https://spring.io/).

Let us begin with the monolith.

In Spring, you define an object, aka a "bean."
The class definition has a constructor with arguments.
It is the job of the application context to furnish each argument (thus freeing the class of the burden of knowing too much about how its environment is constructed).
This list of arguments is basically the information the application context needs in order to construct the object.
It's a convention.

From this information, the application context builds a dependency tree.  And so it knows which objects to construct first.  But I digress..

The important thing to note is that an object is only given references to objects it declares it needs.

Even though the application context may maintain references to hundreds of objects, each object is given references only to its collaborators.

For example, we can imagine a `ProductPage` bean, as follows:

```java
@Component
class ProductPage {
  // We used to have to annotate these with @Autowired, but that is no longer necessary.
  private final DetailsService detailsService;
  private final ReviewService reviewsService;

  public ProductPage(DetailsService ds, ReviewsService rs) {
    this.detailsService = ds;
    this.reviewsService = rs;
  }

  public ProductInfo getProductInfo(int productId) {
    ProductDetails details = detailsService.getProductDetails(productId);
    ProductReviews reviews = reviewsService.getReviews(productId);
    return new ProductInfo(details, reviews);
  }
  ..
}
```

In this application (this system of interacting objects), there may be other services, such as a `ratings` service, but the `ProductPage` component (or bean) does not need to be given a reference to it, because it never interacts with it directly.

Conversely, if Spring's `applicationContext` had to give each object a reference to every other object, it would create an untenable geometric growth of object references, most of these references being literally useless.

Istio has an analog to that constructor that tells the application context "I only need references to these two objects, err.. I mean services."
It's the [`Sidecar` resource](https://istio.io/latest/docs/reference/config/networking/sidecar/).  Except that most of the time, when we do demonstrations, or setup a simple application with only a few services, we don't bother to define these resources.
It's simpler to just let Istio do its default thing, which is to autowire (choice of words here is intended) every service into every other service.

_This has been a source of criticism of Istio_, stating that Istio does not scale beyond a few services.
In the monolith the constructor provides a formal means of declaring dependencies.
Moreover, a developer can hardly publish their objects without declaring their constructor.

Not so with Istio.  In the cloud-native world, this boils down to having discipline.

Here is the `Sidecar` resource we would create in Istio for the `productpage` service (presumably published in the `default` namespace):

```yaml
---
apiVersion: networking.istio.io/v1beta1
kind: Sidecar
metadata:
  name: productpage-sidecar
  namespace: default
spec:
  workloadSelector:
    labels:
      app: productpage
  egress:
  - hosts:
    - "./reviews.default.svc.cluster.local"
    - "./details.default.svc.cluster.local"
    - "istio-system/*"
```

Et voila.  Istio will now inject only references to `reviews` and `details` services into `productpage` workloads.

Of course we are in a cloud-native world, meaning we can scale each service horizontally, upgrade a service to a new version without incurring downtime, and Istio will make sure to keep the endpoint lists up-to-date for all clients such as the `productpage` workloads, while Envoy will take care of load-balancing requests to upstream services (aka "clusters"), and applying whatever network (and other) policies you define.

**By making it mandatory for services to publish their dependencies via `Sidecar` resources, we get a scalable Istio.**

## Thinking a step further

Let's refine our analogy between the monolith and cloud-native worlds.

Roughly, I see a loose mapping between:

| Monolith<br>Java  | Cloud-Native<br>Kubernetes |
| --------- | ------------ |
| Class     | Deployment   |
| Interface | Service      |

The reasons Istio defines the `Sidecar` resource in the first place is because a `Deployment` specification does not include this important extra information we need to answer the question:  _What collaborating services does this deployment talk to?_

If it did, then Istio could just leverage this information and Dependency Injection would be performant out of the box.

I suppose the moral of the story here is that it's worthwhile comparing concepts and constructs that exist in the older and more mature world of monoliths, as an aid to designing our bigger box:  Kubernetes.

## Addendum

Besides Dependency Injection, another potential, and valuable use of this extra metadata about services is automatic configuration of authorization policies.
If we know _a priori_ what services talk to each service, we could generate authorization policies that allow *only those services* to reach them.

Assuming this information is specified using `Sidecar` resources, we could generate authorization policies for service owners to review and apply.  For example:  Only the `productpage` service is allowed to call the `details` service:

```yaml
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allowed-details-clients
  namespace: default
spec:
  selector:
    matchLabels:
        app: details
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
```
