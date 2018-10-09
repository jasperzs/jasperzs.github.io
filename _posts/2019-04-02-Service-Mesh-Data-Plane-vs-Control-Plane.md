---
layout:     post
title:      Service Mesh Data Plane vs. Control Plane
subtitle:   Understanding Data Plane and Control Plane
date:       2019-04-02
author:     Mr.Humorous 🥘
header-img: img/microService/header.png
catalog: true
tags:
    - Microservice
---

## 1. What is a service mesh, really?
_Figure 1_ illustrates the service mesh concept at its most basic level. There are four service clusters (A-D). Each service instance is colocated with a sidecar network proxy. All network traffic (HTTP, REST, gRPC, Redis, etc.) from an individual service instance flows via its local sidecar proxy to the appropriate destination. Thus, _the service instance is not aware of the network at large and only knows about its local proxy_. In effect, the distributed system network has been abstracted away from the service programmer.
![Service Mesh Overview](/img/microService/serviceMeshDataPlaneVSControlPlane/figure1.png)
Figure 1: Service mesh overview

## 2. The Data Plane
In a service mesh, the sidecar proxy performs the following tasks:
- __Service discovery__: What are all of the upstream/backend service instances that are available?
- __Health checking__: Are the upstream service instances returned by service discovery healthy and ready to accept network traffic? This may include both active (e.g., out-of-band pings to a `/healthcheck` endpoint) and passive (e.g., using 3 consecutive 5xx as an indication of an unhealthy state) health checking.
- __Routing__: Given a REST request for `/foo` from the local service instance, to which upstream service cluster should the request be sent?
- __Load balancing__: Once an upstream service cluster has been selected during routing, to which upstream service instance should the request be sent? With what timeout? With what circuit breaking settings? If the request fails should it be retried?
- __Authentication and Authorization__: For incoming requests, can the caller be cryptographically attested using mTLS or some other mechanism? If attested, is the caller allowed to invoke the requested endpoint or should an unauthenticated response be returned?
- __Observability__: For each request, detailed statistics, logging, and distributed tracing data should be generated so that operators can understand distributed traffic flow and debug problems as they occur.

All of the previous items are the responsibility of the service mesh _data plane_. In effect, the sidecar proxy is the data plane. Said another way, the data plane is responsible for conditionally translating, forwarding, and observing every network packet that flows to and from a service instance.

## 3. The Control Plane
The network abstraction that the sidecar proxy data plane provides is magical. However, how does the proxy actually know to route `/foo` to service B? How is the service discovery data that the proxy queries populated? How are the load balancing, timeout, circuit breaking, etc. settings specified? How are deploys accomplished using blue/green or gradual traffic shifting semantics? Who configures systemwide authentication and authorization settings?

All of the above items are the responsibility of the service mesh _control plane_. _The control plane takes a set of isolated stateless sidecar proxies and turns them into a distributed system._

The reason that I think many technologists find the split concepts of data plane and control plane confusing is that for most people the data plane is familiar while the control plane is foreign. We’ve been around physical network routers and switches for a long time. We understand that packets/requests need to go from point A to point B and that we can use hardware and software to make that happen. The new breed of software proxies are just really fancy versions of tools we have been using for a long time.
![Human Control Plane](/img/microService/serviceMeshDataPlaneVSControlPlane/figure2.png)
Figure 2: Human Control Plane

However, we have also been using control planes for a long time, though most network operators might not associate that portion of the system with a piece of technology. There reason for this is simple — most control planes in use today are… _us_.

__Figure 2__ shows what I call the “human control plane.” In this type of deployment (which is still extremely common), a (likely grumpy) human operator crafts static configurations — potentially with the aid of some scripting tools — and deploys them using some type of bespoke process to all of the proxies. The proxies then consume the configuration and proceed with data plane processing using the updated settings.

![Advanced Service Mesh Control Plane](/img/microService/serviceMeshDataPlaneVSControlPlane/figure3.png)
Figure 3: Advanced Service Mesh Control Plane

__Figure 3__ shows an “advanced” service mesh control plane. It is composed of the following pieces:
- __The human__: There is still a (hopefully less grumpy) human in the loop making high level decisions about the overall system.
- __Control plane UI__: The human interacts with some type of UI to control the system. This might be a web portal, a CLI, or some other interface. Through the UI, the operator has access to global system configuration settings such as deploy control (blue/green and/or traffic shifting), authentication and authorization settings, route table specification (e.g., when service A requests `/foo` what happens), and load balancer settings (e.g., timeouts, retries, circuit breakers, etc.).
- __Workload scheduler__: Services are run on an infrastructure via some type of scheduling system (e.g., Kubernetes or Nomad). The scheduler is responsible for bootstrapping a service along with its sidecar proxy.
- __Service discovery__: As the scheduler starts and stops service instances it reports liveness state into a service discovery system.
- __Sidecar proxy configuration APIs__: The sidecar proxies dynamically fetch state from various system components in an eventually consistent way without operator involvement. The entire system composed of all currently running service instances and sidecar proxies eventually converge. Envoy’s [universal data plane API](https://medium.com/@mattklein123/the-universal-data-plane-api-d15cec7a) is one such example of how this works in practice.

> _Ultimately, the goal of a control plane is to set policy that will eventually be enacted by the data plane_. More advanced control planes will abstract more of the system from the operator and require less handholding (assuming they are working correctly!).

## 4. Data Plane vs. Control Plane Summary
- __Service mesh data plane__: Touches every packet/request in the system. Responsible for service discovery, health checking, routing, load balancing, authentication/authorization, and observability.
- __Service mesh control plane__: Provides policy and configuration for all of the running data planes in the mesh. Does not touch any packets/requests in the system. The control plane turns all of the data planes into a distributed system.

## 5. Current project landscape
With the above explanation out of the way, let’s take a look at the current service mesh landscape.
- Data planes: [Linkerd](https://linkerd.io/), [NGINX](https://www.nginx.com/), [HAProxy](https://www.haproxy.com/), [Envoy](https://envoyproxy.github.io/), [Traefik](https://traefik.io/)
- Control planes: [Istio](https://istio.io/), [Nelson](https://verizon.github.io/nelson/), [SmartStack](https://github.com/airbnb/synapse)

Instead of doing an in-depth analysis of each solution above, I’m going to briefly touch on some of the points that I think are causing the majority of the ecosystem confusion right now.

Linkerd was one of the first service mesh data plane proxies on the scene in early 2016 and has done a fantastic job of increasing awareness and excitement around the service mesh design pattern. Envoy followed about 6 months later (though was in production at Lyft since late 2015). Linkerd and Envoy are the two projects that are most commonly mentioned when discussing “service meshes.”

Istio was announced May, 2017. The project goals of Istio look very much like the advanced control plane illustrated in __figure 3__. The default proxy of Istio is Envoy. Thus, _Istio is the control plane and Envoy is the data plane_. In a short time, Istio has garnered a lot of excitement, and other data planes have begun integrations as a replacement for Envoy (both Linkerd and NGINX have demonstrated Istio integration). The fact that it’s possible for a single control plane to use different data planes means that the control plane and data plane are not necessarily tightly coupled. An API such as Envoy’s [universal data plane API](https://medium.com/@mattklein123/the-universal-data-plane-api-d15cec7a) can form a bridge between the two pieces of the system.

Nelson and SmartStack help further illustrate the control plane vs. data plane divide. Nelson uses Envoy as its proxy and builds a robust service mesh control plane around the HashiCorp stack (i.e. Nomad, etc.). SmartStack was perhaps the first of the new wave of service meshes. SmartStack forms a control plane around HAProxy or NGINX, further demonstrating that it’s possible to decouple the service mesh control plane and the data plane.

The service mesh microservice networking space is getting a lot of attention right now (rightly so!) with more projects and vendors entering all the time. Over the next several years, we will see a lot of innovation in both data planes and control planes, and further intermixing of the various components. The ultimate result should be microservice networking that is more transparent and magical to the (hopefully less and less grumpy) operator.

## 6. Key takeaways
- A service mesh is composed of two disparate pieces: the data plane and the control plane. Both are required. Without both the system will not work.
- Everyone is familiar with the control plane — albeit the control plane might be you!
- All of the data planes compete with each other on features, performance, configurability, and extensibility.
- All of the control planes compete with each other on features, configurability, extensibility, and usability.
- A single control plane may contain the right abstractions and APIs such that multiple data planes can be used.
