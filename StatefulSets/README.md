## Statefull sets

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
	serviceName: mysql-h
	podManagementPolicy: Parallel # non ordered
```

Scales gracefully
up
`k scale statefulset mysql --replicas=5`
down
`k scale statefulset mysql --replicas=3`

## Headless services

creates dns entries for a pod

headless-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
	ports:
	- port: 3306
	selector:
		app: mysql
	clusterIP: None # this turns the service into a headless service
```

A pod needs to reference the Headless service by using the subdomain attribute with the same name as that of the headless service

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
	labels:
		app: mysql
spec:
	containers:
	- name: mysql
		image: mysql
	subdomain: mysql-h # <- links back to the headless service
	hostname: mysql-pod
```

this will create a dns record for the pod

To put this together in a StatefulSet:

1. define the headless service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
	ports:
	- port: 3306
	selector:
		app: mysql
	clusterIP: None # this turns the service into a headless service
```

2. create a deploymen and be sure to add the serviceName attribute with the same name as that of the headless service : mysql-h

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-ss
	labels:
		app: mysql
spec:
	serviceName: mysql-h
	replicas: 3
	matchLabels:
		app: mysql
	template:
		metadata:
			name: myapp-pod
			labels:
				app: mysql
			spec:
				containers:
				- name: mysql
					image: mysql

```

### Storage in Stateful sets

PV -> PVC -> POD
SC -> PVC -> POD

How to create a PVC for each opd in a stateful set??

You move the whole PVC definition under `volumeClaimTemplate`

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
	serviceName: mysql-h
	volumeClaimTemplates:
	- metadata:
		name: data-volume
	spec:
		accessModes:
			- ReadWriteOnce
		storageClassName: google-storage
		resources:
			requests:
				storage: 500Mi
```

storage class :

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
	name: google-storage
provisioner: kubernetes.io/gce-pd
```
