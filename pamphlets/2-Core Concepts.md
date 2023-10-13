## 9 - Core Concepts Section Introduction

## 10 - Download Presentation Deck for this section
Please note that some slides are animated so content may not have exported correctly. Kindly use the slides as a reference for commands.

## 11 - Cluster Architecture
- Cargo ships: does the actual work of carrying containers
- Control ships: responsible for monitoring and managing the cargo ships

The K8S cluster consists of a set of nodes which may be physical or virtual, on premise or on cloud, that host apps in the form of containers.
This relates to the cargo ships in the analogy.

The worker nodes in the cluster are ships that can load containers.

But somebody needs to load the containers on the ships and not just load, but to plan how to load, identify the right ships, 
store information about the ships, monitor and track the location of containers on the ships, manage the whole loading process and ... .
These are done by the control ships that host different offices and departments, monitoring equipments, communication equipments, cranes for
moving containers between ships and ... . The control ships relate to the master node in the k8s cluster. The master node is responsible for
managing the k8s cluster, storing info regarding the different nodes, planning which containers go where, monitoring the nodes and containers
on them and ... .

The master node does all of this, using a set of components together known as the control plane components.

### ETCD cluster
We need to maintain info about the different ships, what container is on which ship and what time it was loaded and ... . All of these are stored
in a highly available key-value store, known as ETCD.

### Schedulers(kube-scheduler)
When cargo ships(worker nodes) arrive, the cargo ship(master node) loads containers on them using cranes. The cranes identify the containers
that need to be placed on ships. It identifies the right cargo ship based on it's size, it's capacity, the number of containers already on the ship and
any other conditions such as the destination of the ship, the type of containers it is allowed to carry and ... . Those are schedulers
in a k8s cluster. A scheduler identifies the right node to place a container on, based on the containers resource requirements, the worker nodes capacity
or any other policies or constraints such as taints and tolerations or node affinity rules that are on them.

---

There are different offices in the dock that are assigned to special tasks or departments. For example, the operations team takes care
of ship handling, traffic control and ... . They deal with issues related to damages, the route the different ships take and ... .

The cargo team takes care of containers. When containers are damaged or destroyed, they make sure new containers are made available.

The services office takes care of IT communications between different ships(nodes). Similarly, in K8S we have controllers available that take care
of different areas. The node controller takes care of nodes. They're responsible for onboarding new nodes to the cluster, handling situations where
nodes become unavailable, or get destroyed.
The **replication controller** ensures that the desired number of containers are running at all times in a replication group.

---

How do these components communicate with each other? How does one office reach to another office and who manages them all at a high level?

A: The **kube API server** is the primary management component of k8s. It's responsible for orchestrating all operations within the cluster.
It exposes the K8S API which is used by external users to perform management operations on the cluster as well as various controllers to monitor
the state of the cluster and make necessary changes as required. And by the worker nodes to communicate with the server.

---
We're working with containers and they are everywhere. So we need everything to be container compatible. Our applications are in form of containers.
The different components that form the entire management system on the master node could be hosted in the form of containers. The DNS service,
networking solution can all be deployed in the form of containers. So we need this software that can run containers and that's the container
runtime engine, a popular one being docker. So we need docker or it's supported equivalent installed on all the nodes in the cluster, including
the master nodes, if you wish to host the controlling components as containers. Now it doesn't always have to be docker. K8S supports other container runtime
engines like containerD, rkt.

---
Now let's focus on cargo ships.

Every ship has a captain. The captain is responsible for managing all activities on these ships. The captain is responsible for liasing with the master ships,
starting with letting the master ship know that they're interested in joining the group, receiving info about the containers to be loaded
on the ship and loading the appropriate containers as required, sending reports back to the master about the status of this ship and the status of containers
on the ship.

The captain of the ship is the kubelet in K8S. A kubelet is an agent that runs on each node in a cluster. It listens for instructions from the
kube API server and deploys or destroys containers on the nodes as required. The **kube API server**(which is on the master node) periodically fetches status reports
from the kubelet to monitor the status of nodes and containers on them.

The kubelet was more of a captain on the ship that manages containers on the ship. But the applications running on the worker nodes, need to be able
to communicate with each other. For example, you might have a web server running in one container on one of the nodes and a DB server
running on another container on another node. How would the web server reach the DB server from the other node?
Communication between worker nodes are enabled by another component that runs on the worker node known as the **kube proxy service**.
The kube proxy service ensures that the necessary rules are in place on the worker nodes to allow the containers running on them to reach each other.

---
recap:

We have master and worker nodes:
- **On master node:** we have the ETCD cluster which stores info about the cluster. We have the kube scheduler that is responsible
for scheduling apps or containers on nodes. We have different controllers that take care of different functions like the node controller,
replication controller and ... . The kube API server that is responsible for orchestrating all operations within the cluster. 
- **On the worker node:** We have the kubelet that listens for instructions from the kube API server and manages containers.
The kube proxy that helps in enabling communication between services within the cluster.

Let's look at the components in k8s control plane.

## 12 - ETCD For Beginners
When you start etcd, it starts a service that listens on port 2379 by default. You can then attach any clients to the ETCD service to store and retrieve
info. A default client that comes with ETCD is the ETCD control client which is a command line client.

Note: `etcdctl` itself could be in v3 but the API version could be v2! This is true for old versions.

## 13 - ETCD in Kubernetes
Every info you see when you run: `kubectl get` command, is from the etcd server. Every change you make to the cluster like adding additional nodes,
deploying pods or replica sets, are updated in the etcd server.

Note: Only once it's updated in the etcd server, is the change considered to be complete.

Depending on how you set up your cluster, etcd is deployed differently.

Two types of k8s deployments:
- deployed from scratch
- kubeadm tool

The practice test environments are deployed using the kubeadm tool and later in the course, when we set up a cluster, we set it up from scratch.

### Setup - manual
If you set up your cluster from scratch, then you deploy etcd by downloading the etcd binaries yourself, installing the binaries and configuring etcd
as a service in your master node yourself. There are many options passed into this service. A number of them relate to certificates.
One option is `--advertise-client-urls https://${INTERNAL_IP}:2379` which is the address on which etcd listens. In this examaple, it happens to be
on the IP of the server and on port 2379 which is the default port on which etcd listens. This is the url that should be configured on the
kube API server when it tries to reach the etcd server.

### Setup - kubeadm
If you set up your cluster using kubeadm, then kubeadm deploys the etcd server for you as a pod in the kube-system namespace.

To list all keys stored by k8s, run:
```shell
kubectl exec etcd-master –n kube-system etcdctl get / --prefix –keys-only
```

### Explore etcd
K8s stores data in the specific directory structure. The root directory is a registry and under that, you have the various k8s constructs such as
minions or nodes, pods, replica sets, deployments and ... .

### ETCD in HA environment
In a high availability env, you will have multiple master nodes in your cluster. Then you will have multiple etcd instances spread across
the master nodes. In that case, make sure the etcd instances know about each other by setting the right parameter in etcd config and that param is
`--initial-cluster` which is where you must specify the different instances of the etcd service.

## 14 - ETCD Commands Optional
(Optional) Additional information about ETCDCTL Utility

ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3. By default its set to use Version 2. 
Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:

```shell
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

Whereas the commands are different in version 3

```shell
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put

```

To set the right version of API set the environment variable `ETCDCTL_API` command

```shell
export ETCDCTL_API=3
```

When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When 
API version is set to version 3, version 2 commands listed above don't work.

Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. 
The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of 
this course. So don't worry if this looks complex:

`--cacert /etc/kubernetes/pki/etcd/ca.crt     
--cert /etc/kubernetes/pki/etcd/server.crt     
--key /etc/kubernetes/pki/etcd/server.key`


So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:

```shell
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
```

## 15 - KubeAPI Server
primary management component in K8S.

When you run a kubectl command, the kubectl utility is reaching to the kubeapi-server. The kubeapi server first authenticates the req and validates it.
It then retrieves the data from the etcd cluster and responds back with the requested info. You don't really need to use the kubectl command line,
instead, you could also invoke the APIs directly by sending a POST req.

For example when you want to create a pod, you write the command with kubectl, the req is authenticated first and then validated.
In this case, the API server creates a pod object without assigning it to a node. Updates the info in the etcd server, updates the user that the pod
has been created(sending back the res). The scheduler continuously monitors the API server and realizes that there is a new pod with no node assigned.
The scheduler identifies the right node to place the new pod on and communicates that back to the kube-apiserver. The API server then
updates the info in the etcd cluster. Then the API server passes that info to the kubelet in the appropriate worker node. The kubelet then creates
the pod on the node and instructs the container runtime engine to deploy the application image. Once done, the kubelet updates the status
back to the API server and the API server then updates the data back in the etcd cluster.

A similar pattern is followed, everytime a change is requested. The kubeapi-server is at the center of all the different tasks that needs to be
performed to make a change in the cluster.

The kubeapi server is responsible for:
1. authenticate user
2. validate req
3. retrieve data from etcd
4. update data in etcd
5. scheduler
6. kubelet

Note: The kubeapi server is the only component that interacts directly with the etcd data store.

The other components like scheduler, kube controller manager and kubelet use the API server to perform updates in the cluster in their respective areas.

### Installing kube-api server
IF you boostrapped your cluster using kubeadmin tool, then you don't need to know this, but if you're setting up the hardware, then the kube-api server
is available as a binary in the k8s release page. Download it and configure it to run as a service on your k8s master node.

The kubeapi server is run with a lot of parameters.

All of the various components will have certificates associated with them.

Some of the options of kubeapi-server:

`--etcd-servers=https://...`: This is where you specify the location of the etcd servers. This is how kube-apiserver connects to the etcd servers.

### view api-server - kubeadm
Viewing the kube api-server in an existing cluster, depends on how you set up your cluster. If you set it up with kubeadmin tool, the kubeadmin deploys
the kube-apiserver as a pod in the kube-system namespace on the master node. You can see the options within the pod definition file located at
`etc/kubernetes/manifest` folder.

In a non-kubeadmin setup, you can inspect the options by viewing the kube-api-server service located at `etc/systemd/system/kube-apiserver.service`.
You can also see the running process and the effective options, by listing the process on the master node and searching for kube-apiserver.
```shell
ps -aux | grep kube-apiserver
```

## 16 - Kube Controller Manager
### Controller

Kube Controller Manager manages various controllers in k8s.

A controller is like an office or department within the master ship(node), that have their own set of responsibilities, such as:
an office for the ships would be responsible for monitoring and taking necessary actions about the ships whenever a new ship arrives or when a 
ship leaves or get destroyed. Another office could be one that manages the containers on the ships. It takes care of containers that are 
damaged or fall off ships. So these offices are:
1. continuously on the lookout for the status of the ships
2. take necessary actions to remediate the situation

In k8s terms, a controller is a process that continuously monitors the state of various components within the system and works towards bringing
the whole system to the desired functioning state.

#### Node controller
For example, the node controller is responsible for monitoring the status of the nodes
and taking necessary actions to keep the applications running. It does that through the kube API server. The node controller checks the status
of nodes every 5 seconds. That way the node controller can monitor the health of the nodes. If it stops receiving heartbeat from a node,
the node is marked as unreachable. But it waits for 40 seconds before marking it unreachable.
After a node is marked unreachable, it gives it 5 minutes to come back up. If it doesn't, it removes the PODs assigned to that node and provisions
them on the healthy ones if the PODs are part of a replica set.

#### Replication controller
Responsible for monitoring the status of replica sets and ensuring that the desired number of pods are available at all times within the set.
If a pod dies, it creates another one.

There are other controllers. Whatever concepts we have seen so far in k8s such as deployments, services, namespaces or persistent volumes and
whatever intelligence is built into these constructs, it is implemented through these various controllers. So this is kinda like a brain
behind a lot of things in k8s.

How do you see these controllers and where are they located in your cluster?

They're all packaged into a single process known as the **k8s controller manager**. When you install the k8s controller manager,
the different controllers get installed as well.

### View kube-controller-manager options - kubeadm
How do you view the kube controller manager server options?

Again it depends on how you set up your cluster?

If you set it up with the kube admin tool, kube admin deploys the kube controller manager as a pod in the kube-system namespace on the master node.

You can see the options within the pod definition file located at `etc/kubernetes/manifests/kube-controller-manager.yaml`.

### View kube-controller-manager options
In a non kube admin setup, you can inspect the options by viewing the kube controller manager service located at the services directory at:
`etc/systemd/system/kube-controller-manager.service`.

You can also see the running process and the effective options by listing the processes on the master node and searching for `kube-controller-manager`:
```shell
ps -aux | grep kube-controller-manager
```

## 17 - Kube Scheduler
Responsible for scheduling pods on nodes. Remember the scheduler is only responsible for **deciding** which pods goes on which node.
It doesn't actually place the pod on the nodes. That's the job of the kubelet. The kubelet or the captain on the ship,
is who creates the pod on ships(worker nodes). The scheduler only decides which pod goes where.

### kube-scheduler
Q: Why do you need a scheduler?

A: When there are many ships and many containers, you wanna make sure that the right container ends up on the right ship. For example there could be
different sizes of ships and containers. You wanna make sure the ship has sufficient capacity to accommodate those containers.

In K8S, the scheduler decides which nodes the pods are placed on depending on certain criteria. You may have pods with different resource
requirements. You can have nodes in the cluster dedicated to certain applications.

Q: How does the scheduler assign these pods?

A: The scheduler looks at each pod and tries to find the best node for it. The scheduler goes through 2 phases to identify the best node for the pod:
1. filter nodes: the scheduler tries to filter out the nodes that do not fit the profile for this pod. For example, the nodes that do not have
sufficient CPU and memory resources requested by the pod. Some of the nodes are filtered out in this phase.
2. rank nodes: rank the nodes to identify the best fit for the pod. It uses a priority function to assign a score to the nodes on a scale of zero to 10
For example, the scheduler calculates the amount of resources that would be free on the nodes after placing the pod on them. If a node would have more
resources after placing the pod, it gets the pod(in other words, the node with more resources wins). This can be customized and you can write your
own scheduler as well

At a high level, scheduler is a process.

### Installing kube-scheduler
How do you install the kube-scheduler?

Download the kube-scheduler binary and run it as a service. When you run it as a service, you specify the scheduler configuration file.

How do you view the kube-scheduler server options?

If you set it up with kubeadm tool, the kubeadm tool deploys the kube-scheduler as a pod in the kube-system namespace on the master node.
You can see the options in the pod definition file located at `/etc/kubernetes/manifests/kube-scheduler.yaml`.

You can also see the running process and the effective options by listing the processes on the master node and searching for kube-scheduler:
```shell
ps -aux | grep kube-scheduler
```

## 18 - Kubelet
Kubelet is like the captain on the ship. They're responsible for doing all the paperwork necessary to become part of the cluster. They're the sole
point of contact from the mastership. They load or unload containers on the ship as instructed by the scheduler on the master. They also
send back reports at regular intervals on the status of the ship and the containers on them.

The kubelet in the k8s worker node, registers the node with a k8s cluster. When it receives instructions to load a container or a pod on the node,
it requests the container runtime engine, which may be docker, to pull the required image and run an instance. The kubelet then continues to monitor
the state of the pod and containers in it and reports to the kube api server on a timely basis.

If you use kube admin tool(kubeadm) to deploy your cluster, it does not automatically deploy kubelet. Now that's the difference from other components.
You must always manually install the kubelet on your worker nodes. Download the installer and run it as a service.

You can view the running kubelet process and the effective options by listing the processes on the worker node and searching for kubelet:
```shell
ps -aux | grep kubelet
```

## 19 - Kube Proxy
Within a k8s cluster, every pod can reach every other pod. This is accomplished by deploying a pod networking solution to the cluster.
A pod network is an internal virtual network that spans across all the nodes in the cluster to which all the pods connect to. Through this
network, they're able to communicate with each other.

There are many solutions for deploying such a network.

Let's say there is a web app deployed on first node and the DB app deployed on the second. The web app can reach the DB simply by using the IP
of the pod, but there is no guarantee that the IP of the DB pod will always remain the same. A better way for the web app to access the DB which
is on another pod, is using a service. So we create a service to expose the DB app across the cluster.
The web app can now access the DB using the name of the service DB. The service also gets an IP address assigned to it. Whenever a pod tries to
reach the service using it's IP or name, it forwards the traffic to the backend pod, in this case, the DB. But what is this service and
how does it get an IP? Does the service join the same pod network? The service cannot join the pod network, because the service is not an actual thing,
it's not a container like pods, so it doesn't have any interfaces or an actively listening process. It is a virtual component that only lives
in the k8s memory. But then we also said that the service should be accessible across the cluster from any nodes. So how is that achieved?
That's where kube-proxy comes in. Kube-proxy is a process that runs on each node in the k8s cluster. it's job is to look for new services and everytime
a new service is created, it creates the appropriate rules on each node to forward traffic to those services to the backend pods.
One way it does this, is using IPtables rules. In this case, it creates an iptables rule on each node in the cluster to forward traffic heading
to the IP of the service(which in img is 10.96.0.12) to the IP of the actual pod(in img is 10.32.0.15).

So that's how kube-proxy configures a service(high level overview).

### installing kube-proxy
Download the binary and run it as a service.

### view kube-proxy - kubeadm
The kubeadm tool deploys kube-proxy as pods on each node. In fact, it is deployed as a daemonset, so a single pod is always deployed
on each node in the cluster.

## 20 - Recap PODs
Assumptions:
- app is developed and built into docker images
- the app docker image is available on a docker repository like docker hub, so that k8s can pull it down
- k8s cluster is set up. this could be a single-node setup or a multi-node setup, it doesn't matter. All the services need to be in a running state

With k8s, our ultimate aim is to deploy our app in the form of containers on a set of machines that are configured as worker nodes in a cluster.
However, k8s does not deploy containers directly on the worker nodes. The containers are encapsulated into a k8s object known as pods.

A pod is a single instance of an app. A pod is the smallest object that you can create in k8s.

When we want to scale our app, we need to add additional instances of our app to share the load(traffic). Now, where would you spin up
additional instances? Do we bring up new container instances within the same pod(new containers in a single pod)? No. We create new pod
all together with a new instance of the same app. So now we have two instances of our web app running on two separate pods on the same k8s system or node.

What if the user base further increases and your current node has no sufficient capacity?

Then you can always deploy additional pods on a new node in the cluster. You will have a new node added to the cluster to expand the cluster's
physical capacity.

Pods usually have a one-to-one relationship with containers running your application. To scale up, you create new pods and to scale down,
you delete existing pods. You do not add additional containers to an existing pod to scale your app.

### Multi-container pods
We know pods usually have a one-to-one relationship with containers. But are we restricted to having a single container in a single pod?

No, a single pod can have multiple containers except for the fact that they're usually not multiple containers of the same kind.
As we mentioned, if our intention was to scale our app, then we would need to create additional pods. But sometimes you might have a scenario
where you have a helper container that might be doing some kind of supporting task for our web app(main container), such as processing a user and they're data,
processing file uploads and ... and you want these helper containers to live alongside your application container. In that case,
you can have both of these containers part of the same pod. So that when a new application container is created, the helper container is also
created and when it dies, the helper also dies **since they're a part of the same pod**. The two containers can also communicate
with each other directly by referring to each other as `localhost` since they share the same network space. Plus they can easily share the
same storage space as well.

### pods again!
Let's assume we were developing a process or a script to deploy our app on a docker host. Then we would first deploy our app using
```shell
docker run python-app
```
When the load increases, we deploy more instances of our app by running the `docker run python-app` commands many more times. This works fine.
Now, sometime in the future, our app grows and gets complex. We now have a new helper container that helps our web app by processing or fetching
data from elsewhere. These helper containers maintain a one-to-one relationship with our app container and thus needs to communicate with the
app containers directly and access data from those containers. For this, we need to maintain a map of what app and helper containers are connected
to each other. We would need to establish network connectivity between these containers ourselves using links and custom networks. We would need to
create sharable volumes and share it among the containers. We would need to maintain a map of that as well. And most importantly, we would need to
monitor the state of the app container and when it dies, manually kill the helper container as well as it's no longer required.
When a new container is deployed, we would need to deploy the new helper container as well.

With pods, k8s does all of this for us automatically. We just need to define what containers a pod consists of, and the containers in a pod
by default will have access to the same storage, the same network namespace and same fit as in they will be created together and destroyed together.

Even if our app didn't happen to be so complex and we could live with a single container, k8s still requires you to create pods. But this is good
in the long run as your app is now equipped for architectural changes and scale in the future. However, also note that multi-pod containers(multi-container pods?)
are a rare use case and we're going to stick to single containers per pod.

### Kubectl
How to deploy pods?

what command `kubectl run nginx --image nginx` really does is it deploys a container by creating a pod. It first creates a pod automatically and deploys an instance
of the nginx docker image. But where does it get the application image from? For that, you need to specify the image name using the `--image` parameter.
The app image in this case the nginx image, is downloaded from the docker hub repository which is a public repo where latest docker images
of various applications are stored. You could configure k8s to pull the image from the public docker hub or a private repository within the organization.

When we create a pod, we haven't made it accessible to external users. You can access it internally from the node. We will see how to make the service
accessible to end users.

## 21 - PODs with YAML
A k8s definition file always contains four top(root) level fields. These are also required fields.
- apiVersion: version of the k8s api we are using to create the object. Depending on what we're trying to create, we must use the right
api version. Since we're using with pods, we will set the apiVersion to v1. Few other possible values are: apps/v1beta, extensions/v1beta and ... .
- kind: type of object we're trying to create.
- metadata: data about the object like it's name, labels and ... . This is in the form of a dictionary. The number of spaces for indentation doesn't matter
but should be the same for siblings. We can have any key and value pairs for `labels`. Let's say there are hundreds of pods running a frontend app
and hundreds of pods running a backend app or a DB. It will be difficult for you to group these pods once they're deployed. If you label them now as
frontend, backend or database, you will be able to filter the pods based on this label at a later point in time. Under `metadata`, you can only specify
name or label or anything else that k8s expects to be under metadata. You cannot add any other property as you wish under this. However, under `labels`,
you **can** have **any** kind of key value pairs as you see fit. So it's important to understand what each of these parameters expect.
- spec: depending on object we wanna create, this is where we would provide additional info to k8s pertaining to that object. This is gonna be different for
different objects.

Note: The value for `kind` is case-sensitive. For creating a pod, the value would be `Pod`.

The spacing in yaml would be two spaces or a tab. But it is recommended not to use tabs. So always stick to two spaces.

```shell
k get pods

k describe pod <pod-name>
```

## 22 - Demo PODs with YAML
If you're creating a new object, the `kubectl create` and `kubectl apply` commands work the same.
```shell
k appply -f <path to yaml file>
```

## 23 - Practice Test Introduction

## 24 - Demo Accessing Labs

## 25 - Accessing the Labs
All hands-on labs are hosted on KodeKloud. Use this link to register for the labs associated with this course.  Please make 
sure to use the same name as your profile in Udemy. That's how we know you are our Udemy student.

Note: You don't have to make any additional payment.

Link: https://uklabs.kodekloud.com/courses/labs-certified-kubernetes-administrator-with-practice-tests/

Apply the coupon code udemystudent151113

## 26 - Practice Test Pods
Link to the practical test: https://uklabs.kodekloud.com/topic/practice-test-pods-2/

## 27 - Practice Test Solution Optional

## 28 - Recap ReplicaSets
K8s controllers are the brain behind k8s. They are the processes that monitor k8s objects and respond accordingly.

One controller is the **replication controller**.

What is a replica and why do we need a replication controller?

Let's say we have a single pod. What if our app crashes and the pod fails? Users will no longer be able to access our app. To prevent this,
we would like to have more than one instance or pod running at the same time. So if one fails, we have another one. The replication controller
helps us run multiple instances of a single pod in the k8s cluster. Thus providing high availability. Does that mean you can't use a replication controller
if you plan to have a single pod?

No, even if you have a single pod, the replication controller can help by automatically brining up a new pod when the existing one fails.
Thus, **the replication controller ensures that the specified number of pods are running at all times, even if it's 1 or 100.**

Another reason we need replication controller is to create multiple pods to share the load across them. When the number of users increase.
we deploy additional pod to balance the load across two pods. If the demand further increases and we ran out of resources on the first node,
we could deploy additional pods across the other nodes in the cluster. So the replication controller spans across multiple nodes in the cluster.
It helps us balance the load across multiple pods on different nodes as well as scale our app when the demand increases.

Replication controller and replica set have the same purpose, but they're not the same. Replication controller is the older technology that is being
replaced by replica set. Replica set is the new recommended way to set up replication. There are minor differences in the way each works.

Create a replication controller: 28-1 folder.

**Note:** The `apiVersion` is specific to what object we're creating. In this case, replication controller is supported in k8s apiVersion v1.

For any k8s definition file, the `spec` section defines what's inside the object we're creating.

In this case, we know the replication controller creates multiple instances of a pod. But what pod? For this, we create a `template` section under
`spec` to provide a pod template to be used by the replication controller to create replicas. We can reuse the contents of pod-definition.yaml file
in 22-1 to populate the `template` section.

So we have nested two definition files together. The replication controller being the parent and the pod definition being the child.
```shell
kubectl create -f <path to rc definition yaml file>
```

When the replication controller is created, it first creates the pod(s) using the pod definition template as many as required(based on `replicas`)

```shell
kubectl get replicationcontroller # also shows the number of replicas(or pods) in the output
```

One major difference between replication controller and replica set is that replica set requires a `selector` definition.
The selector section helps the replicaset identify what pods fall under it. But why would you have to specify what pods fall under it, if you
have provided the contents of the pod definition file itself in the `template`?

Because replicaset can also manage pods that were not created as part of the replicaset creation. For example: There were pods created before the
creation of the replicaset that match labels specified in the `selector`, the replica set will also take those pods into consideration when
creating the replicas.

The `selector` is not a required field in replication controller. When you skip it, it assumes it to be the same as the labels provided in the
pod definition file.

The `matchLabels` selector matches the labels specified under it to the labels on the pods. The replicaset selector also provides other options
for matching labels that were not available in a replication controller.

```shell
k create -f <path to replicaset definition file>

k get replicaset
```

One of the use cases of replica sets is we can use it to monitor existing pods if we have them(pods) already created.

### Labels and selectors
Why do we label our pods and objects in k8s?

Let's say we deployed 3 instances of our web app as 3 pods. We would like to create a replication controller or replica set to ensure
that we have three active pods at any time. In case the pods are not created, the replica set will create them for us. The role of the replica set
is to monitor the pods and if any of them were to fail, deploy new one. The replica set is in fact a process that monitors the pods.
Now how does the replica set know what pods to monitor? There could be hundreds of other pods in the cluster running different apps. This is where
labeling our pods **during creation** comes in handy. We could now provide these labels(labels of pods) as a filter for replicaset under the `selector`
section and under `matchLabels` section there. Provide the same labels of the pods you wanna monitor. This way the replica set knows which pods
to monitor.

Same concept of labels and selectors is used in many other places throughput k8s.

Q: Let's say the pods are created already and now we wanna create a replica set to monitor the pods, to ensure the number of replicas are the same
as what we defined at all times. Now when the replication controller or replica set is created, it is not going to deploy a new instance of pod
as we have the right number of them with matching labels are already created. In that case, do we really need to provide the `template` section
in replica set definition? Since we're not expecting the replica set to create a new pod on deployment?

A: Yes, we do. because in case one of the pods were to fail in the future, the replica set needs to create a new one to maintain the desired number of
pods and to create it, the replica set needs the template definition section.

### Scale
How do we scale the replica set? How do we update the replica set to increase the number of replicas ?

There are number of ways:
1. update the number of replicas in the definition file. Then run: `kubectl replace -f <path to replicas set definition file>`.
2. run: `kubectl scale --replicas=<new number> -f <path to replicas set definition file>` or `kubectl scale --replicas=6 replicaset <name of the replicaset>`.
With this approach, we don't have to update the definition file.

Note: remember in approach 2, the number of replicas won't automatically get updated in the definition file. So the number of replicas in dentition file
would still be the same, even though we scaled the replicaset to have more replicas.

There are options available for automatically scaling the replica set based on load.

Note: With `kubectl replace -f replicaset-definition.yml` we replace or update the replica set.

## 29 - Practice Test ReplicaSets
TODO

## 30 - Practice Test ReplicaSets Solution Optional

## 31 - Deployments
How you might want to deploy your app in a production env?

You need not one but many instances of your app running for obvious reasons. Secondly, whenever newer versions of application builds become
available on the docker registry, you would like to upgrade your docker instances(container images) seamlessly. However, when you upgrade
your instances, you do not want to upgrade all of them at once. Because this may impact users accessing our apps. so you might
want to upgrade them **one after the other**. This kind of update is known as **rolling updates**.
Suppose one of the upgrades you performed, resulted in an unexpected error and you're asked to undo the recent change, you would like to be able
to **roll back** the changes that were recently carried out. Finally, say for example, you would like to make multiple changes to your environment,
such as upgrading the underlying web server versions, as well as scaling your environment and also modifying the resource allocations and ... .
You do not want to apply each change immediately after the command is run, instead, you would like to apply a pause to your environment, make the changes
and then resume, so that all the changes are rolled out together.

All of these capabilities are available with the k8s deployments.

We know with pod, we deploy single instance of our app. Each container is encapsulated in a pod. Multiple such pods are deployed using replication controllers
or replica sets and then deployments come higher in the hierarchy. The deployment provides us with the capability to upgrade the underlying instances
seamlessly using rolling updates, undo changes and pause and resume changes as required.

The contents of deployment definition file is exactly the same as replica set definition file except for the `kind`.

Note: The deployment automatically creates a replica set. So after creating a deployment, if you list the deployments, you will see a new replicaset
in the name of the deployment like `myapp-deployment-<some id>` as the name of the replicaset. Now the replicaset also create pods, so if you list
the pods you will see pods with the name of the deployments and replicaset as well, so the name of the pods would be
like: `myapp-deployment-<replicaset name id>-<some id>`

```shell
k get deployments

k get all # to list all the objects
```

## 32 - Certification Tip
Here's a tip!

As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you
might find it difficult to copy and paste YAML files from browser to terminal. Using the kubectl run command can help in generating a 
YAML template. And sometimes, you can even get away with just the kubectl run command without having to create a YAML file at all. 
For example, if you were asked to create a pod or deployment with specific name and image you can simply run the kubectl run command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML 
files. Try to use these as much as you can going forward in all exercises

Reference (Bookmark this page for exam. It will be very handy):

https://kubernetes.io/docs/reference/kubectl/conventions/

Create an NGINX Pod

```shell
kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

```shell
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

Create a deployment

```shell
kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

```shell
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

```shell
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.

```shell
kubectl create -f nginx-deployment.yaml
```

OR

In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

```shell
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

## 33 - Practice Test Deployments

## 34 - Solution Deployments optional

## 35 - Services
K8S services enable communication between various components within and outside of the application. K8S services help us connect apps together
with other apps or users. For example our app has groups of pods running various sections such as a group for serving frontend loads to users,
another group for running backend processes and a third group connecting to an external data source. It is services that enable connectivity
between these groups of pods. Services enable the frontend app to be made available to end users. It helps communication between these
groups of pods and helps in establishing connectivity to an external data source. Thus **services enable loose coupling between microservices
in our app.**

So one use case of services is making pods be able to communicate with each other through internal networking.

Let's look at some other aspects of networking.

### External communication
How do we as an external user, access the app in the pod?

We know k8s node has an IP address, like 192.168.1.2 in the img. Our laptop is on the same network as well, so it has an IP 192.168.1.10 .
The internal pod network is in the range 10.244.0.0 and the pod has an IP 10.244.0.2 . Clearly, we cannot ping or access the pod at address 10.244.0.2
as it's in a separate network. So what are the options to see the app?

If we were to SSH into the k8s node at 192.168.1.2, from the node, we would be able to access the pod's webapp by doing a curl or if the node
has a GUI, we would fire up a browser and see the web page in a browser following the address http://10.244.0.2 . But this is from inside
the k8s node and that's not what we really want. We want to be able to access the web app from our laptop without having to SSH into the node and simply
by accessing the IP of the k8s node. So we need sth in the middle to help us map requests to the node from our laptop, through the node, to the pod running
the web container. This is where k8s service comes into play.

One of the use cases of k8s services is to listen to a port on the node and forward request on that port to a port on the pod running the web app.
This type of service is known as a **node port service**, because the service listens to a port on the node and forward request to the pods.

Cluster IP service creates a virtual IP inside the cluster to enable communication between different services such as a set of frontend servers
to a set of backend servers.

Load balancer service provisions a load balancer for our app in supported cloud providers. An example is to distribute load across different
web servers in your frontend tier.

### Service - Nodeport
A service can map a port on the node to a port on the pod in that node. There are three ports involved in this service. **Remember these terms are
from the viewpoint of the service**. This is why the port of the service is just named **port**.
- the port on the pod where the actual web server is running(in img it's port 80) and it is referred to as the **target port**, because that is where
the service forwards their request to.
- the port on the service itself. it is simply referred to as the **port**. The service is in fact like a virtual server inside the node.
Inside the cluster, it has it's own IP address and that IP address is called the clusterIP of the service.
- the port on the node itself which we use to access the web server(app in the container) externally and that is known as the **nodeport** and in img
it's 30,008. That is because **node ports can only be in a valid range which by default is from 30,000 to 32,767**.

Note: Out of these three, the only mandatory field is **port**. If you don't provide a `targetPort`, it is assumed to be the same as `port` and if
you don't provide a `nodePort`, a free port in the valid range between 30000 and 32767 is automatically used. 

To understand the port values in 35-1 definition file, look at the img.

Currently, sth is missing in the definition file: There is nothing there to connect the service to the pod. We've just specified the `targetPort` but we
didn't mention the targetPort on which pod? There could be hundreds of other pods with web servers running on port 80.

To specify this, as we did with replica-sets previously, and a technique you will see very often in k8s, we will use labels and selectors to link
these(service and the pod) together. We know that the pod was created with a label. We need to use that label into the service definition file.
We use `selector` in `spec` just like replicaset and deployment definition files. Under `selector`, we provide a list of labels to identify the pod.

```shell
kubectl create -f service-definition.yaml

kubectl get services

curl http://192.168.1.2:30008 # http://<IP of the node>:<node port> 
```

So far we talked about a service mapped to a single pod. What do you do when you have multiple pods?

In a production env, you have multiple instances(pods) of your web app running for HA and load balancing purposes. These pods all have the same
labels. Now when the service is created, it looks for a matching pod with the label and finds three of them(in the case of img). The service
then automatically selects all the three pods as endpoints to forward the external request coming from the user. You don't have to do any additional
configuration to make this happen.

Q: What algo the service uses to balance the load across the three different pods?

A random algo. Thus the service acts as a built-in load balancer to distribute load across different pods.

Q: What happens when pods are distributed across multiple nodes? So we have web apps on pods that are on separate nodes in the cluster.

When we create a service, without us having to do any additional configuration, k8s automatically creates a service that spans across all the nodes
in the cluster and maps the targetPort to the same nodePort on all the nodes in the cluster. This way you can access your app using the IP of any
node in the cluster and using the same port number(in img is 30008). Look at the img, for doing curl, the IPs are different and we use the same port 30008.

So in any case, whether it be a single pod on a single node, multiple pods on a single node, or multiple pods on multiple nodes, the service
is created exactly the same without you having to do any additional steps during the service creation. When pods are removed or added,
the service is automatically updated, making it highly flexible and adaptive. Once created, you won't typically have to make any additional
configuration changes.

## 36 - Services Cluster IP
A full stack web app, typically has different kinds of pods hosting different parts of an app. A number of pods running a frontend web server,
another set of pods running a backend server, a set of pods running a key-value store and another set running a persistent DB like mysql.

The frontend server needs to communicate to the backend servers and backend servers communicate to DB and redis.

Q: What is the right way to establish connectivity between these?

**The pods all have an IP address assigned to them, but these IPs are not static. These pods can go down anytime and new pods are created all the time
and so you can't rely on these IP addresses for internal communication between the app.** Also what if the frontend pod at 10.244.0.3 needs to connect
to a backend service? Which of the three would it go to and who makes that decision?
A k8s service can help us group the pods together and provide a single interface to access the pods in a group.

For example a service created for the backend pods, will help group all the backend pods together and provide a single interface for other pods to
access this service. The requests are forwarded to one of the pods under the service randomly. This enables us to easily deploy a microservices-based app
on k8s cluster. Each layer can now scale or move as required without impacting communication between the various services.

Each service gets an IP and name assigned to it inside the cluster and that is the name that should be used by other pods to access the service.
This service is known as cluster IP.

Note: ClusterIP is the default type of `type` of a `kind: Service`. So if you didn't specify it, it will automatically assume the type to be ClusterIP.

```shell
kubectl create -f service.definition.yaml
kubectl get services
```
The service can be accessed by other pods using the IP or the service name.

## 37 - Services Loadbalancer
NodePort service helps us make an external facing app available on a port, on the worker nodes.

Let's say we have a 4-node cluster and to make the apps accessible to external users, we create nodePort services. NodePort services
help to receive traffic on the ports on the nodes and routing the traffic to the respective pods. But what URL would you give your end
users to access the apps?

You can access any of these two apps(in img) using IP of any of the nodes and the port that services are exposed on. That would be 4 IP and port
combinations for voting app and 4 ip and port combinations for the result app.

So even if your pods are only hosted on two of the nodes, they will still be accessible on the IPs of all the nodes in the cluster.

So you would share urls like http://192.168.56.70:30035 to your users to access the app. But that's not what the end users want.
They need a single url like http://voting-app.com and http://result-app.com to access the app.

One way to achieve this is to create a new VM for load balancer purposes and install and configure a load balancer on it like HA proxy or nginx or ... 
and then configure the LB to route traffic to the underlying nodes.

Setting all of that external load balancing and then maintaining that can be a tedious task. However if we were on a supported cloud platform like aws,
we could leverage the native LB of that cloud platform. K8S has support for integrating with the native LBs of certain cloud providers.

If you set the type of service to `LoadBalancer` in an unsupported environment like virtual box, then it would have the same effect as setting it to
node port.

## 38 - Practice Test Services
TODO

## 39 - Solution Services optional
TODO

## 40 - Namespaces

## 41 - Practice Test Namespaces

## 42 - Solution Namespaces optional
## 43 - Imperative vs Declarative
## 44 - Certification Tips Imperative Commands with Kubectl
## 45 - Practice Test Imperative Commands
## 46 - Solution Imperative Commands optional
## 47 - Kubectl Apply Command
## 48 - Heres some inspiration to keep going