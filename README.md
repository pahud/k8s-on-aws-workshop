

### prepare the C9 environment

https://github.com/aws-samples/aws-workshop-for-kubernetes/tree/master/01-path-basics/101-start-here#create-aws-cloud9-environment

choose "**Oregon(us-west-2)**" and "**Launch template with a new VPC**"



### install the build script in C9

```
aws s3 cp s3://aws-kubernetes-artifacts/lab-ide-build.sh . && \
chmod +x lab-ide-build.sh && \
. ./lab-ide-build.sh
```



### disable the AWS managed temporary credentials

![](https://raw.github.com/aws-samples/aws-workshop-for-kubernetes/master/resources/images/cloud9-disable-temp-credentials.png)

## create cluster with kops

```Bash
// change to a new C9 terminal tab

$ kops create cluster \
  --name example.cluster.k8s.local \
  --zones $AWS_AVAILABILITY_ZONES
```

### edit ig node

```bash
$ kops edit ig --name=example.cluster.k8s.local nodes
```

```yaml
apiVersion: kops/v1alpha2
kind: InstanceGroup
metadata:
  creationTimestamp: 2018-03-26T05:07:49Z
  labels:
    kops.k8s.io/cluster: example.cluster.k8s.local
  name: nodes
spec:
  image: kope.io/k8s-1.8-debian-jessie-amd64-hvm-ebs-2018-02-08
  # configure as spot instance with m3.medium
  machineType: t2.medium
  maxPrice: "0.02"
  maxSize: 2
  minSize: 2
  nodeLabels:
    kops.k8s.io/instancegroup: nodes
  role: Node
  subnets:
  - us-west-2a
  - us-west-2b
  - us-west-2c
```

###  update cluster

```Bash
$ kops update cluster example.cluster.k8s.local --yes
```



###  validate cluster



```Bash
$ kops  validate cluster
```

```
Using cluster from kubectl context: example.cluster.k8s.local

Validating cluster example.cluster.k8s.local

INSTANCE GROUPS
NAME                    ROLE    MACHINETYPE     MIN     MAX     SUBNETS
master-us-west-2a       Master  m3.medium       1       1       us-west-2a
nodes                   Node    m3.medium       2       2       us-west-2a,us-west-2b,us-west-2c

NODE STATUS
NAME                                            ROLE    READY
ip-172-20-43-65.us-west-2.compute.internal      node    True
ip-172-20-43-67.us-west-2.compute.internal      master  True
ip-172-20-75-169.us-west-2.compute.internal     node    True
```



###  cluster-info and version

```
$ kubectl cluster-info

$ kubectl version
```



### get nodes

```
$ kubectl get no
NAME                                           STATUS    ROLES     AGE       VERSION
ip-172-20-120-197.us-west-2.compute.internal   Ready     node      36m       v1.8.7
ip-172-20-59-219.us-west-2.compute.internal    Ready     node      36m       v1.8.7
ip-172-20-62-3.us-west-2.compute.internal      Ready     master    37m       v1.8.7
```



## Pod, Deployment, ReplicaSet and Service

#### create your first Pod

```
$ kubectl run nginx --image=nginx
deployment.apps "nginx" created
```

#### display Pod, Deployment, Service and replicaSet

```
$ kubectl get po,deploy,svc,rs
NAME                    READY     STATUS    RESTARTS   AGE
nginx-7c87f569d-bhmpj   1/1       Running   0          2m

NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           2m

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   100.64.0.1   <none>        443/TCP   42m

NAME              DESIRED   CURRENT   READY     AGE
nginx-7c87f569d   1         1         1         2m
```

#### describe a Pod

```
$ kubectl describe po/nginx-7c87f569d-bhmpj
Name:           nginx-7c87f569d-bhmpj
Namespace:      default
Node:           ip-172-20-59-219.us-west-2.compute.internal/172.20.59.219
Start Time:     Tue, 27 Mar 2018 15:18:21 +0000
Labels:         pod-template-hash=374391258
                run=nginx
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"nginx-7c87f569d","uid":"15ed5dc3-31d2-11e8-b40f-02b92302b376","a...
                kubernetes.io/limit-ranger=LimitRanger plugin set: cpu request for container nginx
Status:         Running
IP:             100.96.2.3
Controlled By:  ReplicaSet/nginx-7c87f569d
Containers:
  nginx:
    Container ID:   docker://fdcc19f63225b457d5c61182bae8596e959b0f6195c99e164cfb51d9115b1541
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:c4ee0ecb376636258447e1d8effb56c09c75fe7acf756bf7c13efadf38aa0aca
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 27 Mar 2018 15:18:34 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-9grs5 (ro)
Conditions:
  Type           Status
  Initialized    True 
  Ready          True 
  PodScheduled   True 
Volumes:
  default-token-9grs5:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-9grs5
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.alpha.kubernetes.io/notReady:NoExecute for 300s
                 node.alpha.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From                                                  Message
  ----    ------                 ----  ----                                                  -------
  Normal  Scheduled              6m    default-scheduler                                     Successfully assigned nginx-7c87f569d-bhmpj to ip-172-20-59-219.us-west-2.compute.internal
  Normal  SuccessfulMountVolume  6m    kubelet, ip-172-20-59-219.us-west-2.compute.internal  MountVolume.SetUp succeeded for volume "default-token-9grs5"
  Normal  Pulling                6m    kubelet, ip-172-20-59-219.us-west-2.compute.internal  pulling image "nginx"
  Normal  Pulled                 6m    kubelet, ip-172-20-59-219.us-west-2.compute.internal  Successfully pulled image "nginx"
  Normal  Created                6m    kubelet, ip-172-20-59-219.us-west-2.compute.internal  Created container
  Normal  Started                6m    kubelet, ip-172-20-59-219.us-west-2.compute.internal  Started container
```



#### clean up

```
$ kubectl delete deploy/nginx
deployment.extensions "nginx" deleted
```



### delete cluster

```sh
$ kops delete cluster --name example.cluster.k8s.local --yes
```


