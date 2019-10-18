is what i followed
https://kubernetes.io/docs/setup/learning-environment/minikube/#quickstart

could
"Hello World Node.js app on Kubernetes using Minikube and Katacoda"
https://kubernetes.io/docs/tutorials/hello-minikube/#create-a-deployment

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


### SENTRY-KUBERNETES POD START
kubectl run sentry-kubernetes \
  --image getsentry/sentry-kubernetes \
  --env="DSN=https://cc7b02dae7444f0fb19bd5170c11996b@sentry.io/1783432"
