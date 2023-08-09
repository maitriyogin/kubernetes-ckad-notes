## Services

enable loose coupling of microservices within an application

### External communication

A service is an object, it forwards requests from a pod port to an external port on the node

### Services Types

- NodePort
- ClusterIP
- LoadBalancer

### NodePort

1. Pod port 80 -> target port
2. Service 80 -> to nodeport
3. NodePort 30008

service-definition.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
	type: NodePort
	ports:
		- targetPort: 80 # 1 port on the pod
			port: 80 # 2 port on the service
			nodePort: 30008 # 3 port on the nodeport

```

user labels and selectors to connect pods to their services

service-definition.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
	type: NodePort
	ports:
		- targetPort: 80 # 1 port on the pod
			port: 80 # 2 port on the service
			nodePort: 30008 # 3 port on the nodeport
	selector:
		app: myapp
		type: front-end
```

pod-definition.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
	labels:
		app: myapp
spec:
	containers:
	- name: nginx-container
	image: nginx
```

`k create -f service-definition.yaml`
`k get services`

#### Multiple pods

Connection is achieved via the pods labels and the services selector

## Ingress

MySql (ClusterIp) -> (deployment) -> service (NodePort:38080) -> proxy-server:80

you can use LoadBalancer instead of NodePort

Ingress is a layer 7 LoadBlancer offering SSL and agregated load balancer
Not default in k, you need to deploy one

1. Deploy
2. Configure

Objects needed for the ingress controller

1. Deployment
2. Service
3. ConfigMap
4. Auth

Ingress resources - used for rerouting

ingress-wear.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
	backend:
		serviceName: wear-service
		servicePort: 80
```

### split by path

Route to different services based on the paths

ingress-wear-watch.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
	rules:
	- http:
		paths:
		- path: /wear
			backend:
				service:
					name: wear-service
				 	port: 80
		- path: /watch
			backend:
				service:
					name: watch-service
					port: 80
```

`k describe ingress ingress-wear-watch`

#### Split by domain name

ingress-wear-watch.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
	rules:
	- host: wear.my-online-store.com
		http:
			paths:
				backend:
				service:
					name: wear-service
				 	port: 80
	- host: watch.my-online-store.com
		http:
			paths:
				backend:
				service:
					name: watch-service
					port: 80
```

Now, in k8s version 1.20+ we can create an Ingress resource from the imperative way like this:-

Format - kubectl create ingress <ingress-name> --rule="host/path=service:port"

Example - kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear\*=wear-service:80"

Find more information and examples in the below reference link:-

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-

References:-

https://kubernetes.io/docs/concepts/services-networking/ingress

https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types

#### Commands

name of ingress
`k get ingress --all-namespaces`
this will also tell us which namespace the ingress is in

`k describe ingress -n app-space`

create service accounts
`kubectl create serviceaccount ingress-nginx --namespace ingress-nginx`

````yaml
apiVersion: v1
kind: ServiceAccount
metadata:
 creationTimestamp: "2023-08-09T13:14:44Z"
 name: ingress-nginx-admission
 namespace: ingress-nginx
 resourceVersion: "1458"
 uid: 6b1d0c98-425e-4018-b910-0a18b69bc35c
	```

```yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    creationTimestamp: "2023-08-09T13:09:48Z"
    name: default
    namespace: ingress-nginx
    resourceVersion: "1071"
    uid: 37baad9f-878e-4173-ba6b-708b3e82225d
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    creationTimestamp: "2023-08-09T13:14:13Z"
    name: ingress-nginx
    namespace: ingress-nginx
    resourceVersion: "1417"
    uid: e20a1167-480a-4ed5-9c4f-82289f69da79
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    creationTimestamp: "2023-08-09T13:14:44Z"
    name: ingress-nginx-admission
    namespace: ingress-nginx
    resourceVersion: "1458"
    uid: 6b1d0c98-425e-4018-b910-0a18b69bc35c
kind: List
metadata:
  resourceVersion: ""
	```
````

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /wear
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port:
                  number: 8080
          - path: /watch
            pathType: Prefix
            backend:
              service:
                name: video-service
                port:
                  number: 8080
```

## Traffic

1. Ingress 80 - nginx
2. Egress 5000 ->
3. Ingress 5000 <- api
4. Egress 3306 ->
5. Ingress 3306 <- db

## Network security

Pods should be able to communicate to each other, by default

All Allow rule within the cluster

### Network policy

link a policy to a pod

1. nginx 80 - nginx
2. api 5000 ->
3. db 3306 ->

allow Ingress Traffic from api pod on port 3306

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
	podSelector:
		matchLables:
			role: db
	policytypes:
	- Ingress
	ingress:
	- from:
		- podSelector:
			matchLabels:
				name: api-pod
		ports:
		- protocol: TCP
		port: 3306
```

### Namespace selector

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
	podSelector:
		matchLables:
			role: db
	policytypes:
	- Ingress
	- Egress
	ingress:
	- from:
		# rule 1
		- podSelector:
				matchLabels:
					name: api-pod
			# AND
			namespaceSelector:
				matchLabels:
					name: prod
		#- namespaceSelector: by using the dash we then have an OR
		#		matchLabels:
		#			name: prod
		# OR
		# if it's an external resource you can use an ipBlock
		# rule 2
		- ipBlock:
			cidr: 192.168.5.10/32 # ip of say a backup server
		ports:
		- protocol: TCP
		port: 3306
	egress: # communication from the db pod to the backup server
	- to:
		- ipBlock:
			cidr: 192.168.5.10/32
		ports:
		- protocol: TCP
			port: 80
```

```yaml
apiVersion: v1
items:
  - apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: payroll-policy
    spec:
      ingress:
        - from:
            - podSelector:
                matchLabels:
                  name: internal
          ports:
            - port: 8080
              protocol: TCP
      podSelector:
        matchLabels:
          name: payroll
      policyTypes:
        - Ingress
```

To allow traffic from the Internal app to the payroll service and db-service

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  egress:
    - to:
        - podSelector:
            matchLabels:
              name: payroll
      ports:
        - port: 8080
          protocol: TCP
    - to:
        - podSelector:
            matchLabels:
              name: mysql
      ports:
        - port: 3306
          protocol: TCP
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
    - Egress
```
