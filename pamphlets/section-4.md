# Section 4 - Logging Monitoring

## 81 - Logging and Monitoring Section Introduction

## 82 - Download Presentation Deck
Please note that some slides are animated so content may not have exported correctly. Kindly use the slides as a reference for commands.

## 83 - Monitor Cluster Components
Q: What would you like to monitor?

A: I'd like to know node-level metrics such as
- number of nodes in the cluster
- how many of them are healthy
- performance metrics such as CPU, memory, network and disk utilization

As well as pod-level metrics, such as:
- number of pods
- performance metrics of each pod, cpu, memory

So we need a solution that will monitor these metrics, store them and provide analytics around this data. For now, k8s does not come
with a full-featured built-in monitoring solution. Some open source solutions are: metrics server, prometheus, elastic stack and 
proprietary solutions like datadog and dynatrace

### Heapster vs metrics server
Heapster was one of the original projects that enabled monitoring and analysis feature for k8s, but it's now deprecated and a slimmed-down version
was formed known as metrics-server.

You can have one metrics-server per k8s cluster. Note that metrics-server is only an in-memory monitoring solution, so you can't see historical performance
data. For that you must rely on one of the advanced monitoring solutions.

Q: How are the metrics generated for the pods on these nodes?

A: K8S runs an agent on each node known as the kubelet which responsible for receiving instructions from the k8s-api master server and running pods
on the nodes. The kubelet also contains a sub-component known as **cAdvisor or container advisor**. cAdvisor is responsible for
retrieving performance metrics from pods and exposing them through the kubelet API to make the metrics available for metrics server.

### Metrics server - getting started
If you're using minikube for your local cluster, run:
```shell
minikube addons enable metrics-server
```

For all other envs, deploy the metrics server by cloning the metrics server deployment files and then deploying the required components using kubectl create
command:
```shell
git clone https://github.com/kubernetes-incubator/metrics-serve

# This command deploys a set of pods, services and roles to enable metrics server to poll for performance metrics from the nodes in the cluster.
kubectl create -f deploy/1.8+/
```

### View
Cluster performance can be viewed by running:
```shell
kubectl top node

kubectl top pod # to view performance metrics of pods
```

## 84 - Practice Test Monitoring
## 85 - Solution Monitor Cluster Components Optional
## 86 - Managing Application Logs
Logging in docker
### logs - docker
If we were to run the docker container in the background in a detached mode using `-d`, we wouldn't see the logs. If we want to view the logs,
we can use the `docker logs -f <container id>`. The `-f` option helps us see the live log trail.

### Logs - k8s
Once the pod is running, we can view the logs using `kubectl logs -f <pod name>`. -f option is for streaming the logs live just like the one in docker
command.

Now these logs are specific to the container running inside the pod. But we know k8s pods cna have multiple containers in them.
Now since we have multiple containers in the pod, you must specify the exact container name in the kubectl logs command.
An example of a pod with two containers is in `86-1`.

This is the simple logging functionality implemented within k8s.

## 87 - Practice Test Monitor Application Logs
## 88 - Solution Logging Optional