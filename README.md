docs   
https://kubernetes.io/docs/setup/learning-environment/minikube/#quickstart  

could  
"Hello World Node.js app on Kubernetes using Minikube and Katacoda"  
https://kubernetes.io/docs/tutorials/hello-minikube/#create-a-deployment

cheatsheet  
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

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

### POD STOP / DELETE
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

### MINIKUBE STOP / DELETE
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

1.
```
kubectl logs <sentry-kubernetes>
  2019-10-28 18:04:05,554 Exception when calling CoreV1Api->list_event_for_all_namespaces: (403)
  Reason: Forbidden
  HTTP response headers: HTTPHeaderDict({'Cache-Control': 'no-cache, private', 'Content-Type': 'application/json', 'X-Content-Type-Options': 'nosniff', 'Date': 'Mon, 28 Oct 2019 18:04:05 GMT', 'Content-Length': '291'})
  HTTP response body: b'{"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"events is forbidden: User \\"system:serviceaccount:default:sentry-kubernetes\\" cannot watch resource \\"events\\" in API group \\"\\" at the cluster scope","reason":"Forbidden","details":{"kind":"events"},"code":403}\n'
```
^^ needs RBAC (Remote Based Access Controls)
https://github.com/getsentry/sentry-kubernetes/issues/4


2. DID THIS...
```
kubectl create sa sentry-kubernetes
kubectl create clusterrole sentry-kubernetes --verb=get,list,watch --resource=events
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --user=sentry-kubernetes

kubectl run sentry-kubernetes \
  --image getsentry/sentry-kubernetes \
  --serviceaccount=sentry-kubernetes \
  --env="DSN=https://cc7b02dae7444f0fb19bd5170c11996b@sentry.io/1783432"
```
STILL FAILS... w/ same Forbidden403 error


DEBUG...
```
# show info, look for sa/security/roles/clusterRolebinding
kubectl get deployment sentry-kubernetes -o yaml  
```

3. SOLUTIONS  
TO TRY?

Q1.
```
# when you run `create clusterrolebinding` you can specify `--template` to pass this:

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: sentry-kubernetes
  namespace: default
roleRef:
  kind: ClusterRole
  name: sentry-kubernetes
  apiGroup: rbac.authorization.k8s.io  
subjects:
- kind: ServiceAccount
  name: sentry-kubernetes
  namespace: default  

^^ Chnkr suggested using this: https://github.com/getsentry/sentry-kubernetes/issues/4#issuecomment-362572154

so instead of:
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --user=sentry-kubernetes

do this:
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --user=sentry-kubernetes --template custom-clusterrolebinding.yaml

However, 'template' is for some golang type of template :(

Docs:
https://kubernetes.io/docs/reference/access-authn-authz/rbac/#kubectl-create-clusterrole
https://kubernetes.io/docs/refe

rence/access-authn-authz/rbac/#kubectl-create-clusterrolebinding
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-clusterrolebinding-em-
```

Q.2
Need to run this? where from and when? cluster/minikube
```
"To enable RBAC, start the apiserver with --authorization-mode=RBAC."  
https://kubernetes.io/docs/reference/access-authn-authz/rbac/#kubectl-create-clusterrolebinding
```

TO TRY?  
'secret'
```
secrets:
- name: sentry-kubernetes-token-nq2f9
```

TO TRY?  
FWIW instead of using a cluster role you can also use a normal role and pass in the EVENT_NAMESPACES environment variable to limit monitoring to specific namespaces.


```
kb get pod sentry-kubernetes-69f9bbdfc7-mfn52 -o yaml
kb get clusterrole sentry-kubernetes -o yaml
kb get clusterrolebinding sentry-kubernetes -o yaml
```

 kb get serviceaccounts/sentry-kubernetes -o yaml

-------------------------
kubectl scale deployment sentry-kubernetes --replicas=1
kubectl scale deployment sentry-kubernetes --replicas=0

kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --serviceaccount=sentry-kubernetes

kubectl delete clusterrolebinding sentry-kubernetes

kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --serviceaccount=default:sentry-kubernetes
kubectl create clusterrolebinding sentry-kubernetes --clusterrole=sentry-kubernetes --serviceaccount=sentry-kubernetes
