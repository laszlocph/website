---
layout: default
title: Rancher Kubernetes persistence with GlusterFS
image: rancher.png
link: /Rancher-Kubernetes-persistence-with-GlusterFS
permalink: /Rancher-Kubernetes-persistence-with-GlusterFS
excerpt: How do you handle persistence when you orchestrate Docker containers? The question I'm constantly being asked when starting a new project. 

---

## Rancher Kubernetes persistence with GlusterFS

The bellow described method was tested on the Rancher Kubernetes distribution but I borrow the code from a previous project when I used Openshift. It works on both platforms without any modification. 

Not surprisingly as the Kubernetes upstream generally [supports](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes) GlusterFS. This definitelly raise my confidence level and lowers the risk when you pick a Kube distro. The yaml files are mostly portable, like in an occasion when I flipped my five service large project from Openshift to Google Cloude Engine (GKE) in six hours.

But back to the main question, the infamous one: 

**How do you handle persistence when you orchestrate Docker containers?** 

The question I'm constantly being asked when starting a new project. 

And my answer is: you don't. Or at least I hope you won't. But instead, your services are stateless, and when it comes to state you use Cloud Native solutions like S3 for files and a managed RDS instance for your data. 

Persistence is a hard question, and the answers the Docker world gives you are no different than the current solutions out there. At the end of the day you either make sure that the same folder structure is available on all your cluster nodes (with NFS for example outside of Kubernetes) or talk to the file cluster on a given endpoint through a proprietary protocol like in the case with GlusterFS.

But **let's say you do a lift and shift project, or the managed cloud services are not an option for you, what do you do then?**

Just allow me one more thought here and I promise I won't be the grumpy dev anymore: make sure you containerize your persistence / database for the right reasons. In my experience databases are not the workloads that mostly benefit from the many advantages Docker gives you. You are not releasing new version of your database every day - hack every six month would be uncommon. 

I guess all I'm saying is start with the usecases that most benefit from containers: with your stateless services. Think twice before you containerize your database, leave it til the very end once you gained operational expertise with your new cluster, or perhaps leave it out completely. Or do it only on your QA environments, which by the way the most brilliant usecase for containerized databases in my opinion. I could kill for test environments per pull request backed by its dedicated database serving anonimized production exports. I'm sure your QA would kill for it too. 

## Install GlusterFS

In my experience the tutorials around Gluster often start with installing Gluster as a *containerized* service. Well that was rather confusing to me, perhaps because of the quality of the tutorials I've read, but the moment I installed Gluster on dedicated nodes, everything fell into its place. 

Kubernetes only acted as a client for GlusterFS and I only had to navigate the logical concepts of Kubernetes for accessing the storage.

Installing a standalone Gluster cluster (hah!) was proven to be very easy following the [Gluster quickstart](https://gluster.readthedocs.io/en/latest/Quick-Start-Guide/Quickstart/). It's as easy as the length of the page suggests. Sweet.

## The chain of abstractions

There are quite a few entities you have to create to be able to store a file from your Kube service. 

Bare in mind that they all bring a level of abstraction - thus flexibility - but it's a bit overwhelming at first, and if you miss a piece, your chain will be broken and it won't work.

* a volume on your GlusterFS cluster (cluster admin's responsibility)
* a persistent volume in Kubernetes (cluster admin's responsibility)
* a persistent volume claim (developer's responsibility)
* a volume mount in your yaml (developer's responsibility)

You have to create all the above entities for any given volume, and on top of that, as a one time setup, you have to create

* a Kubernetes endpoint for the GlusterFS cluster (cluster admin's responsibility)
* a Kubernetes service for the endpoint (cluster admin's responsibility)

In brackets I listed who is responsible for each component. 

The Persistent Volume (PV) - Persistent Volume Claim (PVC) construct detach the management of the storage from the usage. The developers don't have to know the underlyng implementation. Well, besides the access characteristics.

## The yamls

First create the endpoint and the service to describe the access details of the standalone GlusterFS.

#### 40-gluster-endpoint.yml

<pre>
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
subsets:
  - addresses:
      - ip: 10.0.1.102
    ports:
      - port: 1
  - addresses:
      - ip: 10.0.1.103
    ports:
      - port: 1
</pre>

#### 40-gluster-service.yml

<pre>
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster
spec:
  ports:
  - port: 1
</pre>


#### 50-persistent-volume.yml

Then reference the endpoint to tell Kubernetes where the GlusterFS cluster is available, and the "gv0" volume name in that you created in GlusterFS.

<pre>
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fileupload-vol
  labels:
    dev: dev
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 500M
  glusterfs:
    endpoints: glusterfs-cluster
    path: gv0
  persistentVolumeReclaimPolicy: Recycle
</pre>

#### 60-pvc.yml

<pre>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: file-upload-claim
spec:
  accessModes:
  - ReadWriteMany
  resources:
     requests:
       storage: 100M
</pre>

#### 60-nginx-svc.yml

Finally deploy your service, and mount the previously defined persistent volume.

<pre>
apiVersion: v1
kind: List
items:
- apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    name: nginx
  spec:
    replicas: 1
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx
          volumeMounts:
          - mountPath: /mnt/
            name: dir-1   
        volumes:
          - name: dir-1
            persistentVolumeClaim:
              claimName: file-upload-claim
</pre>

<blockquote class="instagram-media" data-instgrm-captioned data-instgrm-version="7" style=" background:#FFF; border:0; border-radius:3px; box-shadow:0 0 1px 0 rgba(0,0,0,0.5),0 1px 10px 0 rgba(0,0,0,0.15); margin: 1px; max-width:658px; padding:0; width:99.375%; width:-webkit-calc(100% - 2px); width:calc(100% - 2px);"><div style="padding:8px;"> <div style=" background:#F8F8F8; line-height:0; margin-top:40px; padding:50.0% 0; text-align:center; width:100%;"> <div style=" background:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACwAAAAsCAMAAAApWqozAAAABGdBTUEAALGPC/xhBQAAAAFzUkdCAK7OHOkAAAAMUExURczMzPf399fX1+bm5mzY9AMAAADiSURBVDjLvZXbEsMgCES5/P8/t9FuRVCRmU73JWlzosgSIIZURCjo/ad+EQJJB4Hv8BFt+IDpQoCx1wjOSBFhh2XssxEIYn3ulI/6MNReE07UIWJEv8UEOWDS88LY97kqyTliJKKtuYBbruAyVh5wOHiXmpi5we58Ek028czwyuQdLKPG1Bkb4NnM+VeAnfHqn1k4+GPT6uGQcvu2h2OVuIf/gWUFyy8OWEpdyZSa3aVCqpVoVvzZZ2VTnn2wU8qzVjDDetO90GSy9mVLqtgYSy231MxrY6I2gGqjrTY0L8fxCxfCBbhWrsYYAAAAAElFTkSuQmCC); display:block; height:44px; margin:0 auto -44px; position:relative; top:-22px; width:44px;"></div></div> <p style=" margin:8px 0 0 0; padding:0 4px;"> <a href="https://www.instagram.com/p/BU1z0EShN-V/" style=" color:#000; font-family:Arial,sans-serif; font-size:14px; font-style:normal; font-weight:normal; line-height:17px; text-decoration:none; word-wrap:break-word;" target="_blank">#kubernetes #k8s VolumeMounts and watching changes on the GlusterFS node&#39;s filesystem</a></p> <p style=" color:#c9c8cd; font-family:Arial,sans-serif; font-size:14px; line-height:17px; margin-bottom:0; margin-top:8px; overflow:hidden; padding:8px 0 7px; text-align:center; text-overflow:ellipsis; white-space:nowrap;">A post shared by Laszlo Cloud (@laszlo.cloud) on <time style=" font-family:Arial,sans-serif; font-size:14px; line-height:17px;" datetime="2017-06-02T14:36:38+00:00">Jun 2, 2017 at 7:36am PDT</time></p></div></blockquote>
<script async defer src="//platform.instagram.com/en_US/embeds.js"></script>

## Caveats

If you deployed all the components, you should be able to create a file within your pod and validate the creation on your GlusterFS volume.

There are two issues I faced in my deployment. The recycle *persistentVolumeReclaimPolicy* is not yet implemented, so I had to create a cleanup cron task, and second, the volume was mounted for the root user and I couldn't find a nice solution to grant access to my non root user in the container. I used a *postStart* lifecycle action in the pod definition, but I wasn't very satisfied with this solution.

UPDATE: just realized that fsGroup as described [here](https://kubernetes.io/docs/concepts/policy/security-context/) is solving the problem with the ownership of folders on the mounted disk. 

2017-05-26