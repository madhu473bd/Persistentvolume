Topics
=================
You can persist data in IBM Cloud Container Service to share data between app instances and to protect your data from being lost if a component in your Kubernetes cluster fails. For having a high availability of storage in the IBM cloud container service we have several options available.

# Non-persistent data storage options
## Inside the container
[From](https://console.bluemix.net/docs/containers/cs_storage.html#storage
>      Containers and pods are, by design, short-lived and can fail unexpectedly. However, you can write data to the local file system of the container to store data throughout the lifecycle of the container. Data inside a container cannot be shared with other containers or pods and is lost when the container crashes or is removed. For more information, see Storing data in a container.   
## On the worker node
    
* Persistent data storage options for high availability
    * NFS file store     
       * [Pesistent Volume](#persistent-volume)
       * [Persistent Volume Claim](#persistent-volume-claim)
       * [Create a pod with persistent Volume](#create-a-pod-with-persistent-volume)
    * Cloud database service
    * On-prem database
# NFS file store  

##Persistent Volume

This creates a persistent volume with the name task-pv-volume.
The configuration file specifies that the volume is at /tmp/data on the the cluster’s Node. The configuration also specifies a size of 10 gibibytes and an access mode of ReadWriteOnce, which means the volume can be mounted as read-write by a single Node.
It defines the StorageClass name manual for the PersistentVolume, which will be used to bind PersistentVolumeClaim requests to this PersistentVolume
