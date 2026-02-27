# Section 3 - Scheduling

## 49 - Scheduling Section Introduction

## 50 - Download Presentation Deck for this section

## 51 - Manual Scheduling
### How scheduling works?
Every pod has a field called `nodeName` that by default is not set, k8s adds it automatically. The scheduler goes through all the pods
and looks for those that **do not have this property set**. **Those are the candidates for scheduling**. It then identifies the right node for the pod by
running the scheduling algorithm. Once identified, it schedules the pod on the node by setting the `nodeName` property to the name of the node by creating a
binding object.

### No scheduler!
If there is no scheduler to monitor and schedule nodes, what happens?

The pods continue to be in a `pending` state. What can you do about it? You can manually assign pods to nodes yourself.
Without a scheduler, the easiest way to schedule a pod, is to simply set the `nodeName` field to the name of the node in
your pod specification file while creating the pod. The pod then gets assigned to the specified node. You can only specify the nodeName at
creation time. What if the pod is already created and you want to assign the pod to a node? K8S won't allow you to modify the nodeName property of a pod.
So another way to assign a node to an existing pod is to create a **binding object** and send a POST request to the pod's binding API, thus mimicking
what the actual scheduler does.

In the binding object, you specify a target node(using `target`) with the name of the node(`target.node` like node02). Then send a POST req
to the pod's binding api with the `data` set to the binding object in a json format. So you must convert the yaml file into it's equivalent
JSON form(look at the slide's curl).

```shell
# get the master node pods
kubectl get pods --namespace=kube-system
```

## 52 - Practice Test Manual Scheduling
## 53 - Solution Manual Scheduling optional
## 54 - Labels and Selectors
Labels and selectors are a standard method to group things together.

Labels are properties attached to each item and We use `selector` to group(select) the objects matching the selector.

For each object, attach labels as per your needs.

### Annotations
While labels and selectors are used to group and select objets, annotations are used to record other details for informatory purpose.
For example, tool details like name, version, build information and ..., or contact details, phone numbers, email ids and ... that may be used
for some kind of integration purpose.

## 55 - Practice Test Labels and Selectors
## 56 - Solution Labels and Selectors Optional
## 57 - Taints and Tolerations
We will discuss about the pod to node relationship and how you can restrict what pods are placed on what nodes.

Taints and Tolerations are used to set restrictions on what pods can be scheduled on a node.

When the pods are created, k8s scheduler tries to place these pods on the available worker nodes(let's assume there is no restriction or
limitations for now). The scheduler places the pods across all the nodes to balance them out equally.

Now let's assume that we have dedicated resources on node 1 for a particular use case or application. So we would like only those pods that
belong to this application to be placed on node 1. First, we prevent all pods from being placed on node 1 by placing a taint on the node. Let's
call this taint, blue. **By default, pods have no tolerations which means unless specified otherwise, none of the pods can tolerate any taint.**
So in this case, none of the pods can be placed on node 1, as none of them can tolerate the taint blue. **This solves half of our requirements.
No unwanted pods are going to be placed on this node(node 1). The other half is to enable certain pods to be placed on this node.
For this, we must specify which pods are tolerant to this particular taint.** In our case, we would like to allow only pod D to be placed on this node.
So we add a toleration to pod D. Pod D is now tolerant to blue. So when the scheduler tries to place this pod on node 1, it goes through.
Node 1 can now only accept pods that can tolerate the taint blue. Now for example, the scheduler tries to place pod A on node 1, but due to the
taint, it's thrown off and it goes to node 2 and ... .

**Taints are set on nodes and tolerations are set on pods.**

### Taints - node
For example if you would like to dedicate the node to pods in `app: blue`(that have this toleration), then the key-value pair would be:
`app=blue`.

In the command in the slide, `taint-effect` defines what would happen to the pods if they do not tolerate the taint.

NoExecute taint effect means new pods will not be scheduled on the node and existing pods on the node, if any, will be evicted(pod is
killed) if they do not tolerate the taint. These pods may have been scheduled on the node before the taint was applied to the node.
If the pod is killed, the scheduler needs to schedule the pod on another node.

### tolerations - pods
Look at 57-1.

After adding the toleration, the pods created or updated with that definition, they are either not scheduled on the nodes or evicted from the
existing nodes depending on the effect set.

### Taint - NoExecute
Remember: Taints and tolerations are only meant to **restrict** nodes from accepting certain pods. But it does not guarantee that for example in our case,
pod D will always be placed on node 1. **Since there are no taints or restrictions applied on other two nodes, pod D may very well be placed
on any of the other two nodes(although having a toleration). So remember taints and tolerations does not tell the pod to go to a particular node.
Instead, it tells the node to only accept pods with certain tolerations.**

If your requirement is to restrict a pod to certain nodes, it is achieved through another concept called as **node affinity**.

Master node has all the capabilities of hosting a pod, plus it runs all the management software. The scheduler doesn't schedule any pods on the
master node. Why?

Because when the k8s cluster is first set up, a taint is set on the master node automatically that prevents any pods from being scheduled
on this node. You can see this and modify this behavior if required. However, a best practice is to not deploy application workloads
on a master server. To see this taint, run:
```shell
kubectl describe node kubemaster | grep Taint
```

The taint side effect for master node is `NoSchedule`.

When to Use Which (or Both)

- Just want certain pods on certain nodes (but other pods can also go there) -> Use node affinity (or even the simpler nodeSelector).
- Want to reserve / dedicate nodes so only specific workloads can run there -> Use taints + tolerations (repels everyone else) . Note that
nodes with tolerations could end up somewhere else as well.
Almost always combine with node affinity (to actually attract the desired pods). This is the most common pattern for dedicated node groups in production.

## 58 - Practice Test Taints and Tolerations
```shell
k taint nodes node01 spray=mortein:NoSchedule
```

## 59 - Solution Taints and Tolerations Optional
## 60 - Node Selectors
Let's say we have a three node cluster of which two are smaller nodes with lower hardware resources and you have different kinds of workloads
and we would like to dedicate the workloads that require higher resources to the node with more resources as that is the only node that will
not run out of resources in case the job demands extra resources. However, in the current default setup, any pods can go to any nodes which is
not desired. To solve this, we can set a limitation on the pods so that they only run on particular nodes.

There are two ways to do this:
1. using node selectors
2. node affinity

### Node selectors
simple and easier method. For this(for example to limit a pod to run on a larger node), we use nodeSelector in pod definition
file and use a key-value that are in fact labels assigned to nodes. The scheduler uses these labels to match and identify the right node to place
the pods on. To use labels in a `nodeSelector` like this, you must have first labeled your nodes prior to creating this pod.

### Label nodes
To label a node, we use key-value pairs:
```shell
kubectl label nodes <node-name> <label-key>=<label-value>
```
Now that we have labeled the node(s), we can get back to creating the pod, this time with the `nodeSelector` with the same label. When the pod is created,
it's now placed on the nodes with the matching label.

Node selectors served our purpose, but it has limitations. What if our requirement is much more complex? For example, we would like to say sth like:
"Place the pod on a large or medium node" or "place the pod on any nodes that are not small". You cannot achieve this using node selectors.
For this, node affinity and anti-affinity features are used.

## 61 - Node Affinity
The primary purpose of node affinity is to ensure that pods are hosted on particular nodes.

You can't provide advanced expressions like "or" or "not" with node selectors. The node affinity provides advanced features to limit pod placement
on specific nodes.

The pod definition in 61-1 has the same effect as 60-1 but with affinity to place the pod on the large node.

The `operator: In` ensures that the pod will be placed on a node whose label `size` has any value in the list of values specified in `values`.

Q: What if node affinity could not match a node with the given expression? For example what if there are no nodes with the label called size?(look
at the Exist operator slide). Say we had the labels and the pods are scheduled, what if someone changes the label on the node at a future point in time?
Will the pod continue to stay on the node?

A: All of this is specified using that long property which is the type of the node affinity. The type of node affinity defines the behavior
of the scheduler with respect to node affinity and the stages in the lifecycle of the pod. There are currently two types of node affinity available:
- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution
and another type of node affinity is planned: requiredDuringSchedulingRequiredDuringExecution

There are two states in the lifecycle of a pod when considering node affinity:
- during scheduling: The state where a pod does not exist and is created for the first time. We have no doubt that when a pod is first created,
the affinity rules specified are considered to place the pods on the right nodes.
- during execution

What if the nodes with matching labels are not available? For example, we forgot to label the node as large. That is where the type of node affinity
used, comes into play.

**requiredDuringScheduling:** If you select the required type(first one), the scheduler will mandate that the pod be placed on a node with the given
affinity rules. If it cannot find one, the pod will not be scheduled. This type will be used in cases where the placement of the pod is **crucial**.
If a matching node does not exist, the pod will not be scheduled.

**preferredDuringScheduling:** But let's say the pod placement is less important than running the workload itself.
In that case, you could set it to `preferred` and in cases where a matching node is not found, the scheduler will
simply ignore node affinity rules and place the pod on any available node. This is a way of
telling the scheduler: "Hey, try your best to place the pod on matching node. But if you really cannot find one, just place it anywhere."

**Second part(IgnoredDuringExecution):** During execution is the state where a pod has been running and a change is made in
the environment that affects node affinity, such as a change in the label of a node. For example, say an administrator
removed the label we set earlier called **size=large** from the node. Now what would happen to the pods that are running on the node?

The two types of node affinity available today(not that one planned), has this value set to ignored, which means pods will continue to run
and any changes in node affinity, will not impact them once they are scheduled.

The new type: `requiredDuringSchedulingRequiredDuringExecution` only has a difference in the duringExecution phase. A new option requiredDuringExecution
will be introduced which will evict any pods that are running on nodes that do not meet affinity rules. For example, if the label of the node
is removed, the pods that don't match the node affinity rules, will be evicted.

## 62 - Practice Test Node Affinity
## 63 - Solution Node Affinity Optional
## 64 - Taints and Tolerations vs Node Affinity
To solve the problem using taints and toleration, first we apply a taint to the nodes we want, marking them with their colors and then we set
a toleration on the pods we want, to tolerate the respective colors. When the pods are now created, the nodes we tainted, ensure they only
accept the pods with the right toleration. So the green pod ends up on the green node and the blue pod ends up on blue node. However,
**taints and tolerations does not guarantee that the pods will only prefer these nodes.** So the red pod(or could be other pods) ends up
on one of the other nodes that don't have a taint set. This is not desired.

Let's try the same problem with node affinity. With node affinity, we first label the nodes with their respective colors(blue, red and green).
Then we set node selectors on the pods to tie the pods to the respective nodes. As such, the pods end up on the right nodes. However,
**that does not guarantee that other pods are not placed on these nodes.** In this case, there is a chance that one of the other pods(that don't have
a node selector or a node selector that don't match the labeled node) may end up on our nodes. This is not desirable.
As such, a combination of taints and tolerations and node affinity rules can be used **together** to completely dedicate nodes for specific pods.
**We first use taint sand tolerations to prevent other pods from being placed on our nodes and then we use node affinity to prevent our pods
from being placed on other nodes that we don't want.**

## 65 - Resource Requirements and Limits
### Resource limits
It's the k8s scheduler that decides which node a pod goes to.

If there's no sufficient resources available on any of the nodes, k8s holds back scheduling the pod. You will see the pod in a `pending` state
and there would be a reason in the events.

**By default, k8s assumes that a pod or a container within a pod, requires .5 cpu and 256 mebibyte of memory(256Mi) (we have to create a
LimitRange in that namespace first).** This is known as the resource request for a container(minimum amount of cpu and memory requested by the container).
This is assumed that you have set a LimitRange in that namespace.

You can specify these resources on pod or deployment definition files.

What does one count of cpu(`cpu: "1"`) mean?

Note that you can specify any value as low as 0.1 . `0.1` cpu can also be expressed as `100m` where m stands for **milli**.
You can go as low as `1m` but not lower than that.

One count of CPU(cpu: 1) is equivalent to one vCPU. That's one vCPU in aws or one core in GCP or azure or one hyper thread.

256Mi of memory is equal to 268435456. We can use the suffix G for gigabyte. Note the difference between `G`(gigabyte) and `Gi`(gibibyte).

G is gigabyte and it refers to 1000 megabytes whereas GI is gibibyte and refers to 1024 mebibytes.

**In the docker world, a docker container has no limit to the resources it can consume on a node.** Say a container starts with one vCPU on a node.
It can go up and consume as much resource as it requires, suffocating the native processes on the node or other containers, of resources.
However, you can set a limit for the resource usage on these pods.

**By default, k8s sets a limit of one vCPU(of the node) and 512 mebibytes to containers.** To override the limits, use `limits` under `resources` in the
pod definition file. When the pod is created, k8s sets new limits for the container.

Note: requests and limits are set for each container within the pod.

Q: What happens when a pod tries to exceed resources beyond it's specified limit?

A: In case of CPU, k8s throttles the CPU, so that it does not go beyond the specified limit. A container cannot use more CPU resources
than it's limit. However, this is not the case with the memory. A container **can** use more memory resources than it's limit. So if a pod
tries to consume more memory than it's limit constantly, the pod will be terminated.

## 66 - Note on default resource requirements and limits
**In the previous lecture, I said - "When a pod is created the containers are assigned a default CPU request of .5 and memory of 256Mi". 
For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.**

Look at 66-1 and https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

Look at 66-2 and https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/

References:

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource

## 67 - A quick note on editing PODs and Deployments
### Edit a POD
Remember, you CANNOT edit specifications of an existing POD other than the below.
- `spec.containers[*].image`
- `spec.initContainers[*].image`
- `spec.activeDeadlineSeconds`
- `spec.tolerations`

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of
a running pod. But if you really want to, you have 2 options:

1. Run the `kubectl edit pod <pod name>` command. This will open the pod specification in an editor (vi editor). Then edit the 
required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.
![](../img/67-1.png)
![](../img/67-2.png)

A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

```shell
kubectl delete pod webapp
```

Then create a new pod with your changes using the temporary file

```shell
kubectl create -f /tmp/kubectl-edit-ccvrq.yaml
```

2. The second option is to extract the pod definition in YAML format to a file using the command

```shell
kubectl get pod webapp -o yaml > my-new-pod.yaml
```

Then make the changes to the exported file using an editor (vi editor). Save the changes `vi my-new-pod.yaml`

Then delete the existing pod

```shell
kubectl delete pod webapp
```

Then create a new pod with the edited file

```shell
kubectl create -f my-new-pod.yaml
```

### Edit Deployments
With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  
with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a 
property of a POD part of a deployment you may do that simply by running the command

```shell
kubectl edit deployment my-deployment
```

## 68 - Practice Test Resource Requirements and Limits
Practice Test: https://uklabs.kodekloud.com/topic/practice-test-resource-limits-2/

## 69 - Solution Resource Limits Optional

## 70 - DaemonSets
So far, we have deployed pods on different nodes in our cluster. With the help of replica sets and deployments, we made sure multiple copies
of our applications are made available across different worker nodes. DaemonSets are like replicasets, as they help you deploy multiple instances
of pods, but it runs one copy of your pod, on each node in your cluster. Whenever a new node is added  to the cluster, a replica of the pod
is automatically added to that node and when a node is removed, the pod on that node is automatically removed.

**The daemonSet ensures that one copy of the pod is always present in all nodes in the cluster.**

### daemonSets - use case
What are some use cases of daemonSets?

Say you would like to deploy a monitoring agent or log collector on each of your nodes in the cluster, so you can monitor your cluster better,
a daemonSet is perfect for that as it can deploy your monitoring agent in the form of a pod in all the nodes in your cluster.

### daemonSets - use case- kube proxy
One of the worker nodes components that is required on every node in the cluster, is kube-proxy. This component can be deployed as a daemonSet in the cluster.

### daemonSets - use case- networking
Networking solutions like vivenet requires an agent to be deployed on each node in the cluster.

Creating a daemonSet is similar to replicaset creation process. It has nested pod specification under the template section and `selector` to link
the daemonSet to the pods.

### view daemonsets
```shell
kubectl get daemonsets

kubectl describe daemonsets <daemonset name like monitoring-daemon>
```

### How does it work?
How does it schedule pods on each node? and how does it ensure that every node has a pod? If you were asked to schedule a pod on
each node in the cluster, how would you do it?

We could set the nodeName property on the pod to bypass the scheduler and get the pod placed on node directly. So that's one approach.
So on each pod, set the nodeName property in it's specification before it is created and when they are created, they automatically land
on the respective nodes.

This is how it used to be done until k8s v1.12 . From v1.12 onwards, the daemonset uses the default scheduler and node affinity rules to schedule
one pod per node.

## 71 - Practice Test DaemonSets
## 72 - Solution DaemonSets optional
## 73 - Static Pods
The kubelet relies on the kube-apiserver for instructions on what pods to load on it's node which is based on a decision made by the kube-scheduler
which was stored in the etcd data store.

What if there was no kube-apiserver and kube-scheduler and no controllers and no etcd cluster? What if there was no master node at all?
What if there were no other worker nodes(only this node)? What if you're all alone in the sea yourself, not part of any cluster?
Is there anything that the kubelet can do as the captain on the ship? Can it operate as an independent node? If so, who would provide
the instructions required to create those pods?

The kubelet can manage a node independently. There is no k8s cluster, so there are no kube-apiserver or ... . The one thing that the keubelet knows
to do is create pods. But we don't have an api server here to provide pod details. We know that to create a pod, you need details of the pod
in a pod definition file. But how do you provide the pod definition file to the kubelet without a kube-apiserver?

You can configure the kubelet to read the pod definition files from a directory on the server designated to store information about pods.
Place the pod definition files in that directory. The keubelet, periodically checks this directory for files, reads these files and creates pods
on the host. Not only does it create the pod, it can ensure that the pod stays alive. If the application crashes, the kubelet attempts to
restart it. If you make a change to any of the files within that directory(of pod definition files), the kubelet recreates the pod for those
changes to take effect. If you remove a file from this directory, the pod is deleted automatically. So these **pods that are created
by the kubelet on it's own, without the intervention from the API server or rest of the k8s cluster components, are known as static pods.**
Remember you can only create **pods** this way, you can't create replica sets or deployments or services by placing a definition file in the designated
directory. They're all concepts part of the whole k8s architecture that requires other cluster plane components like replication and deployment
controllers and ... .

The kubelet works at pod level and can only understand pods, which is why it's able to create static pods this way.

### Static pods
That designated directory could be any directory on the host and the location of that directory is passed in to the kubelet as an option while
running the service. The option is named `--pod-manifest-path`.

There's another way to configure this. Instead of specifying the option directly in the kubelet.service file, you could provide a path to another
config file using `--config` option and define the directory path with `staticPodPath` property in that file.

Clusters set up by kubeadmin tool, use this approach. If you're inspecting an existing cluster, you should inspect this option to identify the
path to that directory. You will then know where to place the definition file for your static pods.

So first the option `--pod-manifest-path` in the `kubelet.service` file, if it's not there, then look for the `--config` option and identify the file
used as the config file and then within that config file look for the `staticPodPath` option.

Once the static pods are created, view these with:
```shell
docker ps
```
So why not the kubectl command?

Remember we don't have the rest of the k8s cluster yet. The kubectl utility works with the kube-apiserver, since we don't have an apiserver now,
no kubectl utility which is why we're using the `docker` command.

How does it work when the node is part of a cluster? When there is an api server requesting the kubelet to create pods? Can the kubelet create
both kinds of pods at the same time?

The way kubelet works is it can take in requests for creating pods from different inputs. 
1. The first is through the pod definition files from the
static pods folder.
2. The second is through an ACDP API endpoint and that is how the kube-apiserver provides input to kubelet. The kubelet can create both kinds of 
pods(the static pods and the ones from the api server) at the same time. Well in that case, is the api server aware of the static pods created by the
kubelet? Yes it is. If you run `kubectl get pods` on the master node, the static pods will be listed as any other pod. How is that happening?
When the kubelet creates a static pod, if it is a part of a cluster, it also creates a mirror object in the kube-apiserver. What you see from the
kube-apiserver is just a read-only mirror of the pod, you can view the details of the pod but you cannot edit or delete it like the usual pods.
You can only delete them by modifying the files from the nodes manifest folder. Note that the name of the pod is automatically appended with the node name
like `<pod name>-<node name>`.

### Use case
Why would we want to use static pods?

Since static pods are not dependent on the k8s control plane, you can use static pods to deploy the control plane components itself as pods on a node.
Start by installing kubelet on all the master nodes(they will become master once we installed the control plane components as static pods), then create
pod definition files that uses docker images of the various control plane components such as the api server, controller, etc and ... .
Place the defnition files in the designated manifest folder and the keubelet takes care of deploying the control plane components themselves
as pods on the cluster(on master nodes). This way, you don't have to download the binaries, configure services or worry about the services crashing.
If any of these services were to crash, since it's a static pod, it will automatically be restarted by the kubelet.

That;s how the kubeadmin tool sets up a k8s cluster which is why when you list the pods in the `kube-system` namespace, you see the control plane
components as pods in a cluster setup by the kubeadmin tool:
```shell
kubectl get pods -n kube-system
```

### Static pods vs daemon sets
**daemon sets are used to ensure one instance of an application is available on all nodes in the cluster. It has handled by a daemon set controller
through the kube-apiserver. Whereas static pods are created directly by kubelet without any interference from the kube-apiserver or rest of the
k8s control plane components. Static pods can be used to deploy the k8s control plane components itself. Both static pods and pods created
by daemon sets, are ignored by the kube-scheduler. The kube-scheduler has no effect on these pods.**

## 74 - Practice Test Static Pods
## 75 - Solution Static Pods Optional
## 76 - Multiple Schedulers
Deploying multiple schedulers in a k8s cluster.

We've seen how the default scheduler works. It has an algo that distributes pods across nodes evenly as well as takes into consideration various
conditions we specify through taints and tolerations and node affinity and ... .

But what if none of these satisfy your needs? Say you have a specific application that requires it's components to be placed on nodes after
performing some additional checks? So you decide to have your own scheduling algorithm to place pods on nodes. So that you can add your own
custom conditions and checks in it.

You can write your own k8s scheduler program, package it and deploy it as the default scheduler or as an additional scheduler in the k8s cluster.
That way, all of the other apps can go through the default scheduler, however, some specific apps that you may choose, can use your own
custom scheduler. So your k8s cluster can have multiple schedulers at a time.

When creating a pod or a deployment, you can instruct k8s to have it scheduled by a specific scheduler.

When having multiple schedulers, they must have different names so that we can identify them as separate schedulers. The default scheduler name is
`default-scheduler` and this name is configured in a kube-scheduler config file. For other schedulers, we could create a separate config file
and set the `schedulerName`.

### Deploy additional scheduler
Most simple way of deploying an additional scheduler is to download the kube-scheduler binary and run it as a service with a set of options.

Now to deploy an additional scheduler, you may use the same kube-scheduler binary or use one that you might have built for yourself,
which is what you would do if you needed the scheduler to work differently. If you used the same kube-scheduler binary, you probably want to use
some other config, so use `--config` flag. 

This is not how you would deploy a custom scheduler 99% of the time, because with kubeadm deployment, all the control plane components run
as a pod or a deployment within the k8s cluster.
![](../img/76-1.png)

### deploy additional scheduler as a pod
Look at 76-1 code.

The leader elect(`leaderElection`) option is used when you have multiple copies of the scheduler running on different master nodes as a high-availability
setup where you have multiple master nodes with the kube-scheduler process running on them. If multiple copies of the same scheduler are running on
different nodes, only one can be active at a time and that's where the leader elect option helps in choosing a leader who will lead the scheduling activities.

### View schedulers
If you run:
```shell
kubectl get pods --namespace=kubesystem
```
you can see the new custom scheduler(my-custom-scheduler) running, this is if you run it as a pod, and if you run it as a deployment, you will see a slightly
different name, but you'll see the pod there.

### Use custom schedulers
Now once we have deployed that custom scheduler, the next step is to configure a pod or a deployment to use this new scheduler. So how do you use
that custom scheduler?

Look at 76-1 pod-definition.yaml . We added a field called `schedulerName`.

Note: If the scheduler was not configured correctly, then the pod will continue to remain in a `Pending` state and if everything was good,
the pod will be in a `Running` state. If the pod is in Pending state, you can look at the logs using `kubectl describe` command
![](../img/76-2.png)

### view events
How do you know which scheduler picked scheduling a particular pod?

We can use `kubectl get events` to list all the events in the current namespace and look for the `Scheduled` events under `REASON` column.

Note: The `SOURCE` column is source of the event.

![](../img/76-3.png)

### View scheduler logs
Provide the scheduler name(either pod name or deployment name)
```shell
kubectl logs my-custom-scheduler --name-space=kube-system
```

## 77 - Practice Test Multiple Schedulers
## 78 - Solution Practice Test Multiple Schedulers Optional
## 79 - Configuring Scheduler Profiles
When the pods are created, the pods end up in a **scheduling queue** which is where the pods wait to be scheduled. At this stage, pods are sorted
based on the priority defined on the pods using `priorityClassName`.

To set a priority, you must first create a `PriorityClass`.

Pods with higher priority gets to the beginning of the queue to be scheduled first.

Then our pod enters the **filter phase**. That is where the nodes that cannot run the pod(like the ones that do not have sufficient resources to
run the pod) are filtered out.

The next phase is the **scoring phase**. This is where nodes are scored with different weights. From the remaining nodes, the scheduler associates a
score to each node based on the free space that it will have after reserving the cpu required for that pod. So the node with more CPU gets a higher
score. The node with highest cpu gets picked up.

Finally, in the **binding phase**, the pod is finally bound to a node with the highest score.

All of these operations are achieved with certain plugins. For example in the scheduling queue, it's the priority sort plugin that sorts the pods
in an order based on the priority configured on the pods.

In the filtering stage, it's the `NodeResourcesFit` plugin that identifies the nodes that have sufficient resources required by the pods
and filters out the nodes that don't. Other plugin examples in this stage are NodeName plugin that checks if a pod has a `nodeName` mentioned in the
pod's spec and filters out all the nodes that don't match this name. Another one is NodeUnschedulable plugin that filters out nodes that have
`Unschedulable` flag set to true. This is when you're on the drain. So this plugin makes sure that no pods are set on those nodes.

In scoring phase, the `NodeResourcesFit` plugin associates a score to each node based on the resource available on it and after the pod is allocated
to it. So a single plugin can be associated in multiple different phases. Another plugin in this stage is `ImageLocality` that associates
a high score to the nodes that already has the container image used by the pods among the different nodes. Note that in this phase, the plugins 
do not really reject the pod placement on a particular node. For example in case of the image locality node, it ensures that pods are placed
on a node that already has the image but if there are no nodes available, it will anyway place the pod on a node that does not even have the image.
So it's just the scoring that happens at this stage.

In binding phase, we have the `DefaultBinder` plugin that provides the binding mechanism.

The highly extensible nature of k8s makes it possible for us to customize what plugins go where and for us to write our own plugin and plug them in here
and that is achieved with **extension points**. At each stage, there is an extension point to which a plugin can be plugged to.
In the scheduling queue, we have a queue sort extension to which the `PrioritySort` plugin is plugged to and then we have the filter extension,
score extension and bind extension to which each of these plugins that we mentioned are plugged to. Actually there are more extensions.
There are extensions before entering the filter phase called the `preFilter` extension and after the filter phase called `postFilter` and there are
`preScore` before the score extension point and `reserve` for after the score extension point and there's `permit` and `preBind` before the bind
and `postBind` after the binding extension point.

You can get a custom code of your own to run anywhere in these points by just creating a plugin and plugging it into the respective kind of 
extension point.

There are some plugins that span across multiple extension points.

So these were scheduling plugins and extension points.
![](../img/79-1.png)

### scheduler profiles
One way of deploying schedulers is to use separate scheduler binaries that run with a separate scheduler config file associated with each of them.
That's one way of deploying multiple schedulers. The problem here is that since these are separate processes, an additional effort is required
to maintain the separate processes and also more importantly, since they are separate processes, they may run into race conditions while making
scheduling decisions. For example one scheduler may schedule a workload on a node without knowing that there's another 
scheduler scheduling a workload on that same node at the same time.

With the 1.18 release of k8s, a feature to support multiple profiles in a single scheduler was introduced. So now you can configure multiple 
profiles within a single scheduler in the scheduler config file by adding more entries to the list of `profiles` and for each profile,
specify a separate scheduler name. This creates a separate profile for each scheduler which acts as a separate scheduler itself except that now
multiple schedulers are run in the same binary as opposed to creating separate binaries for each scheduler.

Now how do you configure these different scheduler profiles to work differently? Because right now all of them just simply have
different names. So they're gonna work just like the default scheduler, how do you configure them to work differently?

A: Under each scheduler profile, we can configure the plugins the way we want to. For example we can disable some of the plugins for one profile or 
enable some custom plugins.
![](../img/79-2.png)

References:
- https://github.com/kubernetes/enhancements/tree/04d5df19d396511fe41ed0860b0ab9b96f46a2d/keps/sig-scheduling/1451-multi-scheduling-profiles/README.md
- https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/

## 80 - References
- https://github.com/kubernetes/community/blob/master/contributors/devel/sig-scheduling/scheduling_code_hierarchy_overview.md
- https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/
- https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/
- https://stackoverflow.com/questions/28857993/how-does-kubernetes-scheduler-work