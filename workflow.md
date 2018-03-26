# Kubernetes Volumes 
We have different types of [volumes](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) which can be used with the kubernetes.
## Consider this example of using the hostpath configuration
```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```
Persistent Volumes
=================
You can persist data in IBM Cloud Container Service to share data between app instances and to protect your data from being lost if a component in your Kubernetes cluster fails. For having a high availability of storage in the IBM cloud container service we have several options available.

## Example of the configuration file of Persistent Volume

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```
## Volume Mode
Prior to v1.9, the default behavior for all volume plugins was to create a filesystem on the persistent volume. With v1.9, the user can specify a volumeMode which will now support raw block devices in addition to file systems. Valid values for volumeMode are “Filesystem” or “Block”. If left unspecified, volumeMode defaults to “Filesystem” internally. This is an optional API parameter.

## Access Modes

The access modes are:

ReadWriteOnce – the volume can be mounted as read-write by a single node

ReadOnlyMany – the volume can be mounted read-only by many nodes

ReadWriteMany – the volume can be mounted as read-write by many nodes

In the CLI, the access modes are abbreviated to:

RWO - ReadWriteOnce

ROX - ReadOnlyMany

RWX - ReadWriteMany

> Important! A volume can only be mounted using one access mode at a time, even if it supports many. For example, a GCEPersistentDisk can be mounted as ReadWriteOnce by a single node or ReadOnlyMany by many nodes, but not at the same time.


# Non-persistent data storage options
## [Inside the container](https://console.bluemix.net/docs/containers/cs_storage.html#storage)
> Containers and pods are, by design, short-lived and can fail unexpectedly. However, you can write data to the local file system of the container to store data throughout the lifecycle of the container. Data inside a container cannot be shared with other containers or pods and is lost when the container crashes or is removed. For more information, see [Storing data in a container](https://docs.docker.com/storage/).
## [On the worker node](https://console.bluemix.net/docs/containers/cs_storage.html#storage)
> Every worker node is set up with primary and secondary storage that is determined by the machine type that you select for your worker node. The primary storage is used to store data from the operating system and can be accessed by using a [Kubernetes hostPath volume](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath). The secondary storage is used to store data in /var/lib/docker, the directory that all the container data is written to. You can access the secondary storage by using a [Kubernetes emptyDir volume](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir).

> While hostPath volumes are used to mount files from the worker node file system to your pod, emptyDir creates an empty directory that is assigned to a pod in your cluster. All containers in that pod can read from and write to that volume. Because the volume is assigned to one specific pod, data cannot be shared with other pods in a replica set.
### Example of hostpath persistent Volume.
  #### Persistent Volume
  
   This creates a persistent volume with the name task-pv-volume.The configuration file specifies that the volume is at /tmp/data on the the cluster’s Node. The configuration also specifies a size of 10 gibibytes and an access mode of ReadWriteOnce, which means the volume can be mounted as read-write by a single Node.It defines the StorageClass name manual for the PersistentVolume, which will be used to bind PersistentVolumeClaim requests to this PersistentVolume.
```
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
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/tmp/data"
```
```
kubectl get pv task-pv-volume
```
```
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS        CLAIM                    STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO            Retain           Available     default/task-pv-claim1   manual                   3h
```
Here you see the status of the volume to be available and once you create the Persistent volume Claim and attach the PV and PVC you can see that status to change to Bound.

  #### PersistentVolumeClaim
  
Pods use PersistentVolumeClaims to request physical storage.After you create the PersistentVolumeClaim, the Kubernetes control plane looks for a PersistentVolume that satisfies the claim’s requirements. If the control plane finds a suitable PersistentVolume with the same StorageClass, it binds the claim to the volume.
         
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: task-pv-claim1
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi 
```
```
kubectl get pvc task-pv-claim
```
```
NAME            STATUS    VOLUME               CAPACITY   ACCESS MODES   STORAGECLASS   AGE
task-pv-claim   Bound     task-pv-nfs-volume   2Gi        RWO            manual         2h
```
```
kubectl get pv task-pv-volume
```
```
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                    STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO            Retain           Bound     default/task-pv-claim1   manual                   3h
```
Here you can see that the status has been changed to Bound from Available once the Persistent Volume Claim has been created.
   #### Create a Pod With Persistent Volume
```
kind: Pod
apiVersion: v1
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
       claimName: task-pv-claim1
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
Notice that the Pod’s configuration file specifies a PersistentVolumeClaim, but it does not specify a PersistentVolume. From the Pod’s point of view, the claim is a volume. Here we are mapping task-pv-claim (/tmp/data) with /usr/share/nginx/html in kubernetes Pod. Now /tmp/data is on the host system and /usr/share/nginx/html is inside the Pod, if you update /usr/share/nginx/html inside Pod /tmp/data on host will reflact the same.

```
On HOST
mkdir /tmp/data/
echo 'Hello from Kubernetes storage' > /tmp/data/index.html
```
```
kubectl create -f task-pv-volume.yaml
persistentvolume "task-pv-volume" created
kubectl create -f task-pv-claim.yaml
persistentvolumeclaim "task-pv-claim" created
kubectl create -f task-pv-pod.yaml
pod "task-pv-pod" created
```
```
Get inside the Pod
kubectl exec -it task-pv-pod -- /bin/bash
root@task-pv-pod:/# cd /usr/share/nginx/html/
root@task-pv-pod:/usr/share/nginx/html# ls
index.html
```

# Persistent data storage options for high availability
## [NFS file store](https://console.bluemix.net/docs/containers/cs_storage.html#storage)   
> With this option, you can persist app and container data by using Kubernetes persistent volumes. Volumes are hosted on [Endurance and Performance NFS-based file storage](https://www.ibm.com/cloud/file-storage/details) which can be used for apps that store data on a file basis rather than in a database. File storage is encrypted at REST.

> IBM Cloud Container Service provides predefined storage classes that define the range of sizes of the storage, IOPS, the delete policy, and the read and write permissions for the volume. To initiate a request for NFS-based file storage, you must create a persistent volume claim (PVC). After you submit a PVC, IBM Cloud Container Service dynamically provisions a persistent volume that is hosted on NFS-based file storage. You can mount the PVC as a volume to your deployment to allow the containers to read from and write to the volume.

## [Cloud database service](https://console.bluemix.net/docs/containers/cs_storage.html#storage)
> With this option, you can persist data by using an IBM Cloud database cloud service, such as IBM Cloudant NoSQL DB. Data that is stored with this option can be accessed across clusters, locations, and regions.

> You can choose to configure a single database instance that all your apps access, or to set up multiple instances across data centers and replication between the instances for higher availability. In IBM Cloudant NoSQL database, data is not backed up automatically. You can use the provided backup and restore mechanisms to protect your data from a site failure.

## [On-prem database](https://console.bluemix.net/docs/containers/cs_storage.html#storage)
> If your data must be stored on-site for legal reasons, you can set up a VPN connection to your on-premise database and use existing storage, backup and replication mechanisms in your data center.




