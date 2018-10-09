---
layout:     post
title:      What is a Service Mesh?
subtitle:   Introduction to Service Mesh
date:       2019-04-02
author:     Mr.Humorous 🥘
header-img: img/microService/header.png
catalog: true
tags:
    - Microservice
---

A _service mesh_ is a configurable, low‑latency infrastructure layer designed to handle a high volume of network‑based interprocess communication among application infrastructure services using application programming interfaces (APIs). A service mesh ensures that communication among containerized and often ephemeral application infrastructure services is fast, reliable, and secure. The mesh provides critical capabilities including service discovery, load balancing, encryption, observability, traceability, authentication and authorization, and support for the circuit breaker pattern.

The service mesh is usually implemented by providing a proxy instance, called a _sidecar_, for each service instance. Sidecars handle interservice communications, monitoring, and security‑related concerns – indeed, anything that can be abstracted away from the individual services. This way, developers can handle development, support, and maintenance for the application code in the services; operations teams can maintain the service mesh and run the app.

Istio, backed by Google, IBM, and Lyft, is currently the best‑known service mesh architecture. Kubernetes, which was originally designed by Google, is currently the only container orchestration framework supported by Istio. Vendors are seeking to build commercial, supported versions of Istio. It will be interesting to see the value they can add to the open source project.

Istio is not the only option, and other service mesh implementations are also in development. The sidecar proxy pattern is most popular, as illustrated by projects from Buoyant, HashiCorp, Solo.io, and others. Alternative architectures exist as well: Netflix’s technology suite is one such approach where service mesh functionality is provided by application libraries (Ribbon, Hysterix, Eureka, Archaius), and platforms such as Azure Service Fabric embed service mesh‑like functionality into the application framework.

Service mesh comes with its own terminology for component services and functions:
- __Container orchestration framework.__ As more and more containers are added to an application’s infrastructure, a separate tool for monitoring and managing the set of containers – a container orchestration framework – becomes essential. Kubernetes seems to have cornered this market, with even its main competitors, Docker Storm and Mesosphere DC/OS, offering integration with Kubernetes as an alternative.
- __Services and instances (Kubernetes pods).__ An _instance_ is a single running copy of a microservice. Sometimes the instance is a single container; in Kubernetes, an instance is made up of a small group of interdependent containers (called a _pod_). Clients rarely access an instance or pod directly; rather they access a service, which is a set of identical instances or pods (_replicas_) that is scalable and fault‑tolerant.
- __Sidecar proxy.__ A _sidecar proxy_ runs alongside a single instance or pod. The purpose of the sidecar proxy is to route, or proxy, traffic to and from the container it runs alongside. The sidecar communicates with other sidecar proxies and is managed by the orchestration framework. Many service mesh implementations use a sidecar proxy to intercept and manage all ingress and egress traffic to the instance or pod.
- __Service discovery.__ When an instance needs to interact with a different service, it needs to find – discover – a healthy, available instance of the other service. Typically, the instance performs a DNS lookup for this purpose. The container orchestration framework keeps a list of instances that are ready to receive requests and provides the interface for DNS queries.
- __Load balancing.__ Most orchestration frameworks already provide Layer 4 (network) load balancing. A service mesh implements more sophisticated Layer 7 (application) load balancing, with richer algorithms and more powerful traffic management. Load‑balancing parameters can be modified via API, making it possible to orchestrate blue‑green or canary deployments.
- __Encryption.__ The service mesh can encrypt and decrypt requests and responses, removing that burden from each of the services. The service mesh can also improve performance by prioritizing the reuse of existing, persistent connections, which reduces the need for the computationally expensive creation of new ones. The most common implementation for encrypting traffic is mutual TLS (mTLS), where a public key infrastructure (PKI) generates and distributes certificates and keys for use by the sidecar proxies.
- __Authentication and authorization.__ The service mesh can authorize and authenticate requests made from both outside and within the app, sending only validated requests to instances.
- __Support for the circuit breaker pattern.__ The service mesh can support the circuit breaker pattern, which isolates unhealthy instances, then gradually brings them back into the healthy instance pool if warranted.

The part of a service mesh application that manages the network traffic between instances is called the _data plane_. Generating and deploying the configuration that controls the data plane’s behavior is done using a separate _control plane_. The control plane typically includes, or is designed to connect to, an API, a command‑line interface, and a graphical user interface for managing the app.
![Service Mesh Generic Topology Illustration](/img/microService/introToServiceMesh/serviceMeshGenericTopology.png)
_The control plane in a service mesh distributes configuration across the sidecar proxies in the data plane_

A common use case for service mesh architecture is solving very demanding operational problems when using containers and [microservices](https://www.nginx.com/blog/introduction-to-microservices/). Pioneers in microservices include companies like Lyft, Netflix, and Twitter, which each provide robust services to millions of users worldwide, hour in and hour out. (See our in‑depth description of some of the [architectural challenges facing Netflix](https://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/).) For less demanding application needs, simpler architectures are likely to suffice.

Service mesh architectures are not ever likely to be the answer to all application operations and delivery problems. Architects and developers have a great many tools, only one of which is a hammer, and must address a great many types of problems, only one of which is a nail. The NGINX [Microservices Reference Architecture](https://www.nginx.com/blog/introducing-the-nginx-microservices-reference-architecture/), for instance, includes several different models that give a continuum of approaches to using microservices to solve problems.

The elements that come together in a service mesh architecture – such as NGINX, containers, Kubernetes, and microservices as an architectural approach – can be, and are, used productively in non‑service mesh implementations. For example, Istio was developed as a complete service mesh architecture, but its modular design means developers can pick and choose the component technologies they need. With this in mind, it’s worth developing a solid understanding of service mesh concepts, even if you’re not sure if and when you’ll fully implement a service mesh application.
