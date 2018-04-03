# Persistent Volumes 

Persistent Volumes allow you to persist data in IBM Cloud Container Service to share data between app instances and to protect your data from being lost if a component in your Kubernetes cluster fails. For having a high availability of storage in the IBM cloud container service we have several options available. The options that we have in IBM Cloud Container Service to make our data highly available in a cluster are `NFS file storage`, `Cloud database service`, `On-prem database`(we are not going to use this on-prem database unless in the extreme cases). The option that is right for us depends on the following factors:

* **The type of app that you have:** For example, you might have an app that must store data on a file basis rather than inside a database.
* **Legal requirements for where to store and route the data:** For example, you might be obligated to store and route data in the United States only and you cannot use a service that is located in Europe.
* **Backup and restore options:** Every storage options comes with capabilities to backup and restore data. Check that available backup and restore options meet the requirements of your disaster recovery plan, such as the frequency of backups or the capabilities of storing data outside your primary data center.
* **Global replication:** For high availability, you might want to set up multiple instances of storage that are distributed and replicated across data centers worldwide.

We also have different types of [volumes](https://kubernetes.io/docs/concepts/storage/volumes) which can be used with the Kubernetes. The storage types that are supported for the IBM Cloud Private management console are the following and we are going to discuss about the `NFS` file storage in the IBM Private Cloud in this document.

* [NFS](https://kubernetes.io/docs/concepts/storage/volumes/#nfs)
* [Gluster FS](https://kubernetes.io/docs/concepts/storage/volumes/#glusterfs)
* [vSphere Virtual Volume](https://kubernetes.io/docs/concepts/storage/volumes/#vspherevolume)
* [hostpath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)


## Persistent data storage options for high availability

### 1. [NFS file store](https://console.bluemix.net/docs/containers/cs_storage.html#storage)   
With this option, you can persist app and container data by using Kubernetes persistent volumes. Volumes are hosted on [Endurance and Performance NFS-based file storage](https://www.ibm.com/cloud/file-storage/details) which can be used for apps that store data on a file basis rather than in a database. File storage is encrypted at REST.

IBM Cloud Container Service provides predefined storage classes that define the range of sizes of the storage, IOPS, the delete policy, and the read and write permissions for the volume. To initiate a request for NFS-based file storage, you must create a `persistent volume claim (PVC)`. After you submit a `PVC`, IBM Cloud Container Service dynamically provisions a persistent volume that is hosted on NFS-based file storage. You can mount the `PVC` as a volume to your deployment to allow the containers to read from and write to the volume.

#### Storage classes we have for `NFS` file storage are:

StorageClass objects are created and managed by cluster administrators who can define available storage based on qualitative attributes such as “fast”, “slow”, “gold”, or “silver”, etc. These attributes can, in turn, reflect actual storage settings such as quality of service, availability, replication, backup policies, speed of disk, etc, as defined in the storage system.

IBM Cloud Container Service provides pre-defined storage classes for `NFS` file storage so that the cluster admin does not have to create any storage classes. The `ibmc-file-bronze` storage class is the same as the `default` storage class.

we can get the different types of storage classes available in IBM Cloud Container Service by the following command;

```sh

$ kubectl get storageclasses

NAME                         TYPE
default                      ibm.io/ibmc-file
ibmc-file-bronze (default)   ibm.io/ibmc-file
ibmc-file-custom             ibm.io/ibmc-file
ibmc-file-gold               ibm.io/ibmc-file
ibmc-file-retain-bronze      ibm.io/ibmc-file
ibmc-file-retain-custom      ibm.io/ibmc-file
ibmc-file-retain-gold        ibm.io/ibmc-file
ibmc-file-retain-silver      ibm.io/ibmc-file
ibmc-file-silver             ibm.io/ibmc-file
```

Now we have to choose whether we have to store the data or delete it when we delete the `PVC`. To keep the data we have to choose a `retain` storage class i.e., when we delete the `PVC`, the `PV` is removed, but the `NFS` file and the data still exist in our IBM Cloud infrastructure (SoftLayer) account. Otherwise, if we want the data and your NFS file share to be deleted when we delete the PVC, choose a storage class without retain.

If we choose a bronze, silver, or gold storage class, we get [Endurance storage](https://knowledgelayer.softlayer.com/topic/endurance-storage) that defines the IOPS per GB for each class. However, we can determine the total IOPS by choosing a size within the available range. We can select any whole number of gigabyte sizes within the allowed size range (such as 20 Gi, 256 Gi, 11854 Gi).

| Storage class |	IOPS per gigabyte |	Size range in gigabytes |     
| --- | --- | --- |
| Bronze (default) |	2 IOPS/GB |	20-12000 Gi |
| Silver |	4 IOPS/GB |	20-12000 Gi |
| Gold |	10 IOPS/GB |	20-4000 Gi |

If we choose the custom storage class, we get [Performance storage](https://knowledgelayer.softlayer.com/topic/performance-storage) and have more control over choosing the combination of IOPS and size. 

|Size range in gigabytes |	IOPS range in multiples of 100 |
| --- | --- |
|20-39 Gi |	100-1000 IOPS |
|40-79 Gi |	100-2000 IOPS |
|80-99 Gi |	100-4000 IOPS |
|100-499 Gi |	100-6000 IOPS |
|500-999 Gi |	100-10000 IOPS |
|1000-1999 Gi |	100-20000 IOPS |
|2000-2999 Gi |	200-40000 IOPS |
|3000-3999 Gi |	200-48000 IOPS |
|4000-7999 Gi |	300-48000 IOPS |
|8000-9999 Gi |	500-48000 IOPS |
|10000-12000 Gi |	1000-48000 IOPS |

#### Create a storage configuration file for `PV`
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: mypv
spec:
 capacity:
   storage: "20Gi"
 accessModes:
   - ReadWriteMany
 nfs:
   server: "nfslon0410b-fz.service.networklayer.com"
   path: "/IBM01SEV8491247_0908/data01"
```
#### name 	
Enter the name of the PV object that you want to create.

#### spec/capacity/storage
Enter the storage size of the existing NFS file share. The storage size must be written in gigabytes, for example, 20Gi (20 GB) or 1000Gi (1 TB), and the size must match the size of the existing file share.


#### spec/accessModes

The access modes are:

* ReadWriteOnce – the volume can be mounted as read-write by a single node

* ReadOnlyMany – the volume can be mounted read-only by many nodes

* ReadWriteMany – the volume can be mounted as read-write by many nodes

In the CLI, the access modes are abbreviated to:

* RWO - ReadWriteOnce

* ROX - ReadOnlyMany

* RWX - ReadWriteMany

#### spec/nfs/server 	
Enter the NFS file share server ID.

#### spec/nfs/path 	
Enter the path to the NFS file share where you want to create the PV object.


#### Create a `PVC` Configuration file

We can create the `PVC` with one of the storage classes `bronze,silver,gold` or `custom` which can be billed `hourly` or `monthly`. By default, we are billed `monthly`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "ibmc-file-silver"
  labels:
    billingType: "hourly"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 24Gi
```

To mount the `PVC` to our deployment, we have to create a configuration file as follows.

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: <deployment_name>
  labels:
    app: <deployment_label>
spec:
  selector:
    matchLabels:
      app: <app_name>
  template:
    metadata:
      labels:
        app: <app_name>
    spec:
      containers:
      - image: <image_name>
        name: <container_name>
        volumeMounts:
        - name: <volume_name>
          mountPath: /<file_path>
      volumes:
      - name: <volume_name>
        persistentVolumeClaim:
          claimName: <pvc_name>
```


### 2. Cloud database service
With this option, you can persist data by using an IBM Cloud database cloud service, such as IBM Cloudant NoSQL DB. Data that is stored with this option can be accessed across clusters, locations, and regions.

You can choose to configure a single database instance that all your apps access, or to [set up multiple instances across data centers and replication](https://console.bluemix.net/docs/services/Cloudant/guides/active-active.html#configuring-cloudant-nosql-db-for-cross-region-disaster-recovery) between the instances for higher availability. In IBM Cloudant NoSQL database, data is not backed up automatically. You can use the provided [backup and restore mechanisms](https://console.bluemix.net/docs/services/Cloudant/guides/backup-cookbook.html#cloudant-nosql-db-backup-and-recovery) to protect your data from a site failure.

### 3. On-prem database
At the moment we are not going to use this On-prem database but when needed in the extreme cases we can use this as a option.
If your data must be stored on-site for legal reasons, you can [set up a VPN connection](https://console.bluemix.net/docs/containers/cs_vpn.html#vpn) to your on-premise database and use existing storage, backup and replication mechanisms in your data center.

## Non-persistent data storage options
### Inside the container
Containers and pods are, by design, short-lived and can fail unexpectedly. However, you can write data to the local file system of the container to store data throughout the lifecycle of the container. Data inside a container cannot be shared with other containers or pods and is lost when the container crashes or is removed. For more information, see [Storing data in a container](https://docs.docker.com/storage/).
### On the worker node
Every worker node is set up with primary and secondary storage that is determined by the machine type that you select for your worker node. The primary storage is used to store data from the operating system and can be accessed by using a [Kubernetes `hostPath` volume](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath). The secondary storage is used to store data in `/var/lib/docker`, the directory that all the container data is written to. You can access the secondary storage by using a [Kubernetes `emptyDir` volume](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir).

While `hostPath` volumes are used to mount files from the worker node file system to your pod, `emptyDir` creates an empty directory that is assigned to a pod in your cluster. All containers in that pod can read from and write to that volume. Because the volume is assigned to one specific pod, data cannot be shared with other pods in a replica set.

### Consider the example of hostpath Volume for creating the PV, PVC and the Pod with PVC.
A `hostPath` volume mounts a file or directory from the host node’s filesystem into your pod. This is not something that most Pods will need, but it offers a powerful escape hatch for some applications.
For example, some uses for a `hostPath` are:
* running a container that needs access to Docker internals; use a hostPath of `/var/lib/docker`
* running cAdvisor in a container; use a hostPath of `/sys`
* allowing a pod to specify whether a given hostPath should exist prior to the pod running, whether it should be created, and what it should exist as.

#### Creating a Persistent Volume
  
This creates a persistent volume with the name `task-pv-volume`.The configuration file specifies that the volume is at `/tmp/data` on the the cluster’s Node. The configuration also specifies a size of 10 Gi and an access mode of `ReadWriteOnce`, which means the volume can be mounted as read-write by a single Node.It defines the `StorageClass` name manual for the `PersistentVolume`, which will be used to bind `PersistentVolumeClaim` requests to this `PersistentVolume`.
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/data"
```

#### Note
{% hint style='info' %}

Important! A volume can only be mounted using one access mode at a time, even if it supports many. For example, a volume can be mounted as ReadWriteOnce by a single node or ReadOnlyMany by many nodes, but not at the same time.

{% endhint %}

```sh
kubectl get pv task-pv-volume
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS        CLAIM                    STORAGECLASS   REASON   AGE
task-pv-volume   10Gi       RWO            Retain           Available     default/task-pv-claim    manual                  3h
```
Here you see the status of the volume to be available and once you create the `Persistent volume Claim` and attach the `PV` and `PVC` you can see that status to change to Bound.

#### Creating a PersistentVolumeClaim
  
Pods use `PersistentVolumeClaims` to request physical storage. After you create the `PersistentVolumeClaim`, the Kubernetes control plane looks for a `PersistentVolume` that satisfies the claim’s requirements. If the control plane finds a suitable `PersistentVolume` with the same `StorageClass`, it binds the claim to the volume.
         
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi 
```
```sh
kubectl get pvc task-pv-claim

NAME            STATUS    VOLUME               CAPACITY   ACCESS MODES   STORAGECLASS   AGE
task-pv-claim   Bound     task-pv-nfs-volume   2Gi        RWO            manual         2h

kubectl get pv task-pv-volume

NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                    STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO            Retain           Bound     default/task-pv-claim    manual                   3h
```
Here you can see that the status has been changed to Bound from Available once the Persistent Volume Claim has been created.

#### Create a Pod With Persistent Volume
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
       claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```
Notice that the Pod’s configuration file specifies a `PersistentVolumeClaim`, but it does not specify a `PersistentVolume`. From the Pod’s point of view, the claim is a volume. Here we are mapping `task-pv-claim (/tmp/data)` with `/usr/share/nginx/html` in Kubernetes Pod. Now `/tmp/data` is on the host system and `/usr/share/nginx/html` is inside the Pod, if you update `/usr/share/nginx/html` inside Pod `/tmp/data` on host will reflect the same.

```sh
# On HOST
mkdir /tmp/data/
echo 'Hello from Kubernetes storage' > /tmp/data/index.html

# Creating the PV, PVC, POD
kubectl create -f task-pv-volume.yaml
persistentvolume "task-pv-volume" created
kubectl create -f task-pv-claim.yaml
persistentvolumeclaim "task-pv-claim" created
kubectl create -f task-pv-pod.yaml
pod "task-pv-pod" created

# Get inside the Pod
kubectl exec -it task-pv-pod -- /bin/bash
root@task-pv-pod:/# cd /usr/share/nginx/html/
root@task-pv-pod:/usr/share/nginx/html# ls
index.html
```

