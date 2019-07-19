# kubernetes
Let's play with kubernetes cluster. k8s is used for kubernetes.

## kubectl

### Configure kubectl to ship command to specofic k8s cluster

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

### kubectl play book

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








