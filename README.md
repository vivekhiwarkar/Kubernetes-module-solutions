## Kubernetes module solutions

### Kubernetes Components

#### Task 1

``` {.sourceCode .bash}
minikube start --driver=virtualbox

kubectl get pods -n kube-system
kubectl describe pod kube-apiserver-minikube -n kube-system
kubectl describe pod kube-controller-manager-minikube -n kube-system
kubectl describe pod kube-scheduler-minikube -n kube-system
```
#### Task 2

``` {.sourceCode .bash}
https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/  --controller-string

All controllers: attachdetach, bootstrapsigner, cloud-node-lifecycle, clusterrole-aggregation, cronjob, csrapproving, csrcleaner, csrsigning, daemonset, deployment, disruption, endpoint, endpointslice, endpointslicemirroring, ephemeral-volume, garbagecollector, horizontalpodautoscaling, job, namespace, nodeipam, nodelifecycle, persistentvolume-binder, persistentvolume-expander, podgc, pv-protection, pvc-protection, replicaset, replicationcontroller, resourcequota, root-ca-cert-publisher, route, service, serviceaccount, serviceaccount-token, statefulset, tokencleaner, ttl, ttl-after-finished
Disabled-by-default controllers: bootstrapsigner, tokencleaner
```

### Workloads

#### Task 1

``` {.sourceCode .bash}
kubectl run ping-pod --image=busybox ping 8.8.4.4
```
#### Task 2

``` {.sourceCode .bash}
kubectl apply -f init-container.yaml
```

Wait for the pod to get into running state
For the first 30 seconds of the container's life, there is a /tmp/static_file file. So during the first 30 seconds, the command cat /tmp/static_file returns a success code. After 30 seconds, cat /tmp/static_file returns a failure code.
``` {.sourceCode .bash}
kubectl describe pod init-container
```
At the bottom of the output, there are messages indicating that the liveness probes have failed, and the containers have been killed and recreated.

#### Task 3 

``` {.sourceCode .bash}
kubectl apply -f workloads/nginx-daemonset.yaml
kubectl delete daemonset nginx-app
```

#### Task 4

``` {.sourceCode .bash}
kubectl apply -f workloads/nginx-daemonset.yaml
Change the image tag to 'latest-not-available' in workloads/nginx-daemonset.yaml and run the Task 3 command again to apply the update
kubectl rollout status daemonset.apps/nginx-app
Ctrl + C
kubectl rollout undo daemonset.apps/nginx-app
kubectl delete daemonset nginx-app
```

#### Task 5

``` {.sourceCode .bash}
kubectl apply -f postgre.yaml 
```

#### Task 6

``` {.sourceCode .bash}
docker cp workloads/kube-scheduler-modified.yaml kind-control-plane:/etc/kubernetes/manifests/
docker exec -it kind-control-plane /bin/bash
cd /etc/kubernetes/manifests/
mv kube-scheduler-modified.yaml kube-scheduler.yaml
Ctrl + D
kubectl run ping-google-pod --image=alpine ping 8.8.8.8
kubectl get pods -w
Revert back the image tag change made to kube-scheduler.yaml manifest
```

By default, Kubernetes watches for any changes on static pods, and default scheduler is one of them. 
This means right after you updated kube-scheduler pod changes should automatically be applied in some time.
In the end you should see all system pods running without any issues

### Kubernetes objects:

#### Task 1

``` {.sourceCode .bash}
kubectl apply -f k8s-objects.yaml
Manually change the label in the pod template of nginx-deployment-replicas.yaml file & run the above command again to update those changes
kubectl delete deployment nginx-deployment
```

By changing the label of the pods of replicaset which is managed by deployment, will make replicaset believe that the current state of cluster is not same as desired state because the desired state is determined by pod's label using label selector. 
So it'll spin up a new pod with the old label to match the current state with desired state.

### Networking:


#### Task 1

``` {.sourceCode .bash}
kubectl apply -f nginx-deployment.yaml
kubectl run test-pod-ns-one -n ns-one --image=busybox:latest ping 8.8.8.8
Note down the Endpoint in below command
kubectl describe service -n ns-one nginx-service
kubectl exec -it -n ns-one test-pod-ns-one -- /bin/sh
wget http://(above-endpoint)
cat index.html
Ctrl + D
Experiment - Exposed the nginx service out of the kind cluster as well (Try http://localhost:80 on browser)
kubectl delete namespace ns-one
```

#### Task 2

``` {.sourceCode .bash}
kubectl apply -f networking/nginx-deployment.yaml
kubectl create namespace ns-two
kubectl run test-pod-ns-two -n ns-two --image=busybox:latest ping 8.8.8.8
Note down the Endpoint in below command
kubectl describe service -n ns-one nginx-service
kubectl exec -it -n ns-two test-pod-ns-two -- /bin/sh
wget http://(above-endpoint)
cat index.html
Ctrl + D
kubectl delete namespace ns-one
kubectl delete namespace ns-two
```

#### Task 3


``` {.sourceCode .bash}
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80
kubectl run busybox --rm -ti --image=busybox -- /bin/sh
wget --spider --timeout=1 nginx
kubectl apply -f networking/nginx-policy.yaml
kubectl run busybox --rm -ti --image=busybox -- /bin/sh
wget --spider --timeout=1 nginx
kubectl run busybox --rm -ti --labels="access=true" --image=busybox -- /bin/sh
wget --spider --timeout=1 nginx
kubectl delete deployment nginx
kubectl delete sevice nginx
```

### Configurations:

#### Task 1

``` {.sourceCode .bash}
kubectl apply -f config/configmap.yml
kubectl apply -f config/config-pod.yml
kubectl exec --stdin --tty nginx-pod -- /bin/sh
echo $name
kubectl delete pod nginx-pod
kubectl delete configmap cm-one
```

Created a config map with key-value pair and an nginx pod with env variable's value that is referenced from config map.
From nginx pod, tried printing the environment variable using echo $envname and it printed the correct value.

#### Task 2

``` {.sourceCode .bash}
kubectl apply -f configuration/secret-pod.yaml
kubectl exec -it test-pod -- /bin/bash
env
Look for environment variable 'PASSWORD' with value as 'abvxwt11'
Ctrl + D
kubectl delete pod test-pod
kubectl delete secret secret-example
```


#### Task 3

``` {.sourceCode .bash}
echo -n 'Vml2ZWsgSGl3YXJrYXI=' > config/password.txt
kubectl create secret generic pass-example-file --from-file=password=config/password.txt
kubectl get secret pass-example-file -o yaml
Observe that the value to password key is decoded
kubectl create secret generic pass-example-literal --from-literal=password='Vivek Hiwarkar'
kubectl get secret pass-example-literal -o yaml
Observe that the value to password key is decoded
```

Created secret using --from-literal and via files. 
Values are required to be base64 encoded while creating using file but when using --from-literal,
values can be plain text and k8s will encode it automatically.


### Storage

#### Task 1

``` {.sourceCode .bash}
https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/
```

#### Task 2 

``` {.sourceCode .bash}
kubectl apply -f storage/two-containers.yml
kubectl exec -c test-container-1 -it test-pod -- /bin/sh
echo "Vivek Hiwarkar" >> /cache/test.txt
Ctrl + D
kubectl exec test-pod -c test-container-2 -- cat /cache/test.txt
kubectl delete pod test-pod
```

#### Task 3

``` {.sourceCode .bash}
https://kubernetes.io/docs/concepts/storage/volumes/#hostpath
kubectl apply -f storage/hostPath-example.yaml
kubectl exec -it test-webserver -- ls /var/local/
kubectl exec -it test-webserver -- ls /var/local/aaa
kubectl describe pod test-webserver
Note down the node pod is scheduled on
docker exec -it (node-name) ls /var/local/
docker exec -it (node-name) ls /var/local/aaa
```

Drawbacks of hostPath as volume is that in multinode cluster, pods access the same path on all the nodes and will expect to have the same data on all the nodes. 
It's not possible as they are on different machines. Also, there are security risks as well where a container can access secure details and can attack other parts of the cluster.

### Security

#### Task 1

``` {.sourceCode .bash}
https://kubernetes.io/docs/reference/access-authn-authz/rbac/
https://stackoverflow.com/questions/47973570/kubernetes-log-user-systemserviceaccountdefaultdefault-cannot-get-services
kubectl apply -f security/kanister-pod.yml
kubectl create rolebinding pod-reader-pod --role=get-pods --serviceaccount=default:default
kubectl exec -it kanister-pod -- kubectl get pods
```

#### Task 2

``` {.sourceCode .bash}
--authorization-mode flag sets the authorization mechanism for k8s cluster.
```

#### Task 3

In each namespace, a service account is created by default with the name default and it will be mounted as a volume in all the pods in that namespace.

