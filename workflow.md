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
  #### Persistent Volume
  
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
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS        CLAIM                    STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO            Retain           Available     default/task-pv-claim1   manual                   3h
```
Here yo see the status of the volume to be available and once you create the Persistent volume Claim and attach the PV and PVC you can see that status to change to Bound.

  #### PersistentVolumeClaim
  
     Pods use PersistentVolumeClaims to request physical storage.After you create the PersistentVolumeClaim, the Kubernetes control plane looks for a PersistentVolume that satisfies the claim’s requirements. If the control plane finds a suitable PersistentVolume with the same StorageClass, it binds the claim to the volume.
         
 ```
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
Notice that the Pod’s configuration file specifies a PersistentVolumeClaim, but it does not specify a PersistentVolume. From the Pod’s point of view, the claim is a volume.
Here we are mapping task-pv-claim (/tmp/data) with /usr/share/nginx/html in kubernetes Pod.
Now /tmp/data is on the host system and /usr/share/nginx/html is inside the Pod, if you update /usr/share/nginx/html inside Pod /tmp/data on host will reflact the same.


# Persistent data storage options for high availability
## NFS file store     
       * [Pesistent Volume](#persistent-volume)
       * [Persistent Volume Claim](#persistent-volume-claim)
       * [Create a pod with persistent Volume](#create-a-pod-with-persistent-volume)
## Cloud database service
## On-prem database
# NFS file store  

##Persistent Volume


