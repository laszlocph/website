---
layout: default
title: Why access control is key for a secure multi-tenant Kubernetes deployment?
image: images/kubernetes.png
link: //Why-access-control-is-key-for-a-secure-multi-tenant-Kubernetes-deployment
permalink: /Why-access-control-is-key-for-a-secure-multi-tenant-Kubernetes-deployment
excerpt: Why access control is key for a secure multi-tenant Kubernetes deployment?
--- 

## Why access control is key for a secure multi-tenant Kubernetes deployment?
{{ page.date | date: "%Y-%m-%d" }}

I have a soft spot for early adopters. They are the ones who voice those - sometimes pesky - requirements that push container projects forward, making the cluster more mature and the workflow more tailored for real life usecases.

#### Secrets exposed

Like on my current Kubernetes project, when a few weeks back an early adopter came to me (hey Jonas!) and raised his concern around secret management: *"Look Laszlo, I'm a bit worried about the keys I'm uploading to Kubernetes. It seems devs from other teams are able to see them. You know, they are kind of important.. they allow access to confidential data"*

He had a point. At that time there was only one shared namespace and developers saw each other's deployments, nothing preventing them from wrongdoing. Except their morals of course. 

It was time to iterate on the cluster and nail access control.

#### Access control introduced in Kubernetes v1.6

The upstream Kubernetes had Role Based Access Control (RBAC) since the 1.6 release in late March, and timing couldn't have been better for Rancher - my distribution of choice - to finally support that major release. No wonder I was so pumped seeing the announcement.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">This is fantastic! Rancher v.1.6.3 is out with Role based access control in Kubernetes <a href="https://t.co/FT2AAE4mkp">https://t.co/FT2AAE4mkp</a> <a href="https://twitter.com/Rancher_Labs">@Rancher_Labs</a></p>&mdash; laszlocph (@laszlocph) <a href="https://twitter.com/laszlocph/status/883317597755895808">July 7, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<br/>
Access control is quite an important feature if you ask me. When I worked with the Openshift Kubernetes distribution back in January it felt natural that it has the feature, and only recently I learned that upstream Kubernetes, nor my distribution had it at the time.

The only workaround I could provide until now is to physically separate the clusters along the security boundaries. Which is still the best practice for test and production environments: even though one can label nodes and set up namespaces accordingly, life is easier if those environments remain phisically separate. 

For this article I quickly checked [what's up](https://docs.docker.com/engine/extend/plugins_authorization/) in the Docker Swarm world: *"Docker’s out-of-the-box authorization model is all or nothing. Any user with permission to access the Docker daemon can run any Docker client command."* 

But don't I worry since *"Anyone with the appropriate skills can develop an authorization plugin."* - well, thanks Docker!

#### Access control primitives

Kubernetes [RBAC](https://kubernetes.io/docs/admin/authorization/rbac/) allows cluster administrators to selectively grant particular users or service accounts fine-grained access to specific resources on a per-namespace basis.

The key primitives are:
 
 * Role and ClusterRole to represent a set of permissions to be granted together on a Kubernetes Namespace or on the whole cluster
 
 * RoleBinding and ClusterRoleBinding to associate the defined Roles and ClusterRoles to users or Kubernetes ServiceAccounts
 
The permissions can be fine-grained: granting rights to create, get, list, update, patch, delete any of the Kubernetes API resources, be those Pods, Secrets, NameSpaces, etc. 

But luckily Kubernetes also provides sensible defaults for system components and for three common usecases: admin, edit and view, if you don't want to write your own page long permission set. 

Another nice property of the Roles that they are additive.

#### The secure and multi-tenant container platform

Using these primitives I was able to build the secure, multi-tenant container platform that multiple teams can use while not seeing or interfering with each other's deployments or secrets. 

And the icing on the cake was Rancher's nice touch. Since they already support many authentication providers from Active Directory to Github, **I was able to bind the RBAC roles to the Github teams that already existed** in my client's organization. How cool is that?

So the setup as of today follows the following principles:

* Every team gets their own namespace
* Users are granted permissions to namespaces through their Github team membership
* People get the default "edit" Role within the namespace

and to further protect the api keys and certificates the following workflow is introduced for secrets:

* Secrets are not part of the application source code
* Their lifecycle is decoupled from the application by the Kubernetes Secret artifact.
* The source code is referencing the secrets by their logical name
* Secrets are prepared in the cluster by cluster admins, prior to deployment
* Secrets are stored in an encrypted Chef or Ansible Vault

See the details on how to [define secrets](https://kubernetes.io/docs/concepts/configuration/secret/#overview-of-secrets) in Kubernetes and what's their [security properties](https://kubernetes.io/docs/concepts/configuration/secret/#security-properties) and associated risks.

<br/>

Onwards!



