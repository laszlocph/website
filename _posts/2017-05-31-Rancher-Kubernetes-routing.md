---
layout: default
title: Rancher Kubernetes routing
image: rancher.png
link: /Rancher-Kubernetes-routing
permalink: /Rancher-Kubernetes-routing
excerpt: Recently I got to know Rancher's take on how traffic is routed to my Kubernetes services. I picked Rancher in the beginning of the project because it offered a working cluster with little effort. Routing being one of the things that just worked.

---

## Rancher Kubernetes routing

I identified three major milestones in the Kubernetes projects I had. 

Metrics, log aggregation, and routing being the three features that differ from project to project, depending on the 
existing infrastructure, and the chosen Kubernetes vendor. Delivering these parts are nice wins on the way to get a feature complete infrastructure. 
 
Having those three in place opens up the cluster for wider usage and my focus can be diverted to the ever ongoing iterations of hardening the setup.

## Rancher's take

Recently I got to know Rancher's take on how traffic is routed to my Kubernetes services. I picked Rancher in the beginning of the project because it offered a working cluster with little effort. Routing being one of the things that just worked.

To access a Kubernetes service, the only thing I had to do is to create a Kubernetes service with *"type: LoadBalancer"*

<pre>
kind: Service
apiVersion: v1
metadata:
  name: bot
spec:
  selector:
    app: bot
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  <b>type: LoadBalancer</b>
</pre>

By doing so and running "kubectl get svc" an EXTERNAL-IP showed up after a few seconds and I was good to access the service from the browser. 

Later when I used internal IPs in my cluster, the described method was still working, as long as the internal IP was the one where my Azure Load Balancer was pointing to. 

It felt a bit like magic, but it only started to bother me, when I experienced port collisions for port 80; and when the service IPs were on nodes that were not part of my load balancer pool.
 
**It turned out Rancher creates a LoadBalancer container for each service you define and place them accross all rancher nodes, no matter what their Kubernetes role is.** Meaning one point a load balancer was provisioned on my etcd node. Not quite the design I was going for.

While I knew that I have to address routing at some point, I was slightly concerned by this behavior.

## Ingress to the rescue

It was clear that I wanted to have host based routing and offer all my services on standard ports. The [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#name-based-virtual-hosting) abstraction felt like the way to go, and I was pleased that Rancher [supports](https://docs.rancher.com/rancher/v1.3/en/kubernetes/ingress/) it.

I had the idea that the Ingress definitions will look something like this:

<pre>
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: s1-ingress
spec:
  rules:
  - host: s1.mycompany.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
</pre>

<pre>
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: s2-ingress
spec:
  rules:
  - host: s2.mycompany.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
</pre>

One little Ingress snippet for each service I have, so it can be controlled by the feature team who owns the service and the services would be available on s1.mycompany.com and s2.mycompany.com respectively.

## The Rancher Ingress Controller

The difference between the Ingress and the Ingress controller was not crystal clear for me, but after some digging it materialized for me that the Ingress Controller is a component that listens to Kubernetes lifecycle events, and when an Ingress is defined, deleted or changed it makes the appropriate modifications in the "router" that actually handles the requests.

Let me try to describe each component's behavior in Rancher world:

* **Ingress**: is a Kubernetes logical abstraction that is nothing more than the piece of yaml I showed above.
* **Ingress Controller**: It's a container itself that listens to Kubernetes events and creates a "router" for each Ingress definition. It does not handle traffic, it's simple an event listener.
* **Rancher Load Balancer**: The "router" that the controller creates for each Ingress definition is a Rancher Load Balancer

Since the Rancher Ingress Controller creates a Rancher Load Balancer for every service I was not in a better position than before: port collisions were still an issue, and placement of these LoadBalancers was not in my control.

By re-reading the [Rancher's documentation](https://docs.rancher.com/rancher/v1.3/en/kubernetes/ingress/) I found a somewhat okay solution as I was able to control the placement of the Loadbalancers, plus if I kept all my service entrypoints in one giant Ingress definition I ended up having only one LoadBalancer.
 
<pre>
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
name: host-based-ingress
annotations:
  io.rancher.scheduler.affinity.host_label: "orchestration=true"
  allow.http: "false"
spec:
tls:
 - secretName: mycompany-tls
rules:
- host: s1.mycompany.com
  http:
    paths:
    - backend:
        serviceName: s1
        servicePort: 80
- host: s2.mycompany.com
  http:
    paths:
    - backend:
        serviceName: s2
        servicePort: 80
- host: s3.mycompany.com
  http:
    paths:
    - backend:
        serviceName: s3
        servicePort: 80
</pre>

But as you can tell, having one giant config for all the routing is really not convenient and prone to errors.

## The Kubernetes Ingress Controller

It bothered me a lot, so a few days later I checked out what the vanilla Kubernetes has to offer. 

I knew it can be done as Openshift's Router worked as I would prefer: having many Ingress definitions that all modify one central router software component (which was HAProxy that case).

At first it wasn't very comforting as the Kubernetes Ingress doc just points to a Github repository, but it turns out the [Nginx based controller](https://github.com/kubernetes/ingress/tree/master/controllers/nginx) they recommend works as I expect it to: the ingress controller was serving the traffic itself, and I could define an Ingress per Service.

I quickly deployed the default backend and the controller and while I was able to follow the virtual host changes within the ingress container, one component was missing. Routing traffic to the Ingress Controller was not solved yet. 

I had to create a Rancher LoadBalancer by hand to route all traffic to the Ingress Controller. 

It still feels like an extra manual step that shuldn't be necessary, but I think of this now as if I created an AWS ELB. At some point the sorrounding infrastructure has to pass traffic to Kube, so I think I'm fine with this.
The Rancher LB was also the place where I terminate SSL, so the Kube Ingress doesn't deal with it now.

## Unexpected design

I'm not sure what lead the Rancher team pick this unconventional design, I'm sure they have their reasons. 

What turned out to be a show stopper for me is that I either had to deal with port collisions or one giant shared Ingress file. None I'd like to live with long term and it seems the Kubernetes Ingress serves me well.

2017-05-31
