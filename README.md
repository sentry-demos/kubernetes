docs   
https://kubernetes.io/docs/setup/learning-environment/minikube/#quickstart  

could  
"Hello World Node.js app on Kubernetes using Minikube and Katacoda"  
https://kubernetes.io/docs/tutorials/hello-minikube/#create-a-deployment

### Pre-Req
download `kubectl` and `minikube`

### MINIKUBE START
```
minikube start
```

### POD START
```
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10

kubectl expose deployment hello-minikube --type=NodePort --port=8080
minikube service hello-minikube --url
http://192.168.99.101:32237 <--- can now access it on this URL

describe pod hello-minikube-797f975945-b87ns
```

### POD FINISH
```
kubectl delete deployment hello-minikube
kubectl delete deployment sentry-kubernetes

kubectl delete services hello-minikub
```
or
```
# kubectl delete deployment memory-demo-3 doesn't work so...
kubectl delete -n default pod memory-demo-3
```

### MINIKUBE FINISH
```
minikube stop
minikube delete
```

## EXTRA
### MINIKUBE
`minikube -help`

```
# launches http://127.0.0.1:50159/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/overview?namespace=default
minikube dashboard

minikube addons list
```
### KUBECTL
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
### SENTRY-KUBERNETES POD START
```
kubectl run sentry-kubernetes \
  --image getsentry/sentry-kubernetes \
  --env="DSN=https://cc7b02dae7444f0fb19bd5170c11996b@sentry.io/1783432"
kb get pod sentry-kubernetes-5dbfb4597f-xr7kj
```

### CREATE A POD THAT REQUESTS TOO MANY RESOURCES FROM A NODE
CPU or Memory  
https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/#specify-a-cpu-request-that-is-too-big-for-your-nodes  
https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/

"Specify a Memory request that is too big for your nodes"  
```
kubectl apply -f ./cpu-request-limit-2.yaml
kubectl apply -f ./memory-request-limit-3.yaml
```

```
#Don't run this line NO kubectl create namespace cpu-example
#kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit-2.yaml --namespace=cpu-example

kubectl describe pod cpu-demo-2 --namespace=cpu-example
  Events:
    Type     Reason            Age        From               Message
    ----     ------            ----       ----               -------
    Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 Insufficient cpu.
    Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 Insufficient cpu.
```
^^ but does not show in output of:
`kubectl get events`  
nor did it get captured by sentry-kubernetes


# Next
10/18/19
```
I feel like youd' need to create ACLs for this.

Some RBAC stuff.


> Basically, communicating to Kubernetes API from internal, needs permissions assigned to it.

https://github.com/getsentry/sentry-kubernetes/issues/4

kubectl logs sentry-kubernetes-5dbfb4597f-xr7kj
```



note
```
kubectl create sa sentry-kubernetes
kubectl create clusterrole sentry-kubernetes --verb=get,list,watch --resource=events
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --user=sentry-kubernetes

kubectl run sentry-kubernetes \
  --image getsentry/sentry-kubernetes \
  --serviceaccount=sentry-kubernetes \
  --env="DSN=https://cc7b02dae7444f0fb19bd5170c11996b@sentry.io/1783432"

kubectl logs sentry-kubernetes-5554b747fc-zb4hv
```