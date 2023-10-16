# Section 3 - Scheduling

## 49 - Scheduling Section Introduction

## 50 - Download Presentation Deck for this section

## 51 - Manual Scheduling
### How scheduling works?
Every pod has a field called `nodeName` that by default is not set, k8s adds it automatically. The scheduler goes through all the pods
and looks for those that do not have this property set. Those are the candidates for scheduling. It then identifies the right node for the pod by
running the scheduling algorithm. Once identified, it schedules the pod on the node by setting the nodeName property to the name of the node by creating a
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
call this taint, blue. By default, pods have no tolerations which means unless specified otherwise, none of the pods can tolerate any taint.
So in this case, none of the pods can be placed on node 1, as none of them can tolerate the taint blue. This solves half of our requirements.
No unwanted pods are going to be placed on this node(node 1). The other half is to enable certain pods to be placed on this node.
For this, we must specify which pods are tolerant to this particular taint. In our case, we would like to allow only pod D to be placed on this node.
So we add a toleration to pod D. Pod D is now tolerant to blue. So when the scheduler tries to place this pod on node 1, it goes through.
Node 1 can now only accept pods that can tolerate the taint blue. Now for example, the scheduler tries to place pod A on node 1, but due to the
taint, it's thrown off and it goes to node 2 and ... .

Taints are set on nodes and tolerations are set on pods.

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
Remember: Taints and tolerations are only meant to restrict nodes from accepting certain pods. But it does not guarantee that for example in our case,
pod D will always be placed on node 1. Since there are no taints or restrictions applied on other two nodes, pod D may very well be placed
on any of the other two nodes(although having a toleration). So remember taints and tolerations does not tell the pod to go to a particular node.
Instead, it tells the node to only accept pods with certain tolerations.

if your requirement is to restrict a pod to certain nodes, it is achieved through another concept called as **node affinity**.

Master node has all the capabilities of hosting a pod, plus it runs all the management software. The scheduler doesn't schedule any pods on the
master node. Why?

Because when the k8s cluster is first set up, a taint is set on the master node automatically that prevents any pods from being scheduled
on this node. You can see this and modify this behavior if required. However, a best practice is to not deploy application workloads
on a master server. To see this taint, run:
```shell
kubectl describe node kubemaster | grep Taint
```

The taint side effect for master node is `NoSchedule`.

## 58 - Practice Test Taints and Tolerations
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
it's no placed on the nodes with the matching label.

Node selectors served our purpose, but it has limitations. What if our requirement is much more complex? For example, we would like to say sth like:
"Place the pod on a large or medium node" or "place the pod on any nodes that are not small". You cannot achieve this using node selectors.
For this, node affinity and anti-affinity features are used.

## 61 - Node Affinity
## 62 - Practice Test Node Affinity
## 63 - Solution Node Affinity Optional
## 64 - Taints and Tolerations vs Node Affinity
## 65 - Resource Requirements and Limits
## 66 - Note on default resource requirements and limits
## 67 - A quick note on editing PODs and Deployments
## 68 - Practice Test Resource Requirements and Limits
## 69 - Solution Resource Limits Optional
## 70 - DaemonSets
## 71 - Practice Test DaemonSets
## 72 - Solution DaemonSets optional
## 73 - Static Pods
## 74 - Practice Test Static Pods
## 75 - Solution Static Pods Optional
## 76 - Multiple Schedulers
## 77 - Practice Test Multiple Schedulers
## 78 - Solution Practice Test Multiple Schedulers Optional
## 79 - Configuring Scheduler Profiles
## 80 - References