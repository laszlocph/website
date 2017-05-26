---
layout: default
---

## Rancher Kubernetes persistence with GlusterFS

The bellow described method was tested now on the Rancher Kubernetes distribution but I borrow the code from a previous project when I used Openshift. It works on both platforms without any modification. 

Not surprisingly as the Kubernetes upstream generally [supports](https://kubernetes.io/docs/concepts/storage/volumes/) GlusterFS. This definitelly raise my confidence level and lowers the risk when you pick a Kube distro. The yaml files are mostly portable, like in an occasion when I flipped my five service large project from Openshift to Google Cloude Engine (GKE) in six hours.

But back to the main question, the infamous one: 

**How do you handle persistence when you orchestrate Docker containers?** 

The question I'm constantly being asked when starting a new project. 

And my answer is: you don't. Or at least I hope you won't. But instead, your services are stateless, and when it comes to state you use Cloud Native solutions like S3 for files and a managed RDS instance for your data. 

Persistence is a hard question, and the answers the Docker world gives you are no different than the current solutions out there. At the end of the day you either make sure that the same folder sturcture is available on all your cluster nodes (with NFS for example outside of Kubernetes) or talk to the file cluster on a given endpoint through a proprietary protocol like in the case with GlusterFS.

But **let's say you do a lift and shift project, or the managed cloud services are not an option for you, what do you do then?**

Just allow me one more thought here and I promise I won't be the grumpy old dev anymore: make sure you containerize your persistence / database for the right reasons. In my experience databases are not the workloads that mostly benefit from the many advantages Docker gives you. You are not releasing new version of your database every day, hack every six month would be uncommon. 

I guess all I'm saying is start with the usecases that most benefit form containers: your stateless services. Think twice before you containerize your DB, leave it til the very end when you gained operational expertise with your new cluster, or perhaps leave it out completely, or do it only on your QA environments. Which by the way the most brilliant usecase for containerized DBs. I could kill for test environments per PR with its own DB serving nightly anonimized production exports. I'm sure your QA would kill for too. 

## Install GlusterFS

In my experience the tutorials around Gluster often start with installing Gluster as a *containerized* service. Well that was rather confusing to me - perhaps because of the quality of the tutorials I've read - but the moment I installed Gluster on dedicated nodes, everything fell into its place: Kubernetes only acted as a client for GlusterFS and I only had to navigate the logical concepts of Kubernetes for accessing the storage.

And installing a dummy Gluster cluster (hah!) was proven to be very easy following the [Gluster quickstart](https://gluster.readthedocs.io/en/latest/Quick-Start-Guide/Quickstart/). It's as easy as the length of the page suggests. Sweet.

## The chain of abstractions

There are quite a few entities you have to create to finally be able to store a file from your Kube service. Bare in mind that they all bring a level of abstraction - thus flexibility - but it's a bit overwhelming at first, and if you miss a piece, your chain will be broken and it won't work

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

It references the Kubernetes endpoint to know where the GlusterFS cluster is available, and the "gv0" volume name in GlusterFS.

Warning: the Recycle policy might not be available. At least at the time of writing I had to write my own shell script to clean up used disks since it's not implemented yet in Kube.

All nodes in Kubernetes cluster must have GlusterFS-Client Package installed. (TBD)

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
        - mountPath: /mnt/ifs_storage
          name: dir-1
      volumes:
        - name: dir-1
          persistentVolumeClaim:
            claimName: file-upload-claim
</pre>



cat svc, cat endpoint
create both
get both

demo the disks working on mounts
with watch

show and create pv

show and create pvc

show yaml, deploy

ssh into pod, create file, watch disks for change


