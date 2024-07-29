## Docker images

1. how to create your own image
2. what are containerizing

Steps to creating an image

1. OS -Ubuntu
2. Update apt repo
3. Install deps using apt
4. Instal python deps using pip
5. copy source code to /opt folder
6. run the web server using flask command

create a docker file in your project dir

1.

```docker
FROM node:alpine # based on another image

WORKDIR /app
COPY package.json ./ # copies local files to the docker image
RUN npm install
COPY ./ ./

CMD ["npm", "start"]
```

2. build the image
   `docker build Dockerfile -t swhite/my-app`

   `docker build -t webapp-color .`

3. push the image to your local repo
   `docker push swhite/my-app`

each step is another layer, these can be rolled back or improves rebuild

### What can you containerize

run the container, remap the container port to another on the host

publishes port 8080 from the container on port 8282 of the host
`docker run -p 8282:8080 webapp-color`

get the base image

`docker run python:3.6 cat /etc/*release*`

tagging the docker image
`docker build -t webapp-color:lite .`

## Security Primitives

### Secure Hosts

kubernetes related security

kube-apiserver - first line of defence

1. who can access the server
   username passwords
   username tokens
   service accounts

2. by default all pods can access each other in a cluster

## Authentication

1. Users

- Amins
- Develoers
- Application End Users
- Bots

### Accounts

- Users
- Service Accounts

Cannot create users in cluster
You can create service account
`k create serviceaccount sa1`

All traffic goes through the kubeapi server

1. Authenticates request
2. handles the request

##### Auth with kube-apiserver

kube-apiserver

- Static Password File
- Static Token File
- Certificates
- Identity Services

###### Basic Auth

user-details.csv
red,stephen,u0001,group1
blue,frank,u0002, group1

then in kube-apiserver.yaml

```yaml
spec:
	containers:
	- command:
	- --basic-auth-file=user-details.csv
```

then to access
`curl -v -k https://master-node-ip:6443/api/v1/pods -u "stephen:red`

###### Token Auth

user-token-details.csv
<token1>,stephen,u0001,group1
<token2>,frank,u0002, group1

then in kube-apiserver.yaml

```yaml
spec:
	containers:
	- command:
	- --token-auth-file=user-token-details.csv
```

then to access
`curl -v -k https://master-node-ip:6443/api/v1/pods -header "Authorization: Bearer <token1>`

these methods are not recommended
Setup role based authorization
look to mount a volume for file based authentication

## Kubeconfig

`curl -v -k https://master-node-ip:6443/api/v1/pods \
	--key admin.key
	--cert admin.crt
	--cacert ca.crt
	`

````k get pods
	--server my-kube-playground:6443
	--client-key admin.key
	--client-certificate admin.crt
	--certificate-authority ca.crt
	```

the above can be moved into a kubeconfig file:

$HOME/.kube/config <- needs to live here

```kubeconfig
	--server my-kube-playground:6443
	--client-key admin.key
	--client-certificate admin.crt
	--certificate-authority ca.crt
````

3 sections

1. Clusters
   dev
   prod
   google
2. Contexts
   maps users to clusters
3. Users
   Admin
   dev
   prod
   MyKubeAdmin

custom kube config

```yaml
apiVersion: v1
kind: Config

current-context: dev-user@google

clusters:
  - name: my-kube-playground
  - name: dev
  - name: prod
  - name: google

contexts:
  - name: my-kube-admin@my-kube-playground
  - name: dev-user@google
  - name: prod-user@prod

users:
  - name: my-kube-admin
  - name: admin
  - name: dev-user
  - name: prod-user
```

`k config view`

```yaml
apiVersion: v1
kind: Config

current-context: kubernetes-admin@kubernetes

clusters:
- cluster:
	certificate-authority-data: REDACTED
	server: http://172/17/0/5:6443
	name: kubernetes

contexts:
- context: kubernetes
		cluster: kubernetes
		user: kubernetes-admin
	name: prod-user@prod

users:
- name: kubernetes-admin
	user:
		client-certificate-data: REDACTED
		client-key-data: REDACTED
```

you can also pass in a file
`k config view --kubeconfig=my-custom-config`

COMMAND change the context
`k config use-context prod-user@production`

### Lab

```yaml
apiVersion: v1
clusters:
  - cluster:
      certificate-authority-data: DATA+OMITTED
      server: https://controlplane:6443
    name: kubernetes
contexts:
  - context:
      cluster: kubernetes
      user: kubernetes-admin
    name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
  - name: kubernetes-admin
    user:
      client-certificate-data: DATA+OMITTED
      client-key-data: DATA+OMITTED
```

```yaml
apiVersion: v1
kind: Config

clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: development
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: kubernetes-on-aws
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: test-cluster-1
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

contexts:
- name: test-user@development
  context:
    cluster: development
    user: test-user

- name: aws-user@kubernetes-on-aws
  context:
    cluster: kubernetes-on-aws
    user: aws-user

- name: test-user@production
  context:
    cluster: production
    user: test-user

- name: research
  context:
    cluster: test-cluster-1
    user: dev-user

users:
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
- name: dev-user
  user:
		client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

current-context: research
preferences: {}
```

## Api Groups

`curl https://kube-master:6443/version`

`curl https://kube-master:6443/version/api/v1/pods`

apis:

- /metrics
- /healthz
- /version
- /api
- /apis
- /logs

apis for cluster functionality

- core
  - /api

-named - apis

To list api groups
`curl http://localhost:6443 -k`

`curl http://localhost:6443 -k | grep "name"`

Apis are grouped under:

- api groups
  - Resources
    - Verbs

e.g.

- /apis # api groupd
  - /v1
    # resources
    - /deployments
    - /replicasets
    - /statefulsets # Verbs - list - get - create - delete - update - watch

### Authorization

#### Node

    Requests to the node are handled by the node authorizer

#### ABAC

    access based
    Create a policy file

#### RBAC

    Define roles for say a developer

#### Webhook

    manage auth externally
    Open policy agent

#### AlwaysAllow

#### AlwaysDeny

--authorization-mode=AlwaysAllow | AlwaysDeny

### RBAC role based access control

developer-role.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
	name: developer
rules:
- apiGroups: [""]
	resources: ["pods"]
	verbs: ["list", "get", "create", üpdate", "delete"]
- apiGroups: [""]
	resources: ["ConfigMap"]
	verbs: ["create"]
```

link the user to the role

devuser-developer-binding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
	name: devuser-developer-binding
subjects:
- kind: User
	name: dev-user
	apiGroupd: rbac.authorization.k8s.io
roleRef:
	kind: Role
	name: developer
	apiGroup: rbac.authorization.k8s.io
```

`k get roles``

`k get rolebindings``

`k describe role developer``

#### Check Access

`k auth can-i create deployments`
`k auth can-i delete nodes`

`k auth can-i create deployments --as dev-user`
`k auth can-i delete nodes --as dev-user`

COMMANDS

```
k describe pod kube-apiserver-controlplane -n kube-system
```

`k get roles --all-namespaces`
`k describe role kube-proxy -n kube-system`
`k get role kube-proxy -n kube-system -o wide`
`k describe rolebinding kube-proxy -n kube-system`

`k auth can-i list pods --as dev-user`

Create role and bindings for a dec user

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: ['']
    resources: ['pods']
    verbs: ['list', 'get', 'create', 'delete']
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
subjects:
  - kind: User
    name: dev-user
    apiGroupd: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

Create deployments

```yaml
apiVersion: v1
items:
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      creationTimestamp: '2023-08-11T07:54:21Z'
      name: developer
      namespace: blue
      resourceVersion: '743'
      uid: dbe74d27-bc39-4467-8a52-584fa15a55b1
    rules:
      - apiGroups:
          - 'apps'
        resources:
          - deployments
        verbs:
          - create
      - apiGroups:
          - ''
        resourceNames:
          - blue-app
        resources:
          - pods
        verbs:
          - get
          - watch
          - create
          - delete
      - apiGroups:
          - ''
        resourceNames:
          - dark-blue-app
        resources:
          - pods
        verbs:
          - get
          - watch
          - create
          - delete
kind: List
metadata:
  resourceVersion: ''
```

## Cluster Roles

Roles are created within namespaces, which can be used to group resources

Nodes are cluster wide!

Cluster Scope

- nodes
- PV
- clusterroles
- clusterrolebindings
- certificatesigningrequests
- namespaces

To get a complete list of non namespaced resources

`k api-resources --namespaced=true`
`k api-resources --namespaced=false`

cluster roles

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
  - apiGroups: ['']
    resources: ['nodes']
    verbs: ['list', 'get', 'create', 'delete']
```

link the user to that cluster role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
  - kind: User
    name: cluster-admin
    apiGroupd: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

COMMAND

to see all cluster roles

wc -l will count the amount of lines ..
`k get clusterroles --no-headers  | wc -l`

`k get clusterrolebinding cluster-admin -o yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: michelle-storage-admin
subjects:
  - kind: User
    name: michelle
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: storage-admin
  apiGroup: rbac.authorization.k8s.io
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: 'true'
  labels:
    kubernetes.io/bootstrapping: rbac-storage
  name: storage-admin
rules:
  - apiGroups:
      - ''
    resources:
      - 'persistentvolumes'
      - 'storageclasses'
    verbs:
      - '*'
```
