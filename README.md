# Overview  
This is not a Sentry Kubernetes SDK / designed for K8.

This is a demo implementation of the SEntry Python SDK that passes a Kubernetes Event object to the Sentry Python SDK for sending as an Sentry Event to Sentry.

There's logic built into the python here [getsentry/sentry-kubernetes/sentry-kubernetes.py](https://github.com/getsentry/sentry-kubernetes/blob/master/sentry-kubernetes.py) for evaluating the K8 Event as an exception or not, and you can run this python in a kubernetes pod using [hub.docker getsentry/sentry-kubernetes](https://hub.docker.com/r/getsentry/sentry-kubernetes), which is what this demo will focus on.

There's a python loop watching the Kubernetes Stream/Log [Events](https://www.bluematador.com/blog/kubernetes-events-explained) using Sentry. [Log](https://kubernetes.io/docs/concepts/cluster-administration/logging/).
You can define any arbitrary piece of data to sentry's `capture_message` function for sending to Sentry.io as an Event.

The code is in a docker image which you can run as a container. This is the python code it uses [getsentry/sentry-kubernetes](https://github.com/getsentry/sentry-kubernetes) and this is the docker image that this repo will instruct you to use. [hub.docker getsentry/sentry-kubernetes](https://hub.docker.com/r/getsentry/sentry-kubernetes)

It's a method that comes with every Sentry SDK for 'Capruring a Message' which takes any arbitrary piece of data you pass it. So this repo is in Python but you could do something else.
Attention give to `serviceaccounts` clsuterroles, as this would prevent communication from sentry kubernetes pod from accessing kuberentes stream.

The power of Capture Message lives across all Sentry SDK's and **exemplifies** the power to let you decide what's errorneous, extra information to enhance the context...

#### How this demo works
Kubernetes Pod 1 - emits errors (by trying to use too much CPU)
^ these errors make there way into the "kubernetes" Stream/Log

Kubernetes Pod 2 - sentry-kubernetes - is basically a while loop listening to the Kubernetes EVent stream.

It decides for you what is an EVent or not but you could amplify the logic with your own by updating sentry-python.py and re-making your own image/k8pod.

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

**Bonus** Sentry Helm Charts  
https://github.com/helm/charts/tree/master/stable/sentry