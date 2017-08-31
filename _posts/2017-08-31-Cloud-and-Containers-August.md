---
layout: default
title: Cloud and Containers - August
image: images/1500x500.jpg
link: /cloud-and-containers-august
permalink: /cloud-and-containers-august
excerpt: August had tremendous amount of container news. CNCF grew a lot in influence and I started thinking  that the only true challenger of AWS's hegemony are the CNCF projects. I'm also pleased to see Microsoft as a true innovator in the container space. And the race is on for Windows based container workloads.

--- 

## Cloud and Containers - August
{{ page.date | date: "%Y-%m-%d" }}

August had tremendous amount of container news. CNCF grew a lot in influence and I started thinking  that the only true challenger of AWS's hegemony are the CNCF projects. I'm also pleased to see Microsoft as a true innovator in the container space. And the race is on for Windows based container workloads.

### CNCF

It's the Cloud Native Computing Foundation [https://www.cncf.io/](https://www.cncf.io/), home of the major open source projects that shape the tech landscape these days. Think Kubernetes and Prometheus for the two prominent ones, and also worth knowing that both the Docker container runtime and the competeing rkt ("rocket") are [under its wings now](https://thenewstack.io/separate-votes-cncf-adopts-dockers-containerd-coreos-rkt/). 

Plus their projects have excellent communities. Imagine the process it takes to keep the thousand plus contributors of Kubernetes in sync. Not suprisingly, most of their stuff happening publicly on their Slack and Youtube channels. You could listen in their weekly SIG (Special Interest Group) meetings, track roadmaps and all if that's your cup of tea.

But back to the news now:
* AWS as the **last** major cloud provider joined CNCF. AWS fantastic and all, just keep using them as they are the best Cloud platform out there no question. But on the grand scale of things it's the world vs AWS these days, and the battle might have been already lost. Only community projects and standards can provide competing alternatives to AWS at this point and I'm happy that there is a candidate for doing that - CNCF. So I see this move as something they had to do, not that they wanted to. Nor they will be the key pushers of community projects like Kubernetes. https://www.cncf.io/blog/2017/08/09/cncf-welcome-amazon-web-services/
* Microsoft [also joined](https://www.cncf.io/announcement/2017/07/26/microsoft-joins-cloud-native-computing-foundation-platinum-member/){:target="_blank"} CNCF, but as you will see bellow they are trully pushing the space forward.

### Microsoft

Microsoft had to reinvent itself, and it's surely a fresh take what they are doing these days. Having an Ubuntu shell integrated in Windows, pushing Visual Studio Code are two nice examples of their new way of doing things. 

Plus unlike any other Cloud provider they feature all three leading container orchestrators (Kubernetes, Docker Swarm, Apache Mesos) in their [Azure Container Service](https://docs.microsoft.com/en-us/azure/container-service/){:target="_blank"}. Quite a feet comparing to AWS for example who only has one propreitary solution (ECS).

* Happened late July, but it's important to showcase the Azure Container Instances. While in all other systems I know you provision VMs to get extra capacity in your container platform, Azure launched a new paradigm, provision containers in a matter of seconds to get that extra juice. You get it in a matter of seconds not minutes - for a hefty price of course, but this is what I'm talking about when I mention "innovation" and "Microsoft" in one sentence.

![Azure Container Instances](images/aci-connector-k8s.gif)

* Another nice move from MS is that they [teamed up with Hashicorp](https://venturebeat.com/2017/08/17/hashicorp-and-microsoft-team-up-on-terraform-infrastructure-provisioning/){:target="_blank"} to bring Terraform as a first class tooling for Azure. I mean how cool is that?! I'm looking at you AWS and Cloud Formation. Again proprietary vs *de jure* standard.

### Windows containers

Yes, Windows is not under the Microsoft news, strange but you will see why. Windows based workloads are going to unlock the big money, the enterprise money, no wonder that many companies are trying to be the first ones to orchestrate Windows containers.

* While Docker Swarm is not always making into my Twitter feed, this one is a strong signal.
Docker created their CE and EE versions earlier this year, and now their [EE version allows companies](https://blog.docker.com/2017/08/docker-enterprise-edition-17-06/){:target="_blank"} to have Windows and Linux nodes in the same Docker Swarm cluster, so they can consolidate on containers, in one cluster. In EE of course. Their community edition of Swarm still don't feature simple access control and they simply [shortcut](https://docs.docker.com/engine/extend/plugins_authorization/#basic-principles){:target="_blank"} that with *"Anyone with the appropriate skills can develop an authorization plugin."* - thanks Docker. Role Based Access Control (RBAC) [is available in Kubernetes](https://laszlo.cloud/Why-access-control-is-key-for-a-secure-multi-tenant-Kubernetes-deployment){:target="_blank"} since April this year.
* Another company - that is not Microsoft - is jumping on Windows containers. [Red Hat](https://www.redhat.com/en/blog/supporting-windows-server-containers-red-hat-openshift){:target="_blank"} although it seems it will be only available once the upstream Kubernetes project solved the issue. 
Listening in on the Windows SIG Slack myself Windows nodes were planned in beta in September but they may not have the manpower to pull it off. But be assured in 6-9 months Windows containers will be all the rage.

In the meantime Docker EE and Azure being the two options to run hybrid workloads. Again Azure, as their ACS allows you today to launch Windows nodes in your Kubernetes cluster. I tried that myself, and it's trully as simple as [this page](https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-kubernetes-windows-walkthrough){:target="_blank"} suggests.

### Learning materials

And last but not least two awesome sources of knowledge to get more familiar with Kubernetes:

* Katakoda - [Learn Kubernetes using Interactive Browser-Based Scenarios](https://www.katacoda.com/courses/kubernetes){:target="_blank"}
* [TGI Kubernetes](https://www.youtube.com/watch?v=9YYeE-bMWv8){:target="_blank"} a weekly video stream. Wonderful way to pick up knowledge nuggets from the maker of Kubernetes

That was it for August. Onwards!

Laszlo
