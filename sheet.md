# Kubernetes Cheat Sheet

## General

`kubectl cluster-info`
`kubectl get nodes`

### general structure

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
