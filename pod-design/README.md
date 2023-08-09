## Pod design

### Labels Selectors

used to filter and find the correct pods

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		# add as mnay key value labels as you like
		name: simple-webapp-color
		app: App1
		function: front-end
spec:
	containers:
	- name: simple-webapp-color
		image: simple-webapp-color
		ports:
			- containerPort: 8080
```

to filter on label
`k get pods --selector app=App1`

### Replica sets

```yaml
apiVersion: v1
kind: ReplicaSet
metadata:
	name: simple-webapp
	labels:
		# add as mnay key value labels as you like
		app: App1
		function: front-end
spec:
	replicas: 3
	selector:
		matchLabels:
			# this connects the replica set to the pods with the same label
			# this can also be a service
			app: App1
	template:
		metadata:
			labels:
				app: App1
				function: front-end
		spec:
			containers:
			- name: simple-webapp
				image: simple-webapp
```

### Annotations

```yaml
apiVersion: v1
kind: ReplicaSet
metadata:
	name: simple-webapp
	labels:
		# add as mnay key value labels as you like
		app: App1
		function: front-end
	annotations:
		buildversion: 1.34
spec:
	replicas: 3
	selector:
		matchLabels:
			# this connects the replica set to the pods with the same label
			# this can also be a service
			app: App1
	template:
		metadata:
			labels:
				app: App1
				function: front-end
		spec:
			containers:
			- name: simple-webapp
				image: simple-webapp
```

#### Selection

`k get pods -l bu=finance,env=prod,tier=frontend`

## Deployments

### Rollout command

`k rollout status deploymen/myapp-deployment`
`k rollout history deploymen/myapp-deployment`

### Deployment Strategies

- destroy and then redeploy - recreate
- recreate one by one - rolling update - default deployment strategy

to apply the deployment changes
`k apply -f deployment-definition.yaml`
`k set image deployment/myapp-deployment nginx-container=nginx:1.9.1`

`k describe deployment myapp-deployment`

### Upgrades

`k get replicasets`

if something goes wrong then you can rollback ...

`k rollout undo deployment/myapp-deployment`

#### Summary of commands

- create
  `k create -f deployment-definition.yaml`
- get
  `k get deployments`
- update
  `k apply -f deployment-definition.yaml`
  `k set image deployment/myapp-deployment nginx-container=nginx:1.9.1`
  `k set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2`
- status
  `k rollout status deployment/myapp-deployment`
  `k rollout history deployment/myapp-deployment`
- rollback
  `k rollout undo deployment/`

### JOBS

The default behaviour of a pod is to always recreate the pod.

RestartPolicy
`restartPolicy` : Never | Always | Failure

ReplicaSet - ensures that a number of pods are running at all times
Jobs - ensures a set of pods will run a specific task to completion

#### Create Job

#### job-definition.yaml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
	name: math-add-job
spec:
		containers:
		- name: math-add
			image: ubuntu
			command: ['expr', '3', '+', '2']
		restartPolicy: Never
```

`k create -f job-definition.yaml`

`k get jobs`

`k get pods`

to see the output

`k logs math-add-job-podname`
`k delete job math-add-job`

to see how many times it took for a success
`k describe job throw-dice-job`
look for succeded and failed

```
Duration: 5s
Pods Statuses:    0 Active (0 Ready) / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  batch.kubernetes.io/controller-uid=888fd28e-a22b-4a
```

### Multiple pods

- sequential

```yaml
apiVersion: batch/v1
kind: Job
metadata:
	name: math-add-job
spec:
	completions: 3
	template:
		spec:
			containers:
			- name: math-add
				image: ubuntu
				command: ['expr', '3', '+', '2']
			restartPolicy: Never
```

On error k will create a job until the completions are fulfilled

- parralel

```yaml
apiVersion: batch/v1
kind: Job
metadata:
	name: math-add-job
spec:
	completions: 3
	parellelism: 3
	template:
		spec:
			containers:
			- name: math-add
				image: ubuntu
				command: ['expr', '3', '+', '2']
			restartPolicy: Never
```

### CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
	name: reporting-cron-job
spec:
	schedule: "*/1 * * * *"
	jobTemplate:
		spec:
			completions: 3
			parellelism: 3
			template:
				spec:
					containers:
					- name: math-add
						image: ubuntu
						command: ['expr', '3', '+', '2']
					restartPolicy: Never
```

`k create -f cron-job-definition.yaml`

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  completions: 3
  backoffLimit: 25
  template:
    spec:
      containers:
        - image: kodekloud/throw-dice
          name: throw-dice
      restartPolicy: Never
```

in parallel

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  completions: 3
  parallelism: 3
  backoffLimite: 25
  template:
    spec:
      containers:
        - image: kodekloud/throw-dice
          name: throw-dice
      restartPolicy: Never
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: throw-dice-cron-job
spec:
  schedule: '30 21 * * *'
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      backoffLimit: 25
      template:
        spec:
          containers:
            - name: throw-dice
              image: kodekloud/throw-dice
          restartPolicy: Never
```
