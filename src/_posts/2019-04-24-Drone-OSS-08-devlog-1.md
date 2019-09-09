---
layout: drone
title: DroneOSS08 Devlog &#x23;1
image: images/1500x500.jpg
link: /drone-oss-08-devlog-1
permalink: /drone-oss-08-devlog-1
excerpt: These are tricks I gathered over the years using Drone.io environment variables. All testable with `drone exec`.


---

## DroneOSS08 Devlog #1

{{ page.date | date: "%Y-%m-%d" }}

This is the first Devlog entry for the DroneOSS08 project. 

DroneOSS08 is hosted on [Github](https://github.com/laszlocph/drone-oss-08/){:target="\_blank"} and it's a fork of Drone.io version 0.8

This post is a brain dump. To track progress, to not fool myself six months in the project and to fuel some sentimentalism a year from now.

### Kubernetes integration

[https://github.com/laszlocph/drone-oss-08/compare/kube?expand=1](https://github.com/laszlocph/drone-oss-08/compare/kube?expand=1){:target="\_blank"}

So far so good, but this first part was the easy one. I could reuse the Apache 2.0 licensed code written by @metalmatze and then integrate it in the server. It takes only a few environment variables to enable the Kubernetes engine:

```
- DRONE_KUBERNETES=true
- DRONE_KUBERNETES_NAMESPACE=default
- DRONE_KUBERNETES_STORAGECLASS=example-nfs
- DRONE_KUBERNETES_VOLUME_SIZE=100Mi
- KUBECONFIG=/root/.kube/config
```

I could also test the RWX volume support through storage classes. Did a bunch of measurements as well. NFS is the simplest RWX volume to use but it has bad reputation.

Speed was supposed to be an issue but my first benchmarks were not terrible. A complex `bundle install` was taking 2 minutes, same as on my laptop. Drone Cloud was my benchmark which resulted in 2 minutes too, but one time 1:20 only. I have to do more tests, but NFS is not terrible.

Applied some opinions as well, all Drone pods will go into the same Kubernetes namespace. I believe I can solve the naming collision of services.

### Adoption

I'm happy about adoption. I got green light from my client to install DroneOSS08 for them. Agreed that I have 6 months until the question will be raised again whether purchase Drone 1.0 licenses or stick with the fork. 

I guess their reasoning was that 0.8 works well right now and there is no immediate reason to update to 1.0. This of course may change as 1.0 diverge. But I have some time and a large install base to test my fork.

### Risks

The codebase is old. To be more precise the dependencies are many years old and were not kept up to date. I'm afraid of regressions popping up due to updating them.

Like it happened already when I pulled in the Kubernetes Go client. Bunch of things broke, and a test too. I fear this single test failure is just the tip of the iceberg.


Onwards!