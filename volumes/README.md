## Volumes

pods are transient in nature

Volumes need a storage

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: random-number-generator
spec:
	containers:
	- image: alpine
	name: alpine
	command: ["/bin/sh", "-c"]
	args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]

	volumes:
	- name: data-volume
		hostPath:
			path: /data
			type: Directory
	- name: data-volume-elastic-block-store
		awsElasticBlockStore:
			volumeID: <volume-id>
			fsType: ext4
```

### Persistence volumes

cluster wide pool of persistant volumes

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
	name: pv-voll
spec:
	accessModes:
		- ReadWriteOnce | ReadWriteOnce | ReadWriteMany
	capacity:
		storaget: 1Gi
	hostPath:
		path: /tmp/data
	# another store can be configured like the awsebs
	awsElasticBlockStore:
		volumeID: <volume-id>
		fsType: ext4
```

Volume type can be set to emptyDir, data is only stored when the pod is running and removed when the pod stops
`k get persistentvolume`

### Perisistent Volume Claims

Makes a persistent volume available to a pod

use selectors to match the PVC to a PV

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
	name: myclaim
spec:
	accessModes:
		- ReadWriteOnce | ReadWriteOnce | ReadWriteMany
	resources:
		requests:
			storage: 1Gi
```

`k get persistentvolumeclaim`

On Deletion of the PVC you can specify if the PV should be retained:
`persistentVolumeReclaimPolicy: Retain | Delete | Recycle`

### Lab

Executing commands in a pod

`k exec webapp -- cat /log/app.log`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
  persistentVolumeReclaimPolicy: Retain
```

Connect the PVC to a pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
      env:
        - name: LOG_HANDLERS
          value: file
      volumeMounts:
        - mountPath: /log
          name: log-volume

  volumes:
    - name: log-volume
      persistentVolumeClaim:
        claimName: claim-log-1
```

### Storage Classes

#### Lab

1. Storage Class local storage

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"local-storage"},"provisioner":"kubernetes.io/no-provisioner","volumeBindingMode":"WaitForFirstConsumer"}
  name: local-storage
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

2. Persisent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 500Mi
```

VolumeBindingMode

3. Connect PVC to Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: webapp
      image: nginx:alpine
      volumeMounts:
        - mountPath: /var/www/html
          name: local-pvc

  volumes:
    - name: local-pvc
      persistentVolumeClaim:
        claimName: local-pvc
```

#### Delayed storage class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```
