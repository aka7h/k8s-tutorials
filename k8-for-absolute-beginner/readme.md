# Kubernetes for Absolute Beginner


# run Kubernetes - minikube

====== Installation =====

1. install kubectl - https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/
2. install minikube - https://minikube.sigs.k8s.io/docs/start/


===== Running a simple application in k8 ======


* To get the version
>kubectl version --client

*Start minikube server
>minikube start

*Get list of available nodes
>kubectl get nodes

*Get list of available pods
>kubectl get pods

*Create a deployment of the docker image ==> CMD POD-NAME IMAGE
>kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10

*Get the list of deployment
>kubectl get deployment

*Exposing the application url and port
>kubectl expose deployment hello-minikube --type=NodePort --port=8080


*Getting the url for application its exposed
>minikube service hello-minikube --url

*Delete the service
>kubectl delete service hello-minikube

*Delete the deployment
>kubectl delete deployment hello-minikube


===== PODS =====

1. Kubernetes Components - https://kubernetes.io/docs/concepts/
2. PODS overview - https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/

image name should be same as the one available in the docker hub

>kubectl run nginx --image=nginx
pod/nginx created

>kubectl get pods

* Gives complete information about the nginx pods
>kubectl describe pod nginx


* Get additional information about the status of pods
>kubectl get pods -o wide


===== YAML Introduction =====

Dict vs List vs List of Dict


* Dict
Name: Jacob
Sex: Male
Age: 30
Title: Systems Engineer


* List
Cars:
	- Corvette
	- BMW
	- Mercedes
	- Audi
	- Fiat
	- Ferrari

* Combination of all

Employee:
	Name: Jacob
	Sex: Male
	Age: 30
	Title: Systems Engineer
	Projects:
		- Automation
		- Support
	Payslips:
		- Month: June
		  Wage: 4000
		- Month: July
		  Wage: 4500



===== PODS - YAML Intro =====


* create as basic nginx pod - ./pods.yml file is available for reference

apiversion: v1
kind: Pod
metadata:
  name: nginx
  label:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx


* run the pods.yml file

> kubectl create -f pods.yml
or
>kubectl apply -f pods.yml



** create a pod with postgres - here add new property "env" which is a sibling to name and image under containers

apiVersion: v1
kind: Pod
metadata:
    name: postgres
    labels:
        tier: db-tier
spec:
    containers:
    -   name: postgres
        image: postgres
		env:
		-	name: POSTGRES_PASSWORD
			value: mysecretpassword



===== Replication Controller and ReplicaSet =====

* Brains behind the kubernetes, that monitors kubernetes objects and take necessry actions 
* It automatically restart the pods if incase they are crashed
* they help with loadbalancing and scaling the application pods

* Replication Controller -> rc_definition.yaml
apiVersion: v1
kind: ReplicationController
metadata:
	name: myapp-replication
	labels:
		app: myapp-replication
		type: front-end
spec:
	template: 		# this is the wrapper around the pods
		metadata:
			name: myapp-pod
			labels:
				app: myapp-pod
				type: front-end
		spec:
			containers:
			-	name: nginx
				image: nginx
	replicas: 3

> kubectl create -f rc_definition.yaml
> kubectl get replicationcontroller
> kubectl get pods




* ReplicSet -> replicaset_definition.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: myapp-replicas
	labels:
		app: myapp-replicas
		type: front-end
spec:
	template: 		# this is the wrapper around the pods
		metadata:
			name: myapp-pod
			labels:
				app: myapp-pod
				type: front-end
		spec:
			containers:
			-	name: nginx
				image: nginx
	replicas: 3
	selector: 		# this will also consider all hte pods that are not created/managed by the replicaset
		matchLabels:
			type: front-end

> kubectl create -f replicaset_definition.yaml
> kubectl get replicaset
> kubectl get pods

*** How to scale the pods accordingly

1. you can update the *.yaml file for the replicas section and run
> kubectl replace -f replicaset_definition.yaml

2. we can update the replicas through command line, though they are not permanent
> kubectl scale --replicas=6 -f replicaset_definition.yaml
> kubectl scale --replicas=6 <type> <metadata: name>

*** End - How to scale the pods accordingly

* get the list of replicaset
> kubectl get replicaset

* delete all the replicaset based on the metadata name mentioned
> kubectl delete replicaset <metadata: name> 

> kubectl delete pod <metadata: name>


******** DEMO REPLICASET *********
(base) Akkashs-MacBook-Pro:replicaset aka7h$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-2skf7   1/1     Running   0          42s
myapp-replicaset-btmb8   1/1     Running   0          42s
myapp-replicaset-jb46t   1/1     Running   0          42s
(base) Akkashs-MacBook-Pro:replicaset aka7h$ kubectl delete pods myapp-replicaset-btmb8
pod "myapp-replicaset-btmb8" deleted
(base) Akkashs-MacBook-Pro:replicaset aka7h$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   3         3         2       70s
(base) Akkashs-MacBook-Pro:replicaset aka7h$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-2skf7   1/1     Running   0          76s
myapp-replicaset-5r485   1/1     Running   0          11s
myapp-replicaset-jb46t   1/1     Running   0          76s
(base) Akkashs-MacBook-Pro:replicaset aka7h$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   3         3         3       85s
(base) Akkashs-MacBook-Pro:replicaset aka7h$ 


* open the file in vim and change the codes accordingly. Note that this is temporary
> kubectl edit replicaset myapp-replicaset

* this will scale down the replicas temporarily. Once we rerun the code it will be back to normal
> kubectl scle replicaset myapp-replicaset --replicas=2


===== Deployments =====

1. Download 
2. Upgrade
3. Rolling update
4. Rollback
5. Delayed update


* its similar to the replicaset structure 
* only difference is the kind is replaced with "Deployment"

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: mywebsite
    tier: frontend
spec:
  replicas: 4
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx
          image: nginx
  selector:
    matchLabels:
        app: myapp



*** Update and Rollback
1. Destory the existing instance of pods and deploy the new instaces - this can lead to application downtime (Recreate stratergy)
2. We take down the older version and bring up one by one (Rolling Update) 
> kubectl apply is used to apply the changes

> kubectl set image deployment/myapp-deployment nginx=nginx.1.9.1


* this is to check the current status nd the history of rollouts
> kubectl rollout status deployment/myapp-deployment
> kubectl rollout history deployment/myapp-deployment


** Upgrades and Rollback

> kubectl rollout undo deployment/myapp-deployment

Update is done based on rolling update logic, hence the service will not be distrupted. 


===== Networking in Kubernetes =====

* Each pods in the cluster gets its own IP
* a private IP is set inside a cluster and each pods are asigned with IP from the local IP
* all containers/pods can communicate to one another without NAT
* all node can communicate with all containers and vice verse without NAT

* there are chances of getting a conflict with the IP as pods under 2 different nodes can have same IP

* We need to use external oper source tools like cillium which can maintain the routing
https://cilium.io/


===== Services =====

Services gets the request through the port and communicate with the PODS. It acts as a middleman between the POD and the client.

It is required only when its need to be access by another

NodePort: The service maps the nodeport and port of the pods(TargetPort) It ranges from 30000 - 32767

service-definition.yaml -->

apiversion: apps/v1
kind: Service
metadata:
	name: myapp-service
spec:
	type: NodePort
	ports:
	-	targetPort: 80
		port: 80
		nodePort: 30008
	selector:
		app: myapp
		type: front-end


> kubectl create -f service-definition.yaml

> kubectl get services

when the service is created, it checks for the selector labels in it and the pods are assigned with its specific nodeport mentioned.

for pods with metadata app: myapp and type: front-end, the nodeport is 30008

* if multiple pods are availble in the node, based on the random selection, the service will choose one of the pods that is available

* for single pod in multiple nodes, the service is span across all the nodes and the node port are similr to the all nodes. 

* if single pod on single node, multiple pods on single node or multiple pods on multiple node, 
the service will work the same and no additional configuration is requires

ClusterIP: A cluster can have multiple pods like frontend, backend, database and redis. All pods need to communicate with one or more pods

The pods IP are dynamic and hence hardcoding the pod ips will be difficult. 

Create serive called BACKEND to communicate frontend to backend, create service called REDIS to communicate backend with redis

This creates a microservice architecture. 

apiVersion: v1
kind: Service
metadata:
	name: back-end
spec:
	type: clusterIP
	ports:
	-	targetPort: 80
		port: 80
	selector:
		app: myapp
		type: backend


LoadBalancer: 

this works only the cloud support like GKS, AKS etc

apiVersion: v1
kind: Service
metadata:
	name: back-end
spec:
	type: LoadBalancer
	ports:
	-	targetPort: 80
		port: 80



===== Microservice Architecture using Kubernetes =====


Goal:
1. Deploy containers
2. Enable Connectivity between containers
3. Enable external access to the containers


Steps:
1. Deploy all the container as PODS or Replicasets
2. Create Service (ClusterIP)
	a. redis
	b. postgres
3. Create Service (NodePort)
	a. voting-app
	b. result-app

* to delete all the pods service and deployment at Once
> kubectl delete deployment,pod,svc --all
