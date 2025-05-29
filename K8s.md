# K8s from LinuxTips week

This content is based on YouTube video by LinuxTips Kubernetes week

### Installing kubectl 

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client
```

### Autocomplete 

#### on Bash

```
source <(kubectl completion bash) # configura o autocomplete na sua sessão atual (antes, certifique-se de ter instalado o pacote bash-completion).

echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanentemente ao seu shell.
```

#### on ZSH:

```
source <(kubectl completion zsh)

echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)"
```

### Alias

```
alias k=kubectl

complete -F __start_kubectl k
```

### Checking docker

Lets make sure we have the docker installed

```
[root@vsi-andre-rocky-eu-de ~]# docker -v
Docker version 24.0.2, build cb74dfc
```

#### Installing docker

```
# curl -fsSL https://get.docker.com | bash
```

#### Installing docker on Rocky Linux 

```
# dnf check-update
# sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# dnf install docker-ce docker-ce-cli containerd.io
# sudo systemctl start docker
# systemctl enable docker
# docker ps
```

#### Enabling docker for normal users

```
$ sudo usermod -aG docker [USER]
```


### Let's be kind, installing and creating a cluster

```
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64

# chmod +x ./kind

# sudo mv ./kind /usr/local/bin/kind

# kind create cluster

```

#### Creating a cluster with different name

```
[root@vsi-andre-rocky-eu-de ~]# kind create cluster --name andre
Creating cluster "andre" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-andre"
You can now use your cluster with:

kubectl cluster-info --context kind-andre

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```


Lets check cluster



```
[root@vsi-andre-rocky-eu-de ~]# kubectl config get-contexts
CURRENT   NAME         CLUSTER      AUTHINFO     NAMESPACE
*         kind-andre   kind-andre   kind-andre
          kind-kind    kind-kind    kind-kind
```

```
[root@vsi-andre-rocky-eu-de ~]# k config use-context kind-kind
Switched to context "kind-kind".

[root@vsi-andre-rocky-eu-de ~]# k config current-context
kind-kind
```


```
[root@vsi-andre-rocky-eu-de ~]# k config get-contexts
CURRENT   NAME         CLUSTER      AUTHINFO     NAMESPACE
          kind-andre   kind-andre   kind-andre
*         kind-kind    kind-kind    kind-kind
```

```
[root@vsi-andre-rocky-eu-de ~]# k config use-context kind-andre
Switched to context "kind-andre".
```

```
[root@vsi-andre-rocky-eu-de ~]# k config get-contexts
CURRENT   NAME         CLUSTER      AUTHINFO     NAMESPACE
*         kind-andre   kind-andre   kind-andre
          kind-kind    kind-kind    kind-kind
```



#### Getting current context cluster

```
# kubectl config current-context
kind-kind
```

```
[root@vsi-andre-rocky-eu-de ~]# kind get clusters
andre
kind
```

#### Deleting a cluster


```
[root@vsi-andre-rocky-eu-de ~]# kind delete clusters andre
Deleted clusters: ["andre"]
[root@vsi-andre-rocky-eu-de ~]# k config get-contexts
CURRENT   NAME        CLUSTER     AUTHINFO    NAMESPACE
          kind-kind   kind-kind   kind-kind
```

```
[root@vsi-andre-rocky-eu-de ~]# k config use-context kind-kind
Switched to context "kind-kind".
```

```
[root@vsi-andre-rocky-eu-de ~]# k config current-context
kind-kind
```




#### Creating a cluster with 03 nodes


```
[root@vsi-andre-rocky-eu-de ~]# cat << EOF > kind-3nodes.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
   - role: control-plane
   - role: worker
   - role: worker
EOF
```

```
[root@vsi-andre-rocky-eu-de ~]# ll
total 4
-rw-------. 1 root root 114 Jun  7 20:20 kind-3nodes.yaml
```

```
[root@vsi-andre-rocky-eu-de ~]# kind create cluster --name andre-multinodes --config kind-3nodes.yaml
Creating cluster "andre-multinodes" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-andre-multinodes"
You can now use your cluster with:

kubectl cluster-info --context kind-andre-multinodes

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

```
[root@vsi-andre-rocky-eu-de ~]# k config current-context
kind-andre-multinodes
```

```
[root@vsi-andre-rocky-eu-de ~]# k get no
NAME                             STATUS   ROLES           AGE    VERSION
andre-multinodes-control-plane   Ready    control-plane   2m6s   v1.24.0
andre-multinodes-worker          Ready    <none>          94s    v1.24.0
andre-multinodes-worker2         Ready    <none>          94s    v1.24.0
```






### Kubectl commands

#### Kubectl nodes

```
[root@vsi-andre-rocky-eu-de ~]# kubectl get no
NAME                 STATUS   ROLES           AGE    VERSION
kind-control-plane   Ready    control-plane   168m   v1.24.0
```

#### Checking pods

```
[root@vsi-andre-rocky-eu-de ~]# k get pods -A
NAMESPACE            NAME                                          READY   STATUS    RESTARTS   AGE
kube-system          coredns-6d4b75cb6d-8fnl5                      1/1     Running   0          11m
kube-system          coredns-6d4b75cb6d-9t824                      1/1     Running   0          11m
kube-system          etcd-andre-control-plane                      1/1     Running   0          11m
kube-system          kindnet-cncxt                                 1/1     Running   0          11m
kube-system          kube-apiserver-andre-control-plane            1/1     Running   0          11m
kube-system          kube-controller-manager-andre-control-plane   1/1     Running   0          11m
kube-system          kube-proxy-xkx8g                              1/1     Running   0          11m
kube-system          kube-scheduler-andre-control-plane            1/1     Running   0          11m
local-path-storage   local-path-provisioner-9cd9bd544-r7gzr        1/1     Running   0          11m
```

### Creating a NGINX pod

```
[root@vsi-andre-rocky-eu-de ~]# kubectl run nginx --image nginx
pod/nginx created
[root@vsi-andre-rocky-eu-de ~]#

[root@vsi-andre-rocky-eu-de ~]# k get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          8s
```

#### Describing pods

```
[root@vsi-andre-rocky-eu-de ~]# k describe po nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             andre-multinodes-worker/172.18.0.3
Start Time:       Wed, 07 Jun 2023 20:29:35 +0000
Labels:           run=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.2.2
IPs:
  IP:  10.244.2.2
Containers:
  nginx:
    Container ID:   containerd://1edcd9ef43892b20e33b47b15036a684b4b708b3e09c9d6806bae1e44524d1d1
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:af296b188c7b7df99ba960ca614439c99cb7cf252ed7bbc23e90cfda59092305
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 07 Jun 2023 20:29:40 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9d82w (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-9d82w:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m11s  default-scheduler  Successfully assigned default/nginx to andre-multinodes-worker
  Normal  Pulling    3m11s  kubelet            Pulling image "nginx"
  Normal  Pulled     3m6s   kubelet            Successfully pulled image "nginx" in 5.0419732s
  Normal  Created    3m6s   kubelet            Created container nginx
  Normal  Started    3m6s   kubelet            Started container nginx
  ```


### Connecting within the container

```
[root@vsi-andre-rocky-eu-de ~]# k exec -it nginx -- bash
root@nginx:/#
```

```
root@nginx:/usr/share/nginx/html# echo ANDRE > index.html
root@nginx:/usr/share/nginx/html#
```

```
root@nginx:/usr/share/nginx/html# cat index.html
ANDRE
```

```
root@nginx:/usr/share/nginx/html# curl localhost
ANDRE
```


### Creating a service to allow access from outside of the cluster


We will need to delete the pod and recreate it with the port parameter

```
[root@vsi-andre-rocky-eu-de ~]$ k delete po nginx
pod "nginx" deleted
```

Lets recreate the pod with port parameter

```
[root@vsi-andre-rocky-eu-de ~]$ kubectl run nginx --image nginx --port 80
pod/nginx created
```

Now when we describe the pod we can see the port especified 

```
[root@vsi-andre-rocky-eu-de ~]$ k describe po nginx | grep -i port
    Port:           80/TCP
```


Now we can create a service to expose the pod outside of the cluster

```
[root@vsi-andre-rocky-eu-de ~]$ kubectl expose po nginx
service/nginx exposed
```


Lets confirm the Service has been created

```
[root@vsi-andre-rocky-eu-de ~]$ k get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   39h
nginx        ClusterIP   10.96.197.190   <none>        80/TCP    4s
```


### Creating a service as a NodePort


Let delete the current service already created

```
[root@vsi-andre-rocky-eu-de ~]$ k get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   40h
nginx        ClusterIP   10.96.197.190   <none>        80/TCP    12m

[root@vsi-andre-rocky-eu-de ~]$ k delete svc nginx
service "nginx" deleted
```

lets now expose the pod nginx

```
[root@vsi-andre-rocky-eu-de ~]$ k expose po nginx --type NodePort
service/nginx exposed
```

Notice that now we have a Service of type nodeport

```
[root@vsi-andre-rocky-eu-de ~]$ k get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        40h
nginx        NodePort    10.96.197.138   <none>        80:30691/TCP   23s
```


To access this pod and the nginx content we basically need to check in which node that nginx pod is running on:

```
[root@vsi-andre-rocky-eu-de ~]$ kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE                      NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          2m15s   10.244.2.3   andre-multinodes-worker   <none>           <none>
```

As we see this pod is currently running on node andre-multinodes-worker worker
Now we need to describe the node andre-multinodes-worker and confirm the IP

```
[root@vsi-andre-rocky-eu-de ~]$ k describe no andre-multinodes-worker | grep -i IP
  InternalIP:  172.18.0.3
  ```

As we see the IP is 172.18.0.3 and now we acess it through the port that the Service was created to

```
[root@vsi-andre-rocky-eu-de ~]$ curl 172.18.0.3:30691
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

We can also get endpoints which is basically the endpoint of the Services

```
[root@vsi-andre-rocky-eu-de ~]$ k get endpoints
NAME         ENDPOINTS         AGE
kubernetes   172.18.0.5:6443   41h
nginx        10.244.2.3:80     108m
```



### Creating a deployment

```
[root@vsi-andre-rocky-eu-de ~]$  k create deployment strigus --image nginx --replicas 10
deployment.apps/strigus created

[root@vsi-andre-rocky-eu-de ~]$ k get deployments
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
strigus   10/10   10           10          76s
  
[root@vsi-andre-rocky-eu-de ~]$ k get pods
NAME                       READY   STATUS    RESTARTS   AGE
nginx                      1/1     Running   0          94m
strigus-58c99cf69c-4dxxt   1/1     Running   0          26s
strigus-58c99cf69c-8s6hj   1/1     Running   0          26s
strigus-58c99cf69c-gx2hp   1/1     Running   0          26s
strigus-58c99cf69c-jwxnc   1/1     Running   0          26s
strigus-58c99cf69c-lsjbx   1/1     Running   0          26s
strigus-58c99cf69c-mzgvb   1/1     Running   0          26s
strigus-58c99cf69c-nts9v   1/1     Running   0          26s
strigus-58c99cf69c-r6khm   1/1     Running   0          26s
strigus-58c99cf69c-tfvl6   1/1     Running   0          26s
strigus-58c99cf69c-vxd4q   1/1     Running   0          26s
```

#### Using Explain to understand a command in k8s

```
[root@vsi-andre-rocky-eu-de ~]$ k explain deployments
GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.

FIELDS:
  apiVersion	<string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind	<string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata	<ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec	<DeploymentSpec>
    Specification of the desired behavior of the Deployment.

  status	<DeploymentStatus>
    Most recently observed status of the Deployment.

```

```
[root@vsi-andre-rocky-eu-de ~]$ k explain svc
KIND:       Service
VERSION:    v1

DESCRIPTION:
    Service is a named abstraction of software service (for example, mysql)
    consisting of local port (for example 3306) that the proxy listens on, and
    the selector that determines which pods will answer requests sent through
    the proxy.

FIELDS:
  apiVersion	<string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind	<string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata	<ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec	<ServiceSpec>
    Spec defines the behavior of a service.
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status	<ServiceStatus>
    Most recently observed status of the service. Populated by the system.
    Read-only. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

```


## Creating a Pod with a YAML file

Create a file as extension Yaml

```
$ vim pod.yaml
```

Lets have the containt on it 

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: giropops
  name: giropops-2
spec:
  containers:
  - image: nginx
    name: giropops
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```





TODO

- [ ] buy croceries
- [ ] buy gitf
- [ ] buy beer
- [ ] study vim and macros
- [ ] drink water




 







