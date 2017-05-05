---
layout: default
---

# Roadmap for Kubernetes projects

In this article I document a pragmatic approach how to carry out a Kubernetes project.

I've been following this roadmap with two of my clients, and although we haven't got to the later stages yet, it feels right to approach a new technology like Kubernetes this way.

My main goals with the approach is to reveal problems early on and to maximize learning across the organization. 

Given it's a very hot technology with a steep learning curve, I try to balance the hype with solid pace of deliveries. 
This helps getting through the setbacks what otherwise would be devastating for any pilot project.
I follow an aggressive schedule and try not getting romantic with certain technologies, I use what works.

By the end of the project we have a significant experience and knowledge in the organization to make the final call: if Kubernetes is the right choice for the company.

## Step one
Since I found that basic Docker knowledge is ubiquitous today, step one is to set up a dummy Kubernetes cluster. 

This opens up many parallel activities. First and foremost the vision can be showcased from this moment on. 

While many parts will be replaced and hardened in the coming steps, the basic developer experience of using the kubectl CLI will not change. 
The standardized blue-green or rolling deployments can be demonstrated to any stakeholder anytime and developers can start learning the tricks of kubectl for debugging and other day to day activities.

This step involves having basic routing, so people can demo their services, and the usage of a container registry.

This will trigger early adopters in the organization to start writing Kubernetes service definitions yaml files, and a solid sign that the project is actually happening.

**How to achive this step quickly?**

Kubernetes was well known as something hard to install but these days I find it easier to actually get started with the various distributions vendors offer. 
My pick is Rancher, which makes it easy to install a dummy Kubernetes with basic routing, metrics and other services. 

And while Rancher offer templates for more resilient cluster setups, if we decide to go with another vendor, the lock-in is not that big.
Few months back I was able to flip my Openshift targeted yaml files to Google Container Engine in a matter of six hours for an applications involving five services.

## Step two

Step one quickly triggers new requirements for the cluster. 

Shipping logs to an aggregated logging service, and metrics to the existing metrics infrastructure is a genuine need once people start experimenting with the cluster.


Peculiar routing needs can also surface, and the need of SSL can also come sooner than anticipated. This is a healthy organic process that makes the cluster more production like. 

Perhaps the first incidents occur as well in the form of disks filling up. All these cases grow our toolchain and general understanding of the infrastructure.

By the end of this phase the first service should be deployed on a staging like environment to get some real workloads, and to generate some real pain in case the service is down, triggering further learning in problem mitigation and operational practices. 

## Step three

While basic services are in place on the cluster, and people have a basic understanding of the tools, the cluster still very vulnerable at this point.

Recreating the cluster on a proper network layout in a HA setup is probably the right next step. Doing that in the favored configuration management tool to be able to easily reproduce the work is probably a good idea.  

## Step four

By step four probably there is a sizeable list of TODOs before going to production.

Skipped tasks like SSL certificates should be implemented latest at this step, and improvements to previous pragmatic choices should be carried out.

Most likely infamous problems also surfaced by now, perhaps a hanging Docker daemon, or a periodically dissappearing node should be addressed.
The support workload of the ever growing number services on the cluster will also be a factor.

By this time the project has been ongoing for a few months and every organization should be able to draw their conclusion whether to go to production or not.

## Failure modes
 
* **Don't debate, do**
<br/>The biggest enemy of the Docker / Kubernetes projects I see is the endless debates. Perhaps it's because of overwhelming number of decisions that have to be made, or the learning curve, but projects often stall on the altair of finding the best registry, CI or monitoring tool.
<br/>Keep in mind, that a pull request trumps a thousand argument. If only meetings that happen, it's not an unfriendly move to settle it with a working PR. One can respond with a counter PR if that invested in the decision. In the meantime though there is a working solution.

* **Death by a thousand papercuts**
<br/>We should never forget that Kubernetes and Docker at al. is a new technology. And while the horror stories that one can read on HackerNews perhaps painting a grimmer picture that it really is,
it can easily happen that for a given company it will just not work out. Death by a thousand papercuts is really threatening, the team can be exhausted from the seemingly endless issues, perhaps with a metrics collector integration, or Docker daemon issues in the given infrastructure.
<br/>One can only fight against this by setting a large enough inertia and skill in the team by involving as many parties as humanly possible who want to project to succeed. A non interested party gives up easier on an issue than someone who genuinely believes that the benefits outweigh the issues.
Make sure you identify these people early on, bring in the expertise if needed, but also accept the failure if it happens. It might be that a set of shell scripts running Docker on a VM is what the solution is for you

    