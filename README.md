# kubernetes
Let's play with kubernetes cluster. k8s is used for kubernetes.

## Configure kubectl to ship command to specific k8s cluster

If you are comming here from [my minikube page](https://github.com/nansari/minikube/blob/master/README.md), you have 2 cluster information in minikube, though ```second-cluster``` has alredy been deleted.

Below command take sometimme and exit with error.
```
$ kubectl cluster-info
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
Unable to connect to the server: dial tcp 192.168.99.101:8443: connect: no route to host
```

Why, kubectl was last configured to interact with second-cluster, that we have deleted but kubectl is still configured to talk to that.
```
$ kubectl config  current-context
second-cluster
```

Let's get all cluster contexts and configure kubectl to talk to default cluster minikube.

```
$ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          minikube         minikube         minikube         
          na-mini-kube     minikube         minikube         
*         second-cluster   second-cluster   second-cluster   
$ kubectl config view
  ...
  ... truncated - check cluster name, IP, cert info etc

$ kubectl config set-context minikube
Context "minikube" modified.

$ kubectl config delete-context second-cluster
warning: this removed your active context, use "kubectl config use-context" to select a different one
deleted context second-cluster from /home/nansari/.kube/config

$ kubectl config delete-context na-mini-kube 
deleted context na-mini-kube from /home/nansari/.kube/config

$ kubectl config use-context minikube
Switched to context "minikube".
```

Let's see if we are able to get cluster info now.

```
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl cluster-info dump |less
 ...
 ...truncated - try it

```

## kubectl primer

So, now we have one cluster. How many nodes we have?

```
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   83d   v1.14.0
 
$ kubectl get nodes -o wide
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    master   83d   v1.14.0   10.0.2.15     <none>        Buildroot 2018.05   4.15.0           docker://18.6.2
$ 
```

minikube is single node k8s cluster. cluster name is minikube, node name is also minikube. ```-o wide``` option can be used to almost all k8s resources to add additional information from one level up construct. It means, cluster info added with nodes, nodes info added with pods and so on.

A cluster can be many namespace. If no namespace is specified with ```-n <namespace>``` kubectl take it ```default`` namespace. How many namespaces we have now?

```
$ kubectl get ns
NAME              STATUS   AGE
default           Active   83d
kube-node-lease   Active   83d
kube-public       Active   83d
kube-system       Active   83d
```
Namespaces are a way to divide cluster resources between multiple users.
Most of k8s resources are in some namespace.

How many resources are there in kubernetes? .
What are resources are namespace sensitive? Total 35
Waht are resources are namespace independent?

```
$ kubectl api-resources
$ kubectl api-resources --namespaced=true
$ kubectl api-resources --namespaced=false
```
Out of total 58 api-resources in version 1.14, 35 are namespaced and 24 are not.
Also in above output, pay attention to ```SHORTNAME```. Most of resources have a short name too.
For example, namespace is ns and pods is po.

What is pod in k8s?
- a group of one or more containers
- a basic unit of deployment in k8s
- can run docker and many other type of container runtime
- each pod will get a dynamic IP, each container in pod will get a port. So container internally is accessible at <pod_ip>:<container_port>
- all containers in pod share network, namespace, resources and storage

Let's get some explaination on pod. ```explain``` work on all other k8s resources.
```
kubectl explain pods
kubectl explain svc
```


## First deployment

Let's deploy a web nginx servers container from focker.io. Our deployment name is ngixweb, generator optiona is optional ( to avoid a warning message about a method being deprecated) and image used is docker.io/nginx:latest
```
$ kubectl create deploy nginx-web  --image=nginx:latest
deployment.apps/nginx-web created

$ kubectl get deploy -o wide
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS       IMAGES                       SELECTOR
hello-minikube   1/1     1            1           83d   hello-minikube   k8s.gcr.io/echoserver:1.10   run=hello-minikube
nginx-web        1/1     1            1           40s   nginx            nginx:latest                 app=nginx-web
```
Our first deployment nginx-web is created. Since we did not mention, how many pods/replica set need to be created in this deployment, only one pod/replica set is created. It is created in default namespace.
It's time to know more details about pods environment.

```
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-56cdb79778-sdbqg   1/1     Running   3          83d
nginx-web-66964bcbcd-vrvnx        1/1     Running   0          5m3s

$ kubectl get rs
NAME                        DESIRED   CURRENT   READY   AGE
hello-minikube-56cdb79778   1         1         1       83d
nginx-web-66964bcbcd        1         1         1       7m25s 

# Notice: ^  An string appended to deployment name to make replica set name
#         ^^ One more string appended to rs name to make pod

$ kubectl get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
hello-minikube-56cdb79778-sdbqg   1/1     Running   3          83d     172.17.0.4   minikube   <none>           <none>
nginx-web-66964bcbcd-vrvnx        1/1     Running   0          5m13s   172.17.0.5   minikube   <none>           <none>
$ 
$ kubectl get pods -n default
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-56cdb79778-sdbqg   1/1     Running   3          83d
nginx-web-66964bcbcd-vrvnx        1/1     Running   0          5m28s
$ 
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
default       hello-minikube-56cdb79778-sdbqg         1/1     Running   3          83d
default       nginx-web-66964bcbcd-vrvnx              1/1     Running   0          6m3s
kube-system   coredns-fb8b8dccf-68z2d                 1/1     Running   5          83d
kube-system   coredns-fb8b8dccf-pddhp                 1/1     Running   5          83d
kube-system   etcd-minikube                           1/1     Running   3          83d
kube-system   kube-addon-manager-minikube             1/1     Running   3          83d
kube-system   kube-apiserver-minikube                 1/1     Running   3          83d
kube-system   kube-controller-manager-minikube        1/1     Running   3          83d
kube-system   kube-proxy-wsls5                        1/1     Running   2          39h
kube-system   kube-scheduler-minikube                 1/1     Running   3          83d
kube-system   kubernetes-dashboard-79dd6bfc48-6nqxm   1/1     Running   4          83d
kube-system   storage-provisioner                     1/1     Running   5          83d
$ 
```
A lot of information above, what if need more details!
```
$ kubectl get pods -o json
$ kubectl get pods -o yaml
```

There is still more to come. For now, we have created nginx-web deployment to start a single container inside a sinle pod.
Can we test it now as we see above ```kubectl get pods -o wide``` shows pod IP 172.17.0.5?
No, this is internal pods IP not accessible by outside world.
We need to expose or deployment as a k8s service. Let's do it.
Beloiw command created service with an IP (each service will have it's own IP) and port 80 that will be accessible via node port IP ( Minikube Linux VM IP, not your PC IP)

```
$ kubectl expose deployment nginx-web --port 80 --type=NodePort
service/nginx-web exposed

$ kubectl get service  nginx-web
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-web   NodePort   10.110.25.135   <none>        80:30723/TCP   26m
$ 

$ minikube ip
192.168.99.100
```
Nginx web server of our nginx deployment should be accessible on http://192.168.99.100:30723 !





$ kubectl describe deployment nginx-web
$ kubectl describe pod nginx-web-66964bcbcd-vrvnx





## Creating a docker container











