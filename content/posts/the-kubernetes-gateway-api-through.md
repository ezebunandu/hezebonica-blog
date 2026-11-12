+++
title = "The Kubernetes Gateway API through beginner’s eyes"
date = 2025-05-05T05:47:43Z
draft = false
slug = "the-kubernetes-gateway-api-through"
description = "Kubernetes Adventures in Color"
canonicalURL = "https://ezebunandu.substack.com/p/the-kubernetes-gateway-api-through"
originalURL = "https://ezebunandu.substack.com/p/the-kubernetes-gateway-api-through"
tags = ["substack-migration"]
+++

> Kubernetes Adventures in Color

## Kubernetes Adventures in Color

I have recently been building fun little home automation-style projects with the Phillips Hue lights in my office space. I have a microservice, [hueLightScheduler,](https://github.com/ezebunandu/gohome/tree/main/hueLightScheduler) that turns one set of lights on the right wall of the office (pictured below) off every night at 10:30 PM and then back on in the morning at 5:30 AM. This microservice is written in Go and runs in a Pod within my Kubernetes homelab. One problem with this setup, though, is that after powering the lights back on, they are no longer in the **colorloop mode** that I prefer to have them in. In colorloop mode, the lights continually toggle between various colors. This dance of colors helps me cope with cold, cloudy Canadian winter days, or so I like to think.

Thanks for reading x Through Beginner's Eyes! Subscribe for free to receive new posts and support my work.

{{< figure src="/images/substack/the-kubernetes-gateway-api-through/01-346463b756.jpeg" caption="Some of the Phillips Hue lights in my office" >}}

Well then, what to do to get the dance of colors back? Enter a second microservice, [hueColorLooper](https://github.com/ezebunandu/gohome/tree/main/hueColorLooper) that accepts http requests at a designated endpoint, and puts the lights into colorloop mode. My initial implementation of the project used a cron job on an Ubuntu VM outside of the cluster to make the required http requests using curl daily at 5:35AM. A hacky solution, yes, but it worked!

Unlike the hueLightScheduler service, which I expose outside of my cluster using a Kubernetes **Ingress** resource, I decided to experiment with exposing hueColorLooper using the newer Kubernetes **Gateway API** , which I have been learning about. Ingress and Gateway APIs allow you to route external traffic from outside of the cluster to Pods in the cluster. If you’re running a web service, you will likely want to allow traffic from outside the cluster go get to the Pods running the web service, and this is where Ingress or the Gateway API will come into play. Before I walk you through how this project made use of the Gateway API, a little background on the Kubernetes networking model might be useful.

## Kubernetes Networking in a Couple of Paragraphs — Pod-to-Pod Communication, Service, and Ingress

*The following explanation on the Kubernetes networking model glosses over a lot of important details. The point of the explanation is to establish why one would need abstractions like Ingress or Gateway API.*

When you deploy containerized applications on Kubernetes, you wrap the containers in an abstraction known as a **Pod**. Pods are the smallest unit of scheduling in Kubernetes and encapsulate one or more containers. From a networking point of view, all containers within a pod share the same network namespace — meaning they can reach each other on localhost. By default, all Pods in the cluster can also communicate with every other Pod through an [overlay network.](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

One of the benefits of an orchestration system like Kubernetes is self-healing behavior for your applications. If a containers crashes, Kubernetes will automatically “restart” it for you. An implication of this is that pods are always getting created and destroyed (i.e. they are ephemeral), so their IP addresses are unpredictable. Apps deployed in Pods cannot rely on Pod IP addresses as a reliable way to communicate with other Pods in the cluster. Enter the Kubernetes **Service** object. According to the official Kubernetes documentation, **a Service is an abstraction to help you expose a group of Pods over a network**. A service provides a predictable DNS name, IP address, and port for an application running in one or more Pods. The service will load balance when there are multiple instances (replicas) of the application container defined by a **Deployment**.

### Kubernetes Service Types

As we have established, containerized applications in Kubernetes are wrapped in Pods and Pods must be fronted by a Service object to provide predictable networking and load-balancing between multiple replicas. Services typically come in one of the following three types: **ClusterIP**, **NodePort** , or **LoadBalancer**

The **ClusterIP** service is the default service type and exposes the underlying application on a cluster-internal IP address. By using a ClusterIP service to front an application, it gets a name and IP that is programmed into the internal network fabric and is **only accessible from inside the cluster**. The code snippet below is the YAML manifest for the ClusterIP service deployed as part of the hueColorLooper application

```
apiVersion: v1
kind: Service
metadata:
    name: colorlooper-service
    namespace: gohome
    labels:
        app: colorlooper
spec:
    selector:
        app: colorlooper
    ports:
        - protocol: TCP
          port: 3005
          targetPort: http
          name: http
    type: ClusterIP
```

We are declaring a ClusterIP type service named **colorlooper-service** that targets pods with the label **app=colorlooper**. This makes it so that from anywhere within the cluster, we can communicate with the application fronted by the service using the provided CLUSTER-IP (10.43.68.61 in my case, as shown in the screenshot below)

{{< figure src="/images/substack/the-kubernetes-gateway-api-through/02-8bc364a05e.png" >}}

For our use case — where we want to expose the application so that traffic from the Ubuntu VM that is not part of the cluster can reach it — a ClusterIP alone is not sufficient.

**NodePort** services provide one solution to allow applications to be reachable from outside the cluster. NodePorts build on top of ClusterIP Services by adding a dedicated port on every cluster node that external clients can use. By making a request to the IP address of any node that is part of the cluster on the provisioned port, the external client can communicate with the application being fronted. The problem with this is that NodePorts only use high-numbered ports and clients need to know the names or IPs of nodes, as well as whether nodes are healthy.

A **LoadBalancer** service is the easiest way of exposing services to external clients. They put a load balancer (cloud or virtual) in front of the application. Requests to the load balancer’s IP address get routed to the underlying application. The problem with the LoadBalancer service is that it requires a one-to-one mapping between internal services and the load balancer. [This can become very expensive at scale.](https://aws.amazon.com/elasticloadbalancing/pricing/) Imagine you have more than a thousand microservices and you have to provision a dedicated cloud load balancer for each one of them. This is where you might need the Kubernetes **Ingress** resource to save you from load balancer induced bankruptcy.

### Ingress

An Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the ingress resource. You can think of an ingress as an abstraction that allows re-using the same load balancer for multiple http/https services, using host-based and path-based routing to map connections to the different services. The Ingress model translates ingress rules into configurations that a proxy can understand and execute.

{{< figure src="/images/substack/the-kubernetes-gateway-api-through/03-87ad27d081.png" caption="Illustration of an ingress resource, from the Kubernetes documentation at [docs.kubernetes.io](http://docs.kubernetes.io)" >}}

To utilize an ingress, you must first have an **Ingress Controller** . The Ingress Controller is responsible for ultimately applying routing configurations described via ingress resources to some data plane (e.g. a reverse proxy, a load balancer, etc.). The Ingress resource then defines routing rules that govern how external requests get directed to the service being fronted. Rules can include hostnames, paths, and TLS configurations used to map traffic. As there is no native Ingress controller built into Kubernetes, you must first install one of many optional implementations, such as the **Nginx Ingress Controller**. Once an Ingress Controller is installed, you can then deploy ingress resources to route traffic to the underlying application/service. An example manifest for the Ingress resource used by the **LightScheduler** microservice.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: lightscheduler-ingress
    namespace: gohome
    annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
        nginx.ingress.kubernetes.io/ssl-redirect: "true"
        nginx.ingress.kubernetes.io/proxy-body-size: "0"
spec:
    ingressClassName: nginx
    rules:
        - host: lightscheduler.lab
          http:
              paths:
                  - path: /
                    pathType: Prefix
                    backend:
                        service:
                            name: lightscheduler-service
                            port:
                                number: 8100
```

The YAML manifest configures an ingress to forward traffic to the **lightscheduler-service** on port 8100, when http requests with the host set to **lightscheduler.lab** are received. The request must be to the **root (**`/)` **path**. Notice that in the **metadata** block for the ingress resource, we have included a number of annotations, such as **http://nginx.ingress.kubernetes.io/rewrite-target:/**. These annotations are used to define controller-specific behaviour. Because they are controller specific, Kubernetes doesn’t really know about them and cannot validate their correctness at the time of deployment, so any errors in the annotations may lead to problematic deployments. For **[the nginx ingress controller, there are 104 possible annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)** that control all sorts of behavior of the nginx proxy, including ssl redirects, enabling Opentelemetry and various WAF rules.

Besides the non-trivial problem of a complicated system of implementation-specific annotations, ingress can only filter traffic at layer 7 (HTTP/HTTPS). If you wanted to route traffic to any protocol other than HTTP, you are basically out of luck with an Ingress. Fear not though — the Gateway API can bail you out!

### Gateway API: The New Way to Ingress

{{< figure src="/images/substack/the-kubernetes-gateway-api-through/04-99135c944f.png" caption="The gateway api project logo, from https://gateway-api.sigs.k8s.io" >}}

First introduced at KubeCon 2019 as the **Ingress v2** , the Gateway API was billed at ideation as the next generation of Ingress, load balancing, and service mesh API in Kubernetes. The Gateway API v1.0 was released in November 2023 and sought to address **some of the shortcomings of the Ingress** API that has become obvious over the years, including:

- **Limited protocol support**: The ingress only operates at layer 7, being specifically optimized for http and https
- **Limitations in the core specification**: The core specification lacks support for advanced traffic management such as canary and blue-green deployments
- **Lack of portability**: extending the core functionality required vendor-specific implementations configured using annotations. These annotations can pile up, creating a tangled web that makes portability difficult.

#### Features & Design Principles

The Gateway API has the following design principles:

- Supports both L4 (transport layer) and L7 (application layer) protocols, including TCP, UDP, HTTP, and gRPC
- Has a role-oriented design geared towards 3 different personas (more details below)
- Enables routing customization, including the ability to route traffic based on arbitrary header fields as well as paths and hosts
- Is Portable: unlike ingress definitions which rely on vendor specifics configured by means of annotations, the Gateway API establishes a common standard that works across all compliant implementations, hence removing the need for sprawling annotations.

The gateway API emphasizes expressiveness and extensibility as core design principles. It introduced a modular structure with distinct roles — infrastructure provider, cluster operator, and app developer. The Gateway API also introduced new controller objects like **GatewayClass** for defining controller capabilities, **Gateway** for instantiating network gateways with those capabilities, and **HTTPRoute** for declaring HTTP routes.

{{< figure src="/images/substack/the-kubernetes-gateway-api-through/05-2b95b59719.png" caption="Role-oriented architecture of the Gateway API, courtesy of https://gateway-api.sigs.k8s.io/" >}}

The infrastructure provider, e.g. AWS, Azure, or GCP makes a GatewayClass resource available to be deployed in clusters hosted on their Infrastructure. For example, for deployments in Amazon EKS, AWS has the [AWS Gateway controller](https://www.gateway-api-controller.eks.aws.dev/dev/concepts/concepts/) that can be used with the amazon-vpc-lattice GatewayClass. Cluster operators will then create a Gateway resource based on this GatewayClass that can be used across all namespaces in the cluster. To enable specific routing rules for their applications, application developers will create a HTTPRoute, TLSRoute, etc. This role-oriented design seeks to strike a balance between distributed flexibility and centralized control, allowing shared infrastructure to be used by many different and non-coordinating teams, all bound by the policies and constraints set by the cluster operators.

### Putting it all together — Exposing the Service with a Gateway API Route

To utilize the Gateway resource to expose the **hueColorLooper** service, I first needed to take on the persona of an infrastructure provider. I installed metalLB (a bare metal load balancer much loved by the kubernetes homelab community) along with the **[NGINX Gateway Fabric](https://docs.nginx.com/nginx-gateway-fabric/)** and associated Custom Resource Definitions (CRDs). The NGINX Gateway Fabric is NGINX’s implementation of the Gateway API. The installation steps for the Gateway Fabric and CRDs is documented in the appendix below.

With the Gateway Fabric in place, I then created a Gateway resource referencing the GatewayClass installed with the Gateway Fabric using the YAML snippet below

```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
	    namespace:
		    from: All
```

This Gateway listens on port 80 for http traffic in all namespaces. This means routing rules for service in all namespaces within my cluster can reference this Gateway. Finally, I configured an **HTTPRoute** rule to control traffic to the HueColorLooper service using the YAML manifest below

```
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
    name: colorlooper-route
    namespace: gohome
spec:
    parentRefs:
        - name: nginx-gateway
          namespace: nginx-gateway
    hostnames:
        - colorlooper.lab
    rules:
        - matches:
              - path:
                    type: PathPrefix
                    value: /colorloop
          backendRefs:
              - name: colorlooper-service
                port: 3005
```

This rule routes http traffic with the hostname set to **colorlooper.lab** to the **/colorloop** endpoint and forward that traffic to the colorlooper-service backend listening on port 3005.

In the initial implementation, the cronjob on the external VM simply made a request to this endpoint on a schedule.

{{< figure src="/images/substack/the-kubernetes-gateway-api-through/06-61f92537f2.png" >}}

### So, What Was the Point, Sam?

If you’ve made it this far but are left scratching your head, wondering why we even need Kubernetes to control a stupid lightbulb and why we are f\*\*king around with Ingresses and Gateway APIs where there could be simpler solutions, I’m afraid to tell you that you’re absolutely right.

The Phillips Hue app on my phone has an **automations** feature that would allow me to create on/off schedules and such for the lightbulbs, but where’s the fun in that? Having decided to find ways to torture kubernetes until it confesses to everything, it felt right to experiment with Gateway API to get a sense of how it can be deployed and used for traffic routing to expose services.

In the end, I switched from using the cronjob on the VM external to the cluster to using a Kubernetes **CronJob** resource inside of the cluster, which made the need to expose the service externally somewhat unnecessary. Regardless, I had a ton of fun learning about the Gateway API in particular and Kubernetes networking in general and I can say for a fact that I know a little more about these concepts that before — and that, my friends, is the whole point of this enterprise!

{{< figure src="/images/substack/the-kubernetes-gateway-api-through/07-923abd7320.png" >}}

### References

- [The initial proposal for the Gateway API](https://www.youtube.com/watch?v=Ne9UJL6irXY)
- [The Gateway API docs](https://gateway-api.sigs.k8s.io/)
- <https://opensource.googleblog.com/2023/11/the-story-of-gateway-api.html>
- <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>
- [NGINX Gateway Fabric documentation](https://docs.nginx.com/nginx-gateway-fabric/overview/gateway-architecture/)
- [KubeCon NA 2024 on YouTube: Understanding Kubernetes Networking in 30 Minutes](https://www.youtube.com/watch?v=Mj04QOqAaJ8&t=30s)
- [Nginx controller Annotations Guide](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
- [Nginx blog: Moving from Ingress Controller to the Gateway API](https://blog.nginx.org/blog/kubernetes-networking-ingress-controller-to-gateway-api)
- [Kong blog: Gateway API vs. Ingress: The Future of Kubernetes Networking](https://konghq.com/blog/engineering/gateway-api-vs-ingress)
- [The Kubernetes Podcast from Google: Kubernetes Ingress & Gateway API Updates, with Lior Lieberman](https://pca.st/54yiyhri)
- [The New Stack Podcast: How the Kubernetes Gateway API Beats Network Ingress](https://pca.st/od9gvcdt)
- [DevOps and Docker Talk Podcast: Gateway API](https://pca.st/a6b91ddx)
- The Kubernetes Book, by Nigel Poulton

Thanks for reading x Through Beginner's Eyes! Subscribe for free to receive new posts and support my work.
