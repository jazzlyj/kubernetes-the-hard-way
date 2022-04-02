# Persistent Storage

In this lab you will complete a series of tasks to ensure setup persistent storage using an NFS share.

## Prerequisites
* NFS Server IP = asdfasdf
* NFS Share = /asdfasdf
* K8s Cluster = One master and two worker Nodes

Note: Make sure NFS server is reachable from worker nodes and try to mount nfs share on each worker once for testing.
See [NFS](https://github.com/jazzlyj/procedures/blob/main/nfs.md)



## Configure NFS based PV (Persistent Volume)
* Create a file
```
echo "Hello, NFS Storage NGINX" > /opt/k8s-pods/data/index.html
```

### Create persistent volume (pv) yaml
Create the pv `vi nfs-pv.yaml`

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: asdfasdf
    server: asdfasdf

```

### Create the pv

```
kubectl create -f nfs-pv.yaml
```

> output

```
persistentvolume/nfs-pv created
```


* Verify
Run following kubectl command to verify the status of persistent volume:
```
kubectl get pv
```

> output

```
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
nfs-pv   100Gi       RWX            Recycle          Available           nfs                     20s
```



### Configure Persistent Volume Claim (pvc)
Mount persistent volume inside a pod, we have to specify its persistent volume claim
`vi nfs-pvc.yaml`
```

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi

```


### Create the pvc
```
kubectl create -f nfs-pvc.yaml
```



> output

```
persistentvolumeclaim/nfs-pvc created
```


### Verify the control pane bound a volume to the claim requirement
```
kubectl get pvc nfs-pvc
```

> output

```
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs-pvc   Bound    nfs-pv   100Gi      RWX            nfs            14s
```


```
kubectl get pvc nfs-pv
```

> output
```
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
nfs-pv   100Gi      RWX            Recycle          Bound    default/nfs-pvc   nfs                     3m11s
```


## Use NFS based Persistent Volume inside a Pod
Create a nginx pod using beneath yaml file, it will mount persistent volume claim on 
`/usr/share/nginx/html`. Use the following command `vim nfs-pv-pod` and add the lines:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pv-pod
spec:
  volumes:
    - name: nginx-pv-storage
      persistentVolumeClaim:
        claimName: nfs-pvc
  containers:
    - name: nginxNFStest
      image: nginx
      ports:
        - containerPort: 80
          name: "nginx-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: nginx-pv-storage

```


### Create the pod
```
kubectl create -f nfs-pv-pod.yaml
```

* get status of pod
```
kubectl get pod nginx-pv-pod -o wide
```

> output

```

```


### Try accessing the pod
```
curl http://
```





## Credit due:
https://www.linuxtechi.com/configure-nfs-persistent-volume-kubernetes/


Next: [Title](file.md)
