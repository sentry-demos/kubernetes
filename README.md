is what i followed  
https://kubernetes.io/docs/setup/learning-environment/minikube/#quickstart  

could  
"Hello World Node.js app on Kubernetes using Minikube and Katacoda"  
https://kubernetes.io/docs/tutorials/hello-minikube/#create-a-deployment

you need to download kubectl and minikube  

apply vs create vs run?  

### MINIKUBE START
minikube start

### POD START
```
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10

kubectl expose deployment hello-minikube --type=NodePort --port=8080
minikube service hello-minikube --url
http://192.168.99.101:32237 <--- caln now access it on this URL

describe pod hello-minikube-797f975945-b87ns
```

### POD FINISH
kubectl delete deployment hello-minikube
kubectl delete deployment sentry-kubernetes

kubectl delete service hello-minikube?

### MINIKUBE FINISH
minikube stop
minikube delete


## EXTRA
### MINIKUBE
`minikube -help`
minikube dashboard - http://127.0.0.1:50159/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/overview?namespace=default
minikube addons list
### KUBECTL
`kubectl get -h`
kubectl get deployments
kubectl describe pod <name>
kubectl get pods
kubectl get events <-- see events that sentry-kubernetes isn't capturing
kubectl get services
kubectl config view
kubectl get namespaces

### SENTRY-KUBERNETES POD START
kubectl run sentry-kubernetes \
  --image getsentry/sentry-kubernetes \
  --env="DSN=https://cc7b02dae7444f0fb19bd5170c11996b@sentry.io/1783432"
kb get pod sentry-kubernetes-5dbfb4597f-xr7kj


### CREATE A POD THAT REQUESTS TOO MANY RESOURCES FROM A NODE
"Specify a CPU request that is too big for your nodes"  
```
kubectl create namespace cpu-example
kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit-2.yaml --namespace=cpu-example
kubectl get pod cpu-demo-2 --namespace=cpu-example <--- doesn't show yet in kubectl get pods

kubectl describe pod cpu-demo-2 --namespace=cpu-example
...
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 Insufficient cpu.
  Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 Insufficient cpu.

^^ but does not show in output of:
kubectl get events

^^ nor did it get captured by sentry-kubernetes
```