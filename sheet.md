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

`kubectl run nginx --image nginx `
creates a new pod named nginx based on the nginx image from dockerhub

`kubectl get pods `
outputs a list of available pods

`kubectl describe pod mypod`
show pod details

`kubectl create -f pod-definition.yaml `
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
