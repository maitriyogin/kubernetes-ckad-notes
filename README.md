# kubernetes-ckad-notes

Kubernetes CKAD certification notes

## Kubectl

kubectl run nginx --image nginx

kubectl get pods

kubectl describe pod myapp-pod

## Yaml

uses yaml as input for object creation

```yaml
apiVersion: v1
kind: Pod | Service | ReplicSet | Deployment
metadata:
	name: myapp-pod
	labels:
		app: myapp
spec:
	containers:
		- name: nginx-container
		  image: nginx
```

`kubectl create -f pod-definition.yml`

## Labs

### Pods

`kubectl get pods`
`kubectl run nginx --image=nginx `
`kubectl describe pod newpods-lzztc`
`kubectl get pods -o wide`
`kubectl delete pod webapp`
`kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yml`

```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: redis
name: redis
spec:
containers:

- image: redis
  name: redis
  resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  status: {}
```

`kubectl create -f redis.yml`

change the image name to redis
`kubectl apply -f redis.yml`

Extracting yaml from an existing pod

`kubectl get pod <pod-name> -o yaml > pod-definition.yaml`
`kubectl edit pod <pod-name>`

## Replicas

`k create -f replicas/rc-defintion.yml`

`k get replicationcontroller`

`k get pods`

`k replace -f replicas/replicaset-defintion.yml`

`k scale --replicas=6 -f replicas/replicaset-definition.yml`
`k scale --replicas=6 replicaset myapp-replicaset`

To create a yml from a replicaset
`kubectl get replicaset myapp-replicaset -o yaml > rs-definition.yaml`

get the yaml from an existing replica set
`k get replicaset new-replica-set -o yaml > rs.yaml`

then replace it
`k get replicaset new-replica-set -o yaml > rs.yaml`

`k scale --replicas=5 replicaset new-replica-set`

or you can run
`k edit rs new-replica-set`

## Deployments

These will create :
Deployement 1-_ Replcasets 1-_ Pods

`k get all`

`k create -f deployment-defintion.yml`

### Formats

`k get pods -o json`
`k get pods -o name`
`k get pods -o wide`
`k get pods -o yaml`

## Namespaces

`k get pods --namespace=kube-system`

to create a pod in another namespace
`k create -f pod-definition.yml --namespace=dev`

```
apiVersion: v1
kind: Namespace
metadata:
	name: dev
```

`k get pods --namespace=dev`

if you want to set a namespace globally then ..
`k config set-context $(kubectl config current-context) --namespace=dev`

`k get pods --all-namespaces`

create a resouce quoata
`k create -f quotas/compute-quota.yml`

create a pod imperitevely
`k run redis --image=redis --namespace=finance`

## Imperetive commands

--dry-run=client will not create resources but will tell you if the resouce can be

-o yaml will output the resource definition in yaml format on the screen

applying a yaml definition
`k apply -f ..`

`k run redis --image redis:alpine --labels=tier=db`

`k create service clusterip redis-service --tcp=6379:8080`

`k create deployment webapp --image=kodekloud/webapp-color --replicas=3`

specify a container port

`k run custom-nginx --image=nginx --port=8080`

create a deployment in a particular namespace, dev-ns
`k create deployment redis-deploy --image=redis --replicas=2 --namespace=dev-ns`

this creates a pod and exposes it's port 80 via a clusterip service

`k run httpd --image=httpd:alpine --port=80 --expose=true`

k get service httpd -o yaml ... looks like

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2023-06-28T14:27:39Z"
  name: httpd
  namespace: default
  resourceVersion: "1706"
  uid: 499f85d8-9e08-43e3-b833-42d7b30cfdc9
spec:
  clusterIP: 10.43.247.233
  clusterIPs:
  - 10.43.247.233
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: httpd
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

## Configuration

### Docker

`docker run ubuntu`
`docker ps -a`

Containers are meant to run a specific task or service, once the task completes the container exits.

#### Default command defines the process to run like `CMD ["nginx"]`

The command and it's parameters must be seperate in the list
CMD["sleep", "5"] not CMD["sleep 5"]

in a pod definition :
the command field overrides the entrypoint in the docker config
args overrides the CMD

ApiVersion: v1
kind: Pod
metadata:
name: ubuntu-sleeper-pod
spec:
containers: - name: ubuntu-sleeper
image: ubuntu-sleeper
command: ["sleeper2.0"] -> overrides ENTRYPOINT
args: ["10"] -> overrides CMD

## Editing Pods and deployements

you can only edit the following at runtime

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations

You can do the following to edit and delete a pod

1. `k edit pod <pod name>`
2. edit uneditable field
3. creates a temp copy of your config /tmp/kubectl-edit-ccvrq.yaml
4. `k delete pod webapp`
5. `k create -f /tmp/kubectl-edit-ccvrq.yaml`

or to extract the pod definition at runtime

1. `k get pod webapp -o yaml > my-new-pod.yaml`
2. `vi my-new-pod.yaml`
3. `k delete pod webapp`
4. `k create -f my-new-pod.yaml`

### Edit deployments

this can be done at anytime and will re-create the pods upon any pod changes.
`k edit deployment my-deployment`

### Env vars

```yaml
ApiVersion: v1
kind: Pod
metadata:
name: ubuntu-sleeper-pod
spec:
containers: - name: ubuntu-sleeper
image: ubuntu-sleeper
command: ["sleeper2.0"]
args: ["10"]
env:
	- name: APP_COLOR
		value: blue
```

### Config maps

Key value pairs in the pod

1. create the config map
2. inject it into your pod

3. `k create configmap --from-literal=<key>=<value>`

#### Imperatively

`k create configmap \
	app-config --from-literal=APP_COLOR=blue`

From a file
`k create configmap \
	app-config --from-file=app_config.properties`

#### Declartively

`k create -f config-map.yml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
	name: app-config
data:
	APP_COLOR: blue
	APP_MODE: prod
```

you can have as many config maps as your like

`k get configmaps`
`k dexcribe configmaps`

injecting them into a pod

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
		envFrom:
			- configMapRef:
					name: app-config <- this is the name of your config map as about

		// or as a single env var
		env:
			valueFrom:
				configMapKeyRef:
					name: app-config
					key: APP_COLOR
```

### Secrets

#### Imperative

`k create secret generic \
	app-secret --from-literal=DB_HOST=mysql
						 --from-literal=DB_USER=root
`

`k create secret generic \
	app-secret --from-file=app_secret.properties`

#### Declarative

secret-data.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
	name:	app-secret
data:
	DB_HOST: mysql
	DB_USER: root
```

`k create -f secret-data.yaml`

must convert plain test to encoded

`echo -n 'mysql' | base64`

`k describe secrets`

decode a secret

echo -n 'bXlzcWw=' | base64 --decode

configure with a pod

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
		// ENV
		envFrom:
			- secretRef:
					name: app-secret <- this is the name of your config map as about

		// single env var
		env:
			valueFrom:
				secretKeyRef:
					name: app-secret
					key: DB_PASSWORDJK

	// Volume
	volumes:
	- name: app-secret-volume
		secret:
			secretName: app-secret
```

## Secrets

This will encode the supersecret to base64
`kubectl create secret generic my-secret --from-literal=key1=supersecret`

echo <val> | base

`ETCDCTL_API=3 etcdctl \
   --cacert=/opt/homebrew/bin/etcd/ca.crt   \
   --cert=/opt/homebrew/bin/etcd/server.crt \
   --key=/opt/homebrew/bin/etcd/server.key  \
   get /registry/secrets/default/my-secret | hexdump -C`

## Docker Security

Host has a namespace and the containers have a namespace
Docker can only see processes within it's own namespace
Docker host has a list of users, can be seen with the commands.
you can change the docker use with
`docker run --user=1000 ubuntu sleep 3600`

this can be set in teh docker file

```Dockerfile
From ubuntu

USER 1000
```

docker build -t my-ubuntu-image
docker run my-ubuntu-image sleep 3600

## Security Contexts

Containers are encapsulated in pods
Sec settings can be set on the pod or container label

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: web-pod
spec:
	securityContext: // pod level
		runAsUser: 1000
	containers:
	- name:	ubuntu
		image: ubuntu
		command: ["sleep", "3600"]
		securityContext: // pod level
			runAsUser: 1000
			capabalities:
				add: ["MAV_ADMIN"]
```

Capabilities are only supported at the container level and not at the POD level

### Understand Service Accounts

User - user acocunts

Serivce Account - account used by an application to communicate with the cluster.
jenkins
promethous
Create a service account

`k create serviceaccount dashboard-sa`

`k get serviceaccount`

a secret is created with the service account under Tokens
`k describe serviceaccount dashboard-sa`

you can use this token as a Bearer when calling rest apis

each namespace has it's own default service account

when creating a pod tokens are generated at /var/run/secrets/kubernetes.io/serviceaccount

`k exec -it mykubernetes-dashboard --ls /var/run/secrets/kubernetes.io/serviceaccount`

to view the token
`k exec -it my-kubernetes-dashboard cat /var/run/secrets/kubernetes.io/serviceaccount/token`

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: my-kubernetes-dashboard
spec:
	securityContext: // pod level
		runAsUser: 1000
	containers:
	- name:	my-kubernetes-dashboard
		image: my-kubernetes-dashboard
	serviceAccountName: dashboard-sa // newly created service account
```

you can change this in a deployment but you'll need to recreate the pod otherwise.

every namespace has a default service account

#### TokenRequestAPI

Audience bound
Time bound
Object bound

1.24 doesn't automatically create a secret -> token, you have to do this yourself with
`k create serviceaccoun dashboard-sa`
`k create token dashboard-sa`

if you want the definitions to be generated the old way

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: my-kubernetes-dashboard

	annotations:
		kubernetes.io/service-account.name: dashboard-sa
```

## Taints and tolerations

###

`k taint nodes node-name key=value:taint_effect`

taint_effect = NoSchedule | PreferNoSchedule | NoExecute

`k taint nodes node1 app=blue:NoSchedule`

```
apiVersion: v1
kind: Node
metadata:
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: node01
    kubernetes.io/os: linux
  name: node01
spec:
  taints:
  - effect: NoSchedule
    key: spray
    value: mortein
```

### Tolerations

are set on the pod level

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: my-kubernetes-dashboard

spec:
	containers:
	tolerations:
		key: "app"
		operator: "Equal"
		value: "blue"
		effect: NoSchedule
```

The master node is just a node but has a taint restricting all other pods from running on it
`k describe node kubemaster | grep Taint`

## Node Selectors

Can set a limition on the pods so they only run on specifc nodes.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod

spec:
	containers:
	- name: data-processor
		image: data-processor
	nodeSelector:
		size: Large // name of node, you must have first labeled your node

```

label the node
`k label nodes <node-name> <label-key>=<label-value>`
`k label nodes node-1 size=Large` // size large label

### Node Affinity

For more complex orchestration of where the pods should be run, node Affinity solves the issue.
This is becuase you can't provide advanced selectors

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod

spec:
	containers:
	- name: data-processor
		image: data-processor
	affinity:
		nodeAffinity:
			requireDuringSchedulingIgnoredDuringExection:
				nodeSelectorTerms:
				- matchExoressions:
					- key: size
						operator: In // NotIn | Exists ( doesn't check the values )
						values:
						- Large // large only
						- Medium // large or medium
```

What if there are no nodes with label Large ??
this is answered by the long sentances under nodeAffinity
requireDuringSchedulingIgnoredDuringExection
preferredDuringSchedulingIgnoredDuringExection

- DuringScheduling
- DuringExecution
