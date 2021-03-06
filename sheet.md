# Kubernetes Cheat Sheet

## General

`kubectl cluster-info`
`kubectl get nodes`

### General structure

Every YAML file shares this structure

```
apiVersion: 
kind:
metadata:


spec:
```

## Pods

```
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

`kubectl run nginx --image nginx`
creates a new pod named nginx based on the nginx image from dockerhub

`kubectl get pods`
outputs a list of available pods

`kubectl describe pod mypod`
show pod details

`kubectl create -f pod-definition.yaml`
Create Pod from yaml file

`kubectl get pod <pod-name> -o yaml > pod-definition.yaml`
Extract pod definition to a file


`kubectl edit pod <pod-name>`
Edit pod properties

## Replica Set
```
apiVersion: v2
```
* More recent version of Replication Controller
* Is in charge of making as many replicas as defined avilable
* Monitors Pods
* Matches via labels

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 6
  selector:
    matchLabels:
     type: front-end
```

`kubectl create -f replicaset-definition.yml`
`kubectl get replicaset`
`kubectl delete replicaset myapp-replicaset`
This also deletes underlying pods
`kubectl replace -f replicaset-definition.yml`
update
`kubectl scale -replicas=6 -f replicaset-definition.yml`

`kubectl scale --replicas=6 -f replicaset-definition.yml`

## Namcespaces

```
apiVersion: v1
kind: Namcespace
metadata:
name: dev
```

`kubectl create namespace dev`

`kubectl config set-context $(kubectl config current-context) --namespace dev`

Globally set namespace

## Resource Quota

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

## Commands and Arguments

*  Entypoint in docker corresponds to command inside containers definition
*  CMD in dockerfile corresponds to args in containers definition

## Config Maps

* Detatch configuration from  definition file

create with:

imperative: `kubectl create configmap <config-name> --from-literal=<key>=<value>`
* also possible to load configuration from a file instead of defining multiple from-literal params

declarative: `kubectl create -f file.yaml`

### File Structure

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

### Reference config map in pod

Whole Env

```
spec: 
containers:
    - ...
    envFrom:
        - configMapRef:
            name: app-config
```

Specific Key 

```
spec: 
containers:
    - ...
    env:
      - name: APP_COLORR
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: APP_COLOR
```

Volume

```
spec: 
containers:
    - ...
    volumes:
      - name: app-config-volume
        configMap:
          name: app-config
```

## Secrets

* Similar to configmaps, but stored encoded


### Reference secrets in pod

Whole Env

```
spec: 
containers:
    - ...
    envFrom:
        - secretRef:
            name: app-config
```

Specific Key 

```
spec: 
containers:
    - ...
    env:
      - name: DB_Password
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: DB_Password
```

Volume

```
spec: 
containers:
    - ...
    volumes:
      - name: app-secret-volume
        secret:
          name: app-config
```

## Security Context

* Containers per default run as root
* Security context can be defined on pod or on container level
* Even the root user has limited capabilites on the docker machine
* Keeps the user from manipulating the shared kernel on the host machine
* Capabilities can be granted from within the securityContext

```
    ...
    securityContext:
      runAsUser: 1000
      capabilites:
        add: ["SYS_TIME"]
```

## Resources

* Resources node can be defined per container in the pod

```
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limit:
        memory: "2Gi"
        cpu: 2
```
## Taints and Tolerance

* Mechanism to make a node only accept certain pods
* A node can be tainted 
* Only Pods which are tolerant to that taint can be scheduled on this node
* A not tainted node will also accept pods with certain tolerances
* Master node is tainted per default

    `kubectl taint nodes node-name key=value:taint-effect`

taintEffect: what happens to PODs that do not tolearte this taint? 

possible values: NoSchedule | PreferNoSchedule | NoExecute

## Node Selectors

* Assign Pods to certain nodes with a selector

```
    kind: Pod
    ...
    spec:
      containsers:
        ...
      nodeSelector:
        size: Large
```

* The node selector refers to labels assigned on a node

`kubectl label nodes <node-name> <label-key>=<label-value>`

* Has its limits: you cant say on label Large or Medium
* Can't say something like go to any node which is not small

## Node Affinity

* Node affinity can be used for more complex requirements

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

* requiredDuringSchedulingIgnoredDuringExecution wont run a pod if the requirement has not
been met
* preferredDuringSchedulingIgnoredDuringExecution is a sofer way and run the pod on any node if
the requirement cant be met

Example operator:

* In 
* NotIn
* Exists
* DoesNotExist
* Gt
* Lt

## Multi Container Pods

Three common patterns:

1. Sidecar
2. Adapter
3. Ambassador

Example:
1. Sidecar: Log-Agent to collectl logs and forward them to a central Log Server
2. Convert log files to common format for the Log Server
3. When dealing with multiple environments and therefore different databases an Ambassador could
   deal with the actual choosing of the concrete database while the application only
   connects to localhost


## Readiness Probe

Pods conditions

* PodScheduled
* Initialized
* ContainersReady
* Ready

* Ready means that the application is running and ready to accept user traffic
* Used to avoid routing traffic to a Pod which can not reply
* Traffic is only routed when Pods is marked as running

Different probes exist for different types of applications

```
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10G
  periodSeconds: 5
  failureThreshold: 8
```

```
readinessProbe:
  tcpSocket:
    port: 3306
```

```
readinessProbe:
  exec:
    command:
    - cat
    - /app/is_ready
```

## Liveness Probe

* Application might crashed during runtime?
* Container still running 
* Liveness probes look inside the container to identify such cases and restart the pod if
    required
* Same configurations possible as with the readinessProbe

## Logging

`kubectl logs -f event-simulator-pod event-simulator`

* -f means follow
* second parameter is the name of the container inside a multi container pod
* Can be left if pod only has one container

## Labels, Selectors & Annotations

* Labels and selectors are a method to group things together
* or filter them by one or multiple criteria
* Labels are properties attached to each item
* Selectors help you filter these items


labels in pod def:

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-End
spec:
  ...
```
select pod by labels 

`kubectl get pods --selector app=App1`

* Annotations are used to record details for informative purpose
* Build number, contact details, ...

## Deployment

* Creating a deployment triggers a rollout
* With deployment revision


`kubectl rollout history deployment/myapp-deployment`

There are two types of deployment strategy

1. Recreate (take all old pods down, spawn new pods after)
2. Rolling update (take one of the old pods down and spwan a pod with the new version after)

* First type creates downtime
* Rolling update is the default strategy

* Use `kubectl apply` to update deployment and create a new revision
* Another way is to use `kubectl set image ...`
* Update creates a new replicaset under the hood
* Rollback reactivates old replicase

    Commands

* `kubectl create -f deployment-definition.yaml`
* `kubectl get deployments`
* `kubectl apply -f deployment-definitionyml`
* `kubectl set image deplyoment/myapp-deployment nginx=nginx:latest`
* `kubectl rollout status deployment/myapp-deployment`
* `kuectl rollout history deployment/myapp-deployment`
* `kubectl rollout undo deployment/myapp-deployment`


## Jobs

* Jobs goal is it to successfuly execute a container a specified amount of times
* A job is only completed when the container exits without failure

```
apiVersion: batch/v1
kind: Job
metadata:
  name: random-error-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: random-error
          image: some-image
      restartPolicy: Never
```

### Cronjob

```
apiVersion: batch/v1beta
kind: CronJob
metadata:
  name: random-error-cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
  spec:
    completions: 3
    parallelism: 3
    template:
      spec:
        containers:
          - name: random-error
            image: some-image
        restartPolicy: Never
```
## Services

* Enable communication between various components within and outside the application
* Connect applications together with other applications or users
* NodePort Service: Forwards requests coming to a certain port of the node to a certain port
    of the pod within the node
* ClusterIP: Service creates a virtual IP inside the cluster to enable communication between
    different services (for example frontend to backend communication)
* LoadBalancer: Creates loadbalancer for our application in supported cloud providers

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 8080
      port: 8080
      nodePort: 30008
  selector:
    - app: myapp
      type: front-end
```


### NodePort

Terms defined from the viewpoint of the service 

* TargetPort: Port on the Pod
* Port: The port on the service itself
* NodePort: Port on the node itself to access the application externally (range 30000 - 32767)

* The service is kind of a virtual server inside the node
* Within the cluster it has its own ip adress
* This IP is called the ClusterIp address
* Functions as an internal loadbalancer between pods
* Uses a random algorithm for that
* Service can provide an interface to different pods on multiple nodes
* NodePort can be called on any of the available nodes


### ClusterIP

* Pod ips cant used for stable communication within the application as they are spawned and
    removed on demand
* With multiple replicas which pod should we talk to?
* A Service provides a common interface to access the pods in a group

```
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  ports:
    - targetPort: 8080
      port: 8080
  selector:
    - app: myapp
      type: back-end
```

## Network Policy

* Kubernetes comes with an "Allow all" rule
* When traffic is not limited one pod can talk to every other
* NetworkPolicies are Kubernetes objects
* With Network policies rules for incoming and outgoing traffic can be configured

## Volumes

Docker:

Bind mount mounts a path of the host system into the container system
Volume mount /var/lib/docker/volumes/volume _name is created and can be reused

## Persistent Volumes

* Manage storage more centrally
* Cluster wide pool of storage
* Users can use storage from that pool with persistent volume claims

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-voll
spec:
  accessModes:
    - ReadWriteOnce
```
access modes:

ReadOnlyMany
ReadWriteOnce
ReadWriteMany

once and many relate to single or multi node access


## Persistent Volume Claims

* Separate object 
* Admin creates Persistent Volumes
* User creates persistent volume claims to use the storage
* Once the claims are created Kubernetes binds the claims to the Persistent Volumes
* Binding is based on the request and the properties of the PersistedVolume
* Every persistent Volume Claim is bound to a single Persistent Volume
* Binding is done by criteria like
    * Sufficient Capacity
    * Access Modes
    * Volume Modes
    * Storage Class
* Apart from these criteria also selector based matching is possible
* Relation is 1:1 no second claim on a Persistent Volume

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

* Per default a persisted volume is not deleted if the claim is deleted
* persistentVolumeReclaumPolicy_: Retain | Delete | Recycle

Use claim inside a POT

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim

```
Same for ReplicaSets or Deployments


## Tips for certification

### Setup Environment:

alias k=kubectl

#### vim: 

set tabstop=2
set expandtab
set shiftwidth=2

#### variables:

export do="--dry-run=client -o yaml"

#### optional to switch namespaces:

alias kn='kubectl config set-context --current --namespace '

#### autocompletion

source <(kubectl completion bash)

complete -F __start_kubectl k # to make it work with the alias k

avoid long filenames for the yamls, use exercise number

### Kubectl explain

kubectl explain --recursive pod
