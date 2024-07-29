##Security
###Authentication
###Accounts
All user access is managed by kube api server

Article on Setting up Basic Authentication
Setup basic authentication on Kubernetes (Deprecated in 1.19)
Note: This is not recommended in a production environment. This is only for learning purposes. Also note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases
Follow the below instructions to configure basic authentication in a kubeadm setup.

Create a file with user details locally at /tmp/users/user-details.csv

# User File Contents

password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
password123,user4,u0004
password123,user5,u0005
Edit the kube-apiserver static pod configured by kubeadm to pass in the user details. The file is located at /etc/kubernetes/manifests/kube-apiserver.yaml

apiVersion: v1
kind: Pod
metadata:
name: kube-apiserver
namespace: kube-system
spec:
containers:

- command:
  - kube-apiserver
    <content-hidden>
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
    volumeMounts:
  - mountPath: /tmp/users
    name: usr-details
    readOnly: true
    volumes:
  - hostPath:
    path: /tmp/users
    type: DirectoryOrCreate
    name: usr-details
    Modify the kube-apiserver startup options to include the basic-auth file

apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
name: kube-apiserver
namespace: kube-system
spec:
containers:

- command: - kube-apiserver - --authorization-mode=Node,RBAC
  <content-hidden> - --basic-auth-file=/tmp/users/user-details.csv
  Create the necessary roles and role bindings for these users:

---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
namespace: default
name: pod-reader
rules:

- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---

# This role binding allows "jane" to read pods in the "default" namespace.

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
name: read-pods
namespace: default
subjects:

- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
  roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
  Once created, you may authenticate into the kube-api server using the users credentials

curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"

###Kube config
$HOME/.kube/config
--server my-kube-0pllqaygrounda:6443
--client-key aqdmin.key
--client-certificate admin.crt
--certificate-aquthority ca.crt

`k get pods`
Clusters
--server my-kube-playground:6443
development
production
google

Contexts
linkbetween my-kube-\playground and admin in

user
admin
dev

User
admin:

```
--client-key aqdmin.key
--client-certificate admin.crt
--certificate-aquthority ca.crt
```

admin
dev
produser

Roles:
RBAC - role based authorization

Role and then bind a user with a role binding

developer-role.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: ['']
    resources: ['pods']
    verbs: ['list', 'get', 'create', 'update', 'delete']
  - apiGroups: ['']
    resources: ['ConfigMap']
    verbs: ['create']
```

devuser-developer-binding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io <-- this is the account
roleRef
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

k create -f devuser-developer-binding.yaml

k get roles
k get rolebindings

k describe rolebinding <name>

Checking to see what access you have
k auth can-i delete nodes
k auth can-i create deployments
k auth can-i create deployments --as dev-user -n namespace

you can restrict access to specific resources like pods by providing the resourceNames field

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: ['']
    resources: ['pods']
    verbs: ['list', 'get', 'create', 'update', 'delete']
    resourceNames: ['blue', 'orange']
  - apiGroups: ['']
    resources: ['ConfigMap']
    verbs: ['create']
```

Authorization modes

k cluster-info dump | grep authorization-mode

## Cluster namespaces

k api-resources --namespaced=true

Cluster roles and binding

YOu first create a cluster role and then a cluster role binding to connect a user to that role

cluster-admin-role.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
  - apiGroups: ['']
    resources: ['nodes']
    verbs: ['list', 'get', 'create', 'update', 'delete']
```

cluster-admin-binding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-rolebinding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io <-- this is the account
roleRef
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

A cluster role for all pods in the cluster

to get the amount or count of something you can ..

k get clusterrolebindings -o name | wc -l

# Admission Controllers

k exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins

kube-apiserver.service
NamespaceAutoProvision - not on by default
--enable-admission-plugins=NodeRestriction, NamespaceAutoProvision
--disable-admission-plugins=AlwaysPullImages

k run nginx --image nginx --namespace blue

/etc/kubernetes/manifest/kube-apiserver.yaml

systemctl restart kube-apiserver.service

### RESTART the kube-apiserver

```
k -n kube-system delete pod -l 'component=kube-apiserver'
```

Note that the NamespaceExists and NamespaceAutoProvision admission controllers are deprecated and now replaced by NamespaceLifecycle admission controller.

The NamespaceLifecycle admission controller will make sure that requests
to a non-existent namespace is rejected and that the default namespaces such as
default, kube-system and kube-public cannot be deleted.

Since the kube-apiserver is running as pod you can check the process to see enabled and disabled plugins.
ps -ef | grep kube-apiserver | grep admission-plugins

k describe pod kube-apiserver-controlplane -n kube-system | grep admission

## Validatin and mutation admission controllers

```
apiVersion
kind
metadata
  name
webhooks
- name
  clientConfig
    service
      namespace
      name
    caBundle
  rules
  - apiGroups
    apiVersion
    operations
    resources
    scope
```

### Create Secrete TLS

kubectl -n webhook-demo create secret tls webhook-server-tls \
 --cert "/root/keys/webhook-server-tls.crt" \
 --key "/root/keys/webhook-server-tls.key"

kubectl get po pod-with-defaults -o yaml | grep -A2 " securityContext:"

# API Versions

apis
app
v1
v1alpha1
v1beta1
extensions
networking.k8s.io
v1
v1alpha1
storageversion

k explain deployment

Defualt api version can be set in the kub-apiserver under :

```
--runtime-configl=batch/v2alpha1\\
```

API elements may only be removed by incrementing the version of the APPI group.:
v1alpha1
v1alpha2

Resource

```
kind: Global
apiVersion: traffic.controller/v1
metadata:
  name: datacenter
spec:
  dataField: 2
  access: true
```
