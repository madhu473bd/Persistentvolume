Persistent Volumes
=================
You can persist data in IBM Cloud Container Service to share data between app instances and to protect your data from being lost if a component in your Kubernetes cluster fails. For having a high availability of storage in the IBM cloud container service we have several options available.

# Non-persistent data storage options
## [Inside the container](https://console.bluemix.net/docs/containers/cs_storage.html#storage)
> Containers and pods are, by design, short-lived and can fail unexpectedly. However, you can write data to the local file system of the container to store data throughout the lifecycle of the container. Data inside a container cannot be shared with other containers or pods and is lost when the container crashes or is removed. For more information, see [Storing data in a container](https://docs.docker.com/storage/).
## [On the worker node](https://console.bluemix.net/docs/containers/cs_storage.html#storage)
> Every worker node is set up with primary and secondary storage that is determined by the machine type that you select for your worker node. The primary storage is used to store data from the operating system and can be accessed by using a [Kubernetes hostPath volume](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath). The secondary storage is used to store data in /var/lib/docker, the directory that all the container data is written to. You can access the secondary storage by using a [Kubernetes emptyDir volume](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir).

> While hostPath volumes are used to mount files from the worker node file system to your pod, emptyDir creates an empty directory that is assigned to a pod in your cluster. All containers in that pod can read from and write to that volume. Because the volume is assigned to one specific pod, data cannot be shared with other pods in a replica set.
### Example of hostpath persistent Volume.
  * Persistent Volume
  
       This creates a persistent volume with the name task-pv-volume.
       The configuration file specifies that the volume is at /tmp/data on the the cluster’s Node. The configuration also specifies a size of 10 gibibytes and an access mode of ReadWriteOnce, which means the volume can be mounted as read-write by a single Node.
       It defines the StorageClass name manual for the PersistentVolume, which will be used to bind PersistentVolumeClaim requests to this PersistentVolume.
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
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                    STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO            Retain           Bound     default/task-pv-claim1   manual                   3h
```


# Persistent data storage options for high availability
## NFS file store     
       * [Pesistent Volume](#persistent-volume)
       * [Persistent Volume Claim](#persistent-volume-claim)
       * [Create a pod with persistent Volume](#create-a-pod-with-persistent-volume)
## Cloud database service
## On-prem database
# NFS file store  

##Persistent Volume


