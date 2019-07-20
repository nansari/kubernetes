# kubernetes
Let's play with kubernetes cluster. k8s is used for kubernetes.

## Configure kubectl to ship command to specific k8s cluster

If you are coming here from [my minikube page](https://github.com/nansari/minikube/blob/master/README.md), you have 2 cluster information in minikube, though ```second-cluster``` has already been deleted.

Below command take some time and exit with an error.
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
What are resources are namespace sensitive? Total of 35
What are resources are namespace independent?

```
$ kubectl api-resources
$ kubectl api-resources --namespaced=true
$ kubectl api-resources --namespaced=false
```
Out of total 58 api-resources in version 1.14, 35 are namespaced and 24 are not.
Also in the above output, pay attention to ```SHORTNAME```. Most of the resources have a short name too.
For example, namespace is ns and pods is po.

What is pod in k8s?
- a group of one or more containers
- a basic unit of deployment in k8s
- can run docker and many other type of container runtime
- each pod will get a dynamic IP, each container in pod will get a port. So container internally is accessible at <pod_ip>:<container_port>
- all containers in pod share network, namespace, resources, and storage

Let's get some explanation on pod. ```explain``` work on all other k8s resources.
```
kubectl explain pods
kubectl explain svc
```


## First kubernetes deployment

Let's deploy a web nginx servers container from focker.io. Our deployment name is ngixweb, generator optiona is optional ( to avoid a warning message about a method being deprecated) and image used is docker.io/nginx:latest
```
$ kubectl create deploy nginx-web  --image=nginx:latest
deployment.apps/nginx-web created

$ kubectl get deploy -o wide
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS       IMAGES                       SELECTOR
hello-minikube   1/1     1            1           83d   hello-minikube   k8s.gcr.io/echoserver:1.10   run=hello-minikube
nginx-web        1/1     1            1           40s   nginx            nginx:latest                 app=nginx-web
```
Our first deployment nginx-web is created. Since we did not mention, how many pods/replica set needs to be created in this deployment, only one pod/replica set is created. It is created in the default namespace.
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

There is still more to come. 
For now, we have created nginx-web deployment to start a single container inside a single pod.
Can we test it now as we see above ```kubectl get pods -o wide``` shows pod IP 172.17.0.5?
No, this is internal pods IP not accessible by the outside world.
We need to expose or deployment as a k8s service. Let's do it.
Below command created service with an IP (each service will have its own IP) and port 80 that will be accessible via node port IP ( Minikube Linux VM IP, not your PC IP)

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

Sometime, we need to see details of few resources next to each other. We can use ```get``` command with comma-separated resource names. Ahere is also an ```all``` option that is all we need most of the time
```
$ kubectl get services,pods,nodes -n default

$ kubectl get all

$ kubectl get all --all-namespaces
```

Let's close tis section with ```describe``` command. It prints a detailed description of the selected resources, including related resources such as events or controllers.
```
$ kubectl describe deployment nginx-web
$ kubectl describe pod nginx-web-66964bcbcd-vrvnx
$ kubectl describe nodes                # Also show resource CPU and Memory utilization by pods
$ kubectl describe nodes |grep Runtime  # which is Container runtime being used?
$ kubectl describe pod,services
$ kubectl describe pod,services --all-namespaces
```

### Scaling up replica of first deployment
Let's say, we need to scale up our deployment nginx-web from 1 to 3. 
Save deployment yaml in a file
Change ```replicas: 1``` to ```replicas: 3``` in ```spec:``` section

```
$ kubectl get pods,rs
$ kubectl get deploy nginx-web -o yaml >nginx-web-deployment.yaml
$ vi nginx-web-deployment.yaml       # Change replicas to 3
$ kubectl apply -f nginx-web-deployment-new.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.extensions/nginx-web configured
$ kubectl get pods,rs
NAME                              READY   STATUS              RESTARTS   AGE
hello-minikube-56cdb79778-sdbqg   1/1     Running             3          84d
nginx-web-66964bcbcd-h79tt        0/1     ContainerCreating   0          7s
nginx-web-66964bcbcd-vrvnx        1/1     Running             0          140m
nginx-web-66964bcbcd-xzvk7        0/1     ContainerCreating   0          7s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.extensions/hello-minikube-56cdb79778   1         1         1       84d
replicaset.extensions/nginx-web-66964bcbcd        3         1         1       140m

$ 
```
In the above output, 2 additional pods are being created, In a few seconds, 3 replicas/pods will be running. Since minikube is a single node cluster, all of them will be running on the same node ( repeat previous command with -o wide).
In multi-node cluster, they will be starting on more that one node depending on policies.

Repeat above procedure and scale it down from three to one again.

We know basics, now let use create our own container and create our second deployment

## Creating a docker container
I highly recomment to [create your own container by following steps I have a document on another page](https://github.com/nansari/docker/blob/master/README.md) . I you do not want to do it for some reason, you can continue to do the second deployment by using images I have already created


## Second kubernetes deployment
Now we will create second deployment centos-web to deploy pods running Apache httpd web server build on vanilla centos container.

```
$ kubectl create deployment centos-web --image docker.io/nansari/centos-httpd:v1.0 
deployment.apps/centos-web created

$ kubectl get deploy centos-web
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
centos-web   0/1     1            0           16s

$ kubectl get deploy centos-web
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
centos-web   0/1     1            0           26s
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
centos-web   1/1     1            1           55s

$ kubectl get deploy centos-web -o wide
NAME         READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS     IMAGES                                SELECTOR
centos-web   1/1     1            1           3m16s   centos-httpd   docker.io/nansari/centos-httpd:v1.0   app=centos-web
 
$ kubectl get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
centos-web-86454b55b6-w6csg       1/1     Running   0          2m19s   172.17.0.6   minikube   <none>           <none>

```

Expose deployment centos-web as a service centos-web (same name as deployment) on service port 9040
```
$ kubectl expose deployment centos-web --port 9040 --type=NodePort
service/centos-web exposed

$ kubectl get service centos-web -o wide
NAME         TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
centos-web   NodePort   10.105.47.88   <none>        9040:30557/TCP   38s   app=centos-web

$ minikube ip
192.168.99.100
```
Our web service is accessible using http://192.168.99.100:30557 ? no, it is not.
But similar command worked in first deployment above, Isn't it?

Why did it work for the first deployment? Port number (80) given by ```--port``` ws used for service as well as the container. Remember, we have web server inside the container. nginx web server inside the container is exposing (listening) port 80 when container started. So port number 80 used for ```--port``` option qwork like this - create a service to use port number 80 and redirect traffic to port number 80 to container.

Why it did not work for this deployment? Because service 9040 traffic is being redirected to port 9040 of the container ( that is not up and listening, httpd web server is listening on 80)

Let's delete the service and recreate with appropriate options
```
$ kubectl delete service centos-web
service "centos-web" deleted
$ 
$ kubectl expose deployment centos-web --port 9040 --type=NodePort --target-port=80 --name=centos-web-svc
service/centos-web exposed
$ 
$ kubectl get service centos-web -o wide
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
centos-web-svc   NodePort   10.101.170.122   <none>        9040:31164/TCP   15s   app=centos-web
```
Service it accessible on http://192.168.99.100:31164 now. I see a web page showing in 'Blue Docker Containerized Web Server'
We have also changed service name as centos-wev-svc as bydefault, it has same name as deployment. Confusing for beginers!

I do not like to remember this random port 31167 all the time. Let change it to something that we can remember easily - 40000. edit the service and change nodePort to 40000. Below command will open service definition in default test editor in YAML format. Update nodePort and save it.

```
$ kubectl edit service centos-web-svc
```
What happened?
It did not save? Because nodePort 40000 is out of valid service port 30000-32767. 
Whenever kubectl submit any yaml definition to api-server, it will validate it before accepting and executing it.
Let's change nodePort to 32000 and save it.

```
$ kubectl edit service centos-web-svc
error: services "centos-web-svc" is invalid
service/centos-web-svc edited
$ kubectl get service centos-web-svc -o wide
NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE     SELECTOR
centos-web-svc   NodePort   10.110.9.136   <none>        9040:32000/TCP   9m49s   app=centos-web
```

nodePort has changed to 32000. http://192.168.99.100:32000 is looking so Blue!

It is time to explore minikube dashboard. Below command will start minikube dashboard in your default browser (it does not work in IE)

```
minikube dashboard
```
Explore it. Check service, pods, replication sets, and deployments.
Click on 3 dots of centos-web and explore edit yaml option. Pay attention to strategy (RollingUpdate). We need this later. Click Cancel for now. Again click on 3 dots and scale it up to 8. Click ok. And quickly check ``` kubectl get pods,rs``` to see centos-web deployment scaling.

**Horizontal pod autoscalling** Deployment can scale to autoscale up and down. default criteria are 80% CPU utilization. that can be changed with --cpu-percentage
```
$ kubectl autoscale deployment centos-web --min=2 --max=10
horizontalpodautoscaler.autoscaling/centos-web autoscaled

$ kubectl get hpa
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
centos-web   Deployment/centos-web   <unknown>/80%   2         10        8          57s
 
 $ kubectl delete hpa centos-web
horizontalpodautoscaler.autoscaling "centos-web" deleted

$ kubectl autoscale deployment centos-web --min=2 --max=10 --cpu-percent=90
horizontalpodautoscaler.autoscaling/centos-web autoscaled
$ 

```

There is another way to scale up and down
```
$ kubectl scale --replicas=15 deployment centos-web
deployment.extensions/centos-web scaled

$ kubectl get pods |grep entos-web # Run it few times
```
Do you see the number of pods increased to 15 from 8 and then 5 of them are terminated?
Because of autoscaling limit, we have set previously.

What will happen if we delete few of pods?
```
$ kubectl delete pod centos-web-86454b55b6-4pwz5 centos-web-86454b55b6-69blh centos-web-86454b55b6-7cfhl
pod "centos-web-86454b55b6-4pwz5" deleted
pod "centos-web-86454b55b6-69blh" deleted
pod "centos-web-86454b55b6-7cfhl" deleted

$ kubectl get pods |grep centos-web
```

Replication Controller will bring new pods to match count!

What will happen if we will add a new label to centos-web-svc. Access http://192.168.99.100:32000/ before and after adding label
```
$ kubectl label service centos-web-svc new-label=test-label
service/centos-web-svc labeled
$ kubectl get service centos-web-svc -o yaml
```
Nothing will happen. Because any object in k8s can have any number of labels
Remember that objects with similar label will work togather. The way we deployed our centos-web, every object created under that deployment has same lavel ```app: centos-web```. So they are working togather.
Our service has not two labels ```app: centos-web``` and ```new-label: test-label```
Let's delete ```app: centos-web```
```
$ kubectl label service centos-web-svc app-
service/centos-web-svc labeled
$ kubectl get service centos-web-svc -o yaml
```
How about now, is the web page still opening? yes, because this service has selector ```app: centos-web``` to select pods to send the request. So it will continue to work. Though, if there would have been any other object using this service ( say ingress), that will not work.

Let's edit service and change ```app: centos-web``` in selector label to ```new-label: test-label```
```
$ kubectl edit service centos-web-svc 
service/centos-web-svc edited
$ 
```

Now the web page does not open.

Edit service again and restore selector and test it again.

## Rolling upgrade

I have already pushed centos-httpd:v2.0 that has a green index page. v1.0 has a blue index page

```
$ kubectl --record deployment.apps/centos-web set image centos-httpd=centos-httpd:v999 

$ kubectl get deployment
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
centos-web       8/10    5            8           100m
hello-minikube   1/1     1            1           84d
nginx-web        1/1     1            1           4h18m
$ kubectl get pods
NAME                              READY   STATUS              RESTARTS   AGE
centos-web-54ff9f4574-2jsl8       0/1     ErrImagePull        0          11s
centos-web-54ff9f4574-fd2lh       0/1     ContainerCreating   0          11s
centos-web-54ff9f4574-hmggp       0/1     ContainerCreating   0          11s
centos-web-54ff9f4574-hwsvg       0/1     ContainerCreating   0          11s
centos-web-54ff9f4574-prgg5       0/1     ErrImagePull        0          11s
centos-web-86454b55b6-88vp2       1/1     Running             0          23m
centos-web-86454b55b6-c8bxs       1/1     Running             0          47m
centos-web-86454b55b6-f2lhp       1/1     Running             0          23m
centos-web-86454b55b6-fqnj8       1/1     Running             0          47m
centos-web-86454b55b6-gj8h7       1/1     Running             0          31m
centos-web-86454b55b6-h25gd       1/1     Running             0          31m
centos-web-86454b55b6-s9t6g       1/1     Running             0          47m
centos-web-86454b55b6-w6csg       1/1     Running             0          100m
hello-minikube-56cdb79778-sdbqg   1/1     Running             3          84d
nginx-web-66964bcbcd-vrvnx        1/1     Running             0          4h18m
$ 
$ kubectl get pods
NAME                              READY   STATUS             RESTARTS   AGE
centos-web-54ff9f4574-2jsl8       0/1     ImagePullBackOff   0          39s
centos-web-54ff9f4574-fd2lh       0/1     ImagePullBackOff   0          39s
centos-web-54ff9f4574-hmggp       0/1     ImagePullBackOff   0          39s
centos-web-54ff9f4574-hwsvg       0/1     ImagePullBackOff   0          39s
centos-web-54ff9f4574-prgg5       0/1     ImagePullBackOff   0          39s
centos-web-86454b55b6-88vp2       1/1     Running            0          23m
centos-web-86454b55b6-c8bxs       1/1     Running            0          47m
centos-web-86454b55b6-f2lhp       1/1     Running            0          24m
centos-web-86454b55b6-fqnj8       1/1     Running            0          47m
centos-web-86454b55b6-gj8h7       1/1     Running            0          32m
centos-web-86454b55b6-h25gd       1/1     Running            0          32m
centos-web-86454b55b6-s9t6g       1/1     Running            0          47m
centos-web-86454b55b6-w6csg       1/1     Running            0          101m
hello-minikube-56cdb79778-sdbqg   1/1     Running            3          84d
nginx-web-66964bcbcd-vrvnx        1/1     Running            0          4h19m
$ kubectl get pods
NAME                              READY   STATUS             RESTARTS   AGE
centos-web-54ff9f4574-2jsl8       0/1     ImagePullBackOff   0          39s
centos-web-54ff9f4574-fd2lh       0/1     ImagePullBackOff   0          39s
centos-web-54ff9f4574-hmggp       0/1     ImagePullBackOff   0          39s
centos-web-54ff9f4574-hwsvg       0/1     ImagePullBackOff   0          39s
centos-web-54ff9f4574-prgg5       0/1     ImagePullBackOff   0          39s
centos-web-86454b55b6-88vp2       1/1     Running            0          23m
centos-web-86454b55b6-c8bxs       1/1     Running            0          47m
centos-web-86454b55b6-f2lhp       1/1     Running            0          24m
centos-web-86454b55b6-fqnj8       1/1     Running            0          47m
centos-web-86454b55b6-gj8h7       1/1     Running            0          32m
centos-web-86454b55b6-h25gd       1/1     Running            0          32m
centos-web-86454b55b6-s9t6g       1/1     Running            0          47m
centos-web-86454b55b6-w6csg       1/1     Running            0          101m
hello-minikube-56cdb79778-sdbqg   1/1     Running            3          84d
nginx-web-66964bcbcd-vrvnx        1/1     Running            0          4h19m
$ 

$ kubectl get deployment centos-web
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
centos-web   8/10    5            8           104m

$ kubectl rollout status deployment.apps/centos-web
# ^ even failed pod upgrade is counted as updated! Press control-c to come out of that

```
I used non existing version image for a rolling upgrade to simulate failed rollout. Also, the command has a few more problems!
Few pods were tried first. they failed. So, no further upgrade. 
Let's undo previously failed deployment. Based on the rollout strategy, it can happen automatically.

```
$ kubectl rollout undo deployment.apps/centos-web
deployment.apps/centos-web rolled back

$ kubectl get deployment centos-web
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
centos-web   10/10   10           10          105m

$ kubectl get pods
```

Correct rollout command is below. See deployment name is same as printed in ```kubectl get deploy```. It is used twice. Last option can also be writtedn as centos-httpd=docker.io/nansari/centos-httpd:v2.0
```
$ kubectl --record deployment.apps/centos-web set image deployment.apps/centos-web \ 
 centos-httpd=nansari/centos-httpd:v2.0

$ kubectl rollout status deployment.apps/centos-web
deployment "centos-web" successfully rolled out

```

http://192.168.99.100:32000/ is green now!

## Cleanup

Cleanup everything and retry it again!

```
kubectl delete deployment.apps/centos-web deployment.apps/hello-minikube deployment.apps/hello-minikube
kubectl delete service service/centos-web-svc service/hello-minikube service/nginx-web
kubectl delete hpa centos-web
minikube stop
minikube delete
```

## References
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

