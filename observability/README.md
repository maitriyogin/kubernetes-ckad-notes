## POD Status

- pod statuses : pending ...
- pod conditions :
  - PodScheduled
  - Initialised
  - ContainersReady
  - Ready

## Knowing when the pod is ready

### Readiness port

test `/api/ready`

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
		image: simple-webapp-color
		ports:
			- containerPort: 8080
		readinessProbe:
			httpGet:
				path: /app/ready
				port: 8080
			# with delay
			initialDelaySeconds: 10
			periodSeconds: 5
			failureThreshold: 8
			# tcpSocket
			tcpSocket:
				port: 3306
			# command
			exec:
				command:
					- cat
					- /app/is_ready
```

## Liveness probes

Can be configured on the container to periodically test that the app is Ready or up and running

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
		image: simple-webapp-color
		ports:
			- containerPort: 8080
		livenessProbe:
			httpGet:
				path: /api/healthy
				port: 8080
		# with delay
		initialDelaySeconds: 10
		periodSeconds: 5
		failureThreshold: 8
		# tcpSocket
		tcpSocket:
			port: 3306
		# command
		exec:
			command:
				- cat
				- /app/is_ready
```

## Container logging

### Docker

`docker run -d kodekloud/even-simulator` <- no logs
`docker logs -f ecf` <- logs

### Kubernetes

`k create -f event-simulator.yaml`
`k logs -f event-simulator-pod`

if there are multiple containers then you must specify a container name when logging.

`k logs -f event-simulator-pod event-simulator``

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: event-simulator-pod
spec:
	containers:
	- name: event-simulator
		image: kodekloud/event-simulator
	- name: image-processor
		image: some-image-processor
```

## Monitor

- number of nodes in the cluster, how many are helpfull, number of pods, cpu, mem consumption ...

- metrics server
- prometheus
- elastic stack
- datadog
- dynatrace

### Metrics server - Heapster ( deprecated )

- one metrics server per kubernetes cluster
- in memory monitoring solution

### How are the metrics generated

    - kubelet
    	- cAdvisor - retrieves performance metrics from the pod

    - minikube `minikube addons enable metrics server`
    - others `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

    	- `git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git`
    	- `kubectl create -f .` // from within the kubernetes-sig dir

#### To get metrics

    `k top node`
    `k top pods`
