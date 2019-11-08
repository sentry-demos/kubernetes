# Overview  
This is not a Sentry Kubernetes SDK / designed for K8. THink of it as a demo, a use case, an application/impelmentaiton of Sentry (method).
This repo is for capturing Kubernetes Stream [Events](https://www.bluematador.com/blog/kubernetes-events-explained) using Sentry. [Log](https://kubernetes.io/docs/concepts/cluster-administration/logging/). There is nothing inherently different about how Sentry does this.  

...in a docker container that does the work for you, here's the code [getsentry/sentry-kubernetes](https://github.com/getsentry/sentry-kubernetes) for it if you're curious. also [hub.docker getsentry/sentry-kubernetes](https://hub.docker.com/r/getsentry/sentry-kubernetes)

This is not a new Sentry SDk designed specifically for Kubernetes.
It's a method that comes with every SEntry SDK for 'Capruring a Message' which takes any arbitrary piece of data you pass it. So this repo is in Python but you could do something else.
Attention give to `serviceaccounts` clsuterroles, as this would prevent communication from sentry kubernetes pod from accessing kuberentes stream.
**"Exemplifies"**

#### How this demo works
Kubernetes Pod 1 - emits errors (by trying to use too much CPU)
^ these errors make there way into the "kubernetes" Stream/Log
Kubernetes Pod 2 - sentry-kubernetes - is basically a while loop listening to the Kubernetes EVent stream.
decides what is an EVent or not. **you should amplify your own logic here**
captures it and sends to SEntry as an Event.


#### Versions
kbctl 1.16  
kbctl Client Version v1.16.2  
kbctl Server Version v1.16.0  
minikube version: v1.4.0  


## Setup / RUN
#### Prerequisites
`kubectl` and `minikube`  

#### Steps
1. `git clone git@github.com:<your_fork>/sentry-native.git`
2. [start minikube](#start-minikube)
3. [start pod(s)](#start-pod)
4. see event on Sentry.io

### Start Minikube
https://kubernetes.io/docs/setup/learning-environment/minikube/#quickstart  
```
minikube start
```

### Start Pod

#### SENTRY-KUBERNETES POD START
```
kubectl run sentry-kubernetes \
  --image getsentry/sentry-kubernetes \
  --env="DSN=https://cc7b02dae7444f0fb19bd5170c11996b@sentry.io/1783432"
kb get pod sentry-kubernetes-5dbfb4597f-xr7kj
```
#### CREATE A POD THAT REQUESTS TOO MANY RESOURCES FROM A NODE
```
# Specifies a Memory/CPU resource request that is too big for your nodes"  
kubectl apply -f ./cpu-request-limit-2.yaml
kubectl apply -f ./memory-request-limit-3.yaml
```

### Stop / Delete Pod
```
kubectl delete deployment hello-minikube
kubectl delete deployment sentry-kubernetes

kubectl delete services hello-minikub
```
or
```
# kubectl delete deployment memory-demo-3 doesn't work so...
kubectl delete -n default pod memory-demo-3

# WORKS
kb delete -n default pod cpu-demo-2
```

### Stop / Delete Minikube
```
minikube stop
minikube delete
```

## USEFUL COMMANDS
`minikube -help`

```
# launches http://127.0.0.1:50159/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/overview?namespace=default
minikube dashboard

minikube addons list
```
`kubectl get -h`
```
kubectl get deployments
kubectl get pods
kubectl describe pod # lists all, with statuses
kubectl describe pod <name>
kubectl get events <-- see events that sentry-kubernetes isn't capturing
kubectl get services
kubectl config view
kubectl get namespaces
kubectl logs <name> <--- to see logs of the pod, not to be confused with Events of the pod.
kubectl get sa <---get service accounts
```
and
```
kubectl logs <sentry-kubernetes>
```
and
```
# show info, look for sa/security/roles/clusterRolebinding
kubectl get deployment sentry-kubernetes -o yaml  
```
and
YES TO-ADD:  
```
kubectl scale deployment sentry-kubernetes --replicas=1
kubectl scale deployment sentry-kubernetes --replicas=0
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --serviceaccount=sentry-kubernetes
kubectl delete clusterrolebinding sentry-kubernetes
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --serviceaccount=default:sentry-kubernetes
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --serviceaccount=sentry-kubernetes
```

DID THIS...
```
kubectl create sa sentry-kubernetes
kubectl create clusterrole sentry-kubernetes --verb=get,list,watch --resource=events
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --user=sentry-kubernetes NOOOO

kubectl run sentry-kubernetes \
  --image getsentry/sentry-kubernetes \
  --serviceaccount=sentry-kubernetes \
  --env="DSN=https://cc7b02dae7444f0fb19bd5170c11996b@sentry.io/1783432"
```

"or else you get Forbidden403 error in logs/stream"  
and
```
kb get pod sentry-kubernetes-69f9bbdfc7-mfn52 -o yaml
kb get clusterrole sentry-kubernetes -o yaml
kb get clusterrolebinding sentry-kubernetes -o yaml

and
kb get serviceaccounts/sentry-kubernetes -o yaml
```


# Documentation 
https://kubernetes.io/docs/tutorials/hello-minikube/

cheatsheet  
https://kubernetes.io/docs/reference/kubectl/cheatsheet/  

"Create a Pod that Requests too many resources from a Node"  
CPU or Memory  
https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/#specify-a-cpu-request-that-is-too-big-for-your-nodes  
https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/  

other users who made their own sentry-kubernetes  
https://hub.docker.com/search?q=sentry-kubernetes&type=image