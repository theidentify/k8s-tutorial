# Explained
## How to persist data in k8s using volumes?
- Persistent Volume
- Persistent Volume Claim
- Storage Class


## The need for volumes
- Is to persist data between the pod restart

## Storage Requirements
1. Storage that doesn't depend on the pod lifecycle
2. Storage must be available on all nodes
3. Storage needs to survive even if cluster crashes.

## Persistent Volume
- A cluster resoure
  - ex. CPU RAM
- create via YAML file
  - kind: PersistentVolume
  - spec: e.g. how much storage?
- Needs actual physical storage external plugin to your cluster, like local disk, nfs server, cloud-storage
- You need to create and manage them by yourself
  
### Persistent Volume YAML Example
- Use that physical storages in the spec section
- Depending on storage type, spec attributes differ
  
```yaml
# NFS Storage
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-name
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.0
  nfs:
    path: /dir/path/on/nfs/server
    server: nfs-server-ip-address

# Google Cloud
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-volume
  labels:
    failure-domain.beta.kubernetes.io/zone: us-central1-a__us-central1-b
spec:
  capacity:
    storage: 400Gi
  accessModes:
  - ReadWriteOnce
  gcePersistentDisk:
    pdName: my-data-disk
    fsType: ext4

# Local storage
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity: 
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

## Persistent Volumes are NOT namespaced
- OV outside of the namespaces
- Accessible to the whole cluster

## Local vs Remote Volume Types
- Each volume type has it's own use case!
- Local volume types violate 2. and 3. requirement for data persistence:
  1. Being tied to 1 specific node
  2. Surviving cluster crashes
- For DB persistence use remote storage!

## K8s Administrator and K8s User
1. [K8sAdmin] sets up and maintains the cluster
   1. Storage resource is provisioned by Admin
   2. creates the PV components from these storage backends
2. [K8sUser] deploys application in cluster
   1. Setup application to claim the Persistent Volume in YAML 


## Persistent Volume Claim component
- Claims must be in the same namespace
  
```yaml
# Claim-file.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-name
spec:
  storageClassName: manual
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

--
# Podfile.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMountes:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        clainName: pvc-name
```
   
## Levels of Volumes abstractions
1. Pod requests the volume through PV Claim
2. Claim tries to find a volume in cluster
3. Volume has the actual storage backend

## Why so many abstractions?
In design of this Admin provisions storage resource, User creates claim to PV
- Data should be safely stored
- Don't want to set up the actual storages
- Easier for developer

## Remark to ConfigMap and Secret
- ConfigMap and Secret is in local volumes
- not created via PV and PVC
- managed by K8s
- In case, Configuration file for your pod or Certificate file for your pod, You need the file to available to your pod as follow steps:
  1. Create ConfigMap and/or Secret component
  2. Mount that into your pod/container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: busybox-container
      image: busybox
      volumeMounts:
        - name: config-dir
          mountPath: /etc/config
    volumes:
      - name: config-dir
        configMap:
          name: bb-configmap

```

## Storage Class
- SC Provisions Persistent Volumes dynamically when PersistentVolumeClaim claims it
- StorageBackend is defined in the SC component via "provisioner" attribute each storage backend has own provisioner
  - internal provisioner = "kubernetes.io"
  - external provisioner
  - configure parameters for storage we want to request to PV
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-class-name
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4

--
# PVC Config
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: storage-class-name
```

## Storage Class Usage
1. Pod claims storage via PVC
2. PVC requests storage from SC
3. SC creates PV that meets the needs of the Claim