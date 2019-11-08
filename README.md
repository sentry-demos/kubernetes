# Overview
## HOW THIS DEMO WORKS
**TL;DR**  
Kubernetes Pod 1 - emits errors (by trying to use too much CPU). These errors make there way into the "kubernetes" Stream/Log  
Kubernetes Pod 2 - sentry-kubernetes - watches/consumes the Kubernetes EVent stream.

1. This demo involves running 2 kubernetes pods.  

2. **Pod 1** requests too many resources (CPU/RAM) from the K8S cluster, and thus causes an error that gets sent into the Kubernetes Steam/log of Events

3. **Pod 2** uses the Sentry Python SDK to capture this error from the Kubernetes Stream and send it as a Event to Sentry.io

4. Docker Images and yaml's are provided for both of these pods. If you want to see or update the python source code for the second pod and DIY, see https://github.com/getsentry/sentry-kubernetes/blob/master/sentry-kubernetes.py

#### NOTES ON WHAT POD 2 DOES
The docker image used for pod2 is here [hub.docker getsentry/sentry-kubernetes](https://hub.docker.com/r/getsentry/sentry-kubernetes). The container runs a file called `sentry-kuernetes.py` watches/consumes the Kubernetes Stream and has logic for evaluating if it's something benign/ignorable or errorneous/problematic, in which case it will pass to Sentry's `capture_message` method which will send it to Sentry.io.

It decides for you what is an EVent or not but you could amplify the logic with your own by updating sentry-python.py and re-making your own image/k8pod.

#### NOTES ON K8 PODS TALKING TO EACH OTHER
Attention give to `serviceaccounts` clsuterroles, as this would prevent communication from sentry kubernetes pod from accessing kuberentes stream.

#### THE POWER OF SENTRY'S CAPTURE MESSAGE

- Don't let the power of Capture Message stop here. You can use it in any major language or framework as it's available in all of our [SDK's](http://sentry.io/platforms). 

- You can define any arbitrary piece of data to sentry's `capture_message` function for sending to Sentry.io as an Event. This helps add extra information context to your overall tech stack, in addition to the usual places where you use Sentry (REST API's, front-end Javascript). This demo **exemplifies** this power/visibility by using it with Kubernetes.

#### Versions
```
kbctl 1.16  
kbctl Client Version v1.16.2  
kbctl Server Version v1.16.0  
minikube version: v1.4.0  
```

## Setup
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

cluster administration logging
https://kubernetes.io/docs/concepts/cluster-administration/logging/

**Bonus** Sentry Helm Charts  
https://github.com/helm/charts/tree/master/stable/sentry