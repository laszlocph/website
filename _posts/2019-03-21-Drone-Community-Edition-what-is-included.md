---
layout: drone
title: Drone Community Edition - what is included?
image: images/1500x500.jpg
link: /drone-community-edition-what-is-included
permalink: /drone-community-edition-what-is-included
excerpt: The licensing of Drone 1.0 left a few questions open. In this article I capture what I know about the Drone Community Edition and my experience building the opensource version.

---

## Drone Community Edition - what is included?

{{ page.date | date: "%Y-%m-%d" }}

I've been using Drone 0.8 extensively for 18 months now and enjoyed great success with it.

Drone 1.0 is tagged now on Github and there are a few licensing changes that are worth noting. 

>Warning. I'm no expert on OSS licensing, consult with your legal advisor before you do things based on information published in this blog post.

### The licensing landscape is rather confusing

There is information scattered in source code, the Discourse forum and various FAQs and at times they hold conflicting information. 

There is a recent [Discourse thread](https://discourse.drone.io/t/drone-community-edition/3938){:target="\_blank"} on it where Brad cleared up some of my concerns.

<br>
![Drone.io licensing](images/drone-licensing.png) 

### Source code trumps everything in licensing questions

Brad was clear on this one. The source code has the final say of what is part of the drone.io Community Edition.

Since Docker Hub holds the enterprise version going forward, the `drone/drone:1` will have resource limits or it may raise licensing questions that I have no intent to figure out.

Instead I build the code from source to get the pure OSS version: `go build --tags oss github.com/drone/drone/cmd/drone-server`

The command results in a `drone-server` binary that I can start after setting the right environment variables:

```
DRONE_GITHUB_SERVER=https://github.com \
DRONE_GITHUB_CLIENT_ID=xxx \
DRONE_GITHUB_CLIENT_SECRET=xxx \
DRONE_SERVER_HOST=mydomain.com \
DRONE_SERVER_PROTO=http \
DRONE_USER_CREATE=username:yourgithubusername,admin:true \
DRONE_USER_FILTER=yourgithubusername \
DRONE_LOGS_DEBUG=true \
./drone-server
```

### There are no agents in the Community Edition

Surprisingly agents are not part of the Community Edition. 

While the server node can run builds and Drone works in a single node setup, scaling requires the purchase and Enterprise license (or fit in the low volume Drone usage to use the Enterprise version for free).


TI tried building the `drone-agent` binary from source and could confirm that is not part of the OSS license.

```
➜  drone git:(master) ✗ go build --tags oss github.com/drone/drone/cmd/drone-agent

can't load package: package github.com/drone/drone/cmd/drone-agent: build constraints exclude all Go files in /home/laszlo/projects/drone/cmd/drone-agent
```

Luckilly the `github.com/drone/drone/cmd/drone-agent` package is only a thin layer on top of the runner architecture of Drone. The runner source code is part of the Community Edition which we saw earlier when the single node setup ran builds.

### The Kubernetes integration is not part of the Community Edition

The same is true about the Kubernetes native builds. 

Only a ["noop" implementation](https://github.com/drone/drone/blob/master/scheduler/kube/kube_oss.go){:target="\_blank"} is tagged with `// +build oss` thus when you build with `go build --tags oss` all you get is pending builds when you try enabling the Kubernetes scheduler.

### What is the Drone Community Edition is good for then?

Smaller agentless single node setups where the price of running a node 24/7 is not prohibiting.

Price sensitivity varies a lot, some may run a 16 core node and don't mind the price, but the Community Edition does not scale infinitely.

At least not today. Drone is famous for the clean code and interfaces, I wouldn't be surprised if community maintained schedulers pop up.

Onwards!