# Section 5 - Application Lifecycle Management

## 89 - Application Lifecycle Management Section Introduction

## 90 - Download Slide Deck
## 91 - Rolling Updates and Rollbacks
### Rollout and versioning in a deployment
When you first create a deployment, it triggers a rollout. A new rollout, creates a new deployment revision, let's call it revision 1.
In the future, when the app is upgraded, meaning when the container version is updated, a new rollout is triggered and a new deployment revision
is created named revision 2. This helps us keep track of the changes made to our deployment and enables us to roll back to a previous version
of deployment if necessary.

To see the status of the rollout:
```shell
kubectl rollout status <deployment name>
```

To see the revisions and history of rollout:
```shell
kubectl rollout history <deployment name>
```

## 92 - Practice Test Rolling Updates and Rollbacks
## 93 - Solution Rolling update Optional
## 94 - Configure Applications
Configuring applications comprises of understanding the following concepts:
- Configuring Command and Arguments on applications 
- Configuring Environment Variables 
- Configuring Secrets

We will see these next

## 95 - Commands
### Command, arguments and entry points in docker
Unlike virtual machines, containers are not meant to host an operating system. Containers are meant to run a specific task or process,
such as to host an instance of a web server or application server or a DB server or simply to carry out some computation or analysis.
Once the task is complete, the container exits. A container only lives as long as the process inside it is alive. If the web server inside
the container is stopped or crashes, the container exits.

Who defines what process is run within the container?
```dockerfile
# defines the program that will be run within the container when it starts
CMD ["nginx"]
```
Another example:
```dockerfile
# ubuntu image
CMD ["bash"]
```
Here, we're using bash as the default command. Now bash is not really a process like a web server or db server. It is a shell that listens 
for inputs from a terminal, if it can't find a terminal, it exits.

So when we're using the ubuntu image and create a container from it, docker launches the bash program(CMD program). By default, docker does not
attach a terminal to a container when it is run and so the bash program doesn't find the terminal and so it exits. Since the process that was
started when the container was created, finished, the container exits as well.

Q: So how do you specify a different command to start the container?

A: One option is to append a command to the `docker run command` and that way, it overrides the default command(CMD) specified within the dockerfile.
For example:
```shell
docker run ubuntu sleep 5
```
Now when the container starts, it runs the `sleep` program, waits for 5 seconds and then exits.

Q: But how do you make that change permanent?

A: Say you want the image to always run the sleep command when it starts. You would then create your **own image**(dockerfile), from the base ubuntu image
and specify a new CMD:
```dockerfile
FROM ubuntu

CMD sleep 5
```

There are different ways of specifying the command:
- enter the command simply as is in a shell form: `CMD command param1` CMD sleep 5
- or in a json array format: `CMD["command", "param1"]` `CMD ["sleep", "5"]`

Remember when you specify the command in a json array format, the first element in the array should be the executable, in this case, the `sleep`
program. Do not specify the command and parameters together like: CMD ["sleep 5"]. Because we don't have an executable named `sleep 5`.
The command and it's parameters should be separate elements in the list.

Q: What if I want to change the number of seconds it sleeps?

A: Currently, it is hard coded to 5 seconds. So one option to change this, is to run the `docker run` command with the command we want to overwrite,
appended to it, like: `docker run <image name> <new command>`, so: `docker run ubuntu-sleeper sleep 10` and so the command that will be run at
startup will be sleep 10. But it doesn't look very good. Because the name of the image(ubuntu-sleeper) in itself implies that the container
will sleep, so we shouldn't have to specify the `sleep` command again. Instead, we would like it to be sth like:
```shell
docker run ubuntu-sleeper 10
```
This is where the entry point instruction(`ENTRYPOINT`) comes in.

The ENTRYPOINT instruction is like the CMD instruction, as in you can specify the program that will be run when the container starts and
whatever you specify on the command line(in previous example 10), will get appended to the ENTRYPOINT.
So when we have ENTRYPOINT ["sleep"] in dockerfile and we run the container by saying: `docker run <image name> 10`, the `10` will get appended
to `["sleep"]`, so we would have sleep 10. But in case of CMD instruction, the command line parameters(in this case 10), will get **replaced entirely**.
Whereas in case of ENTRYPOINT, the command line parameters will get **appended**.

Q: Now in case of ENTRYPOINT, what if we run the ubuntu-sleeper image without appending the number of seconds?

A: Then, the command at startup will be just `sleep` and you get the error: `sleep: missing operand`. 

Q: So how do you configure a default value for the command, if one was not specified in the command line?

A: That's where you would use both ENTRYPOINT as well as the CMD instruction. In this case, the CMD instruction will be appended to the ENTRYPOINT
instruction.

For example: When we have:
```dockerfile
FROM ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```
At startup, the command would be: `sleep 5` if you didn't specify any parameters in `docker run` like: `docker run ubuntu-sleeper`.
But if you did, like: `docker run ubuntu-sleeper 10`, that will overwrite the CMD instruction(entirely just like before), so 5 will be replaced
by whatever command we specified in docker run which in this case is `10`. So the command at startup for second docker run is: sleep 10.
Remember for this to happen, we should always specify the ENTRYPOINT and CMD instructions in a json format.

Q: What if you really really want to modify the ENTRYPOINT during runtime? Not just appending to it?

A: You can overwrite the ENTRYPOINT using --entrypoint option of docker run:
```shell
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```
The command at startup would be: `sleep2.0 10`.

## 96 - Commands and Arguments
### Commands and arguments in a pod definition file(application commands)
```shell
docker run --name ubuntu-sleeper ubuntu-sleeper
```
By default, it sleeps for 5 seconds but you can override it by passing a command line arg:
```shell
docker run --name ubuntu-sleeper ubuntu-sleeper 10
```

Now let's create a pod using this image. Look at 96-1. When the pod is created, it creates a container from the specified image and the container
sleeps for 5 seconds(default) before exiting. Now if you need the container to sleep for 10 seconds, how do you specify the additional arg
in the pod definition file?

A: Anything that is appended to the `docker run` command, will go into the `args` property of pod definition file.

Now our dockerfile has an ENTRYPOINT and a CMD. The ENTRYPOINT is the command that is run at startup and the CMD is the default parameter passed to the
command. **With the `args` option in the pod definition file, we override the CMD instruction in the dockerfile**. But what if you need to override
the ENTRYPOINT of the image? In the docker world, we would run the docker run command with the --entrypoint option set to the new command.
The corresponding option in the pod definition file is `command`.

So command corresponds to ENTRYPOINT instruction in the dockerfile.

- The command field overrides the ENTRYPOINT instruction 
- The args field overrides the command instruction(in dockerfile)

Remember: It's not the command field that overrides the CMD instruction in dockerfile.

## 97 - Practice Test Commands and Arguments
TODO

Practice Test: https://uklabs.kodekloud.com/topic/practice-test-commands-and-arguments-2/

## 98 - Solution Commands and Arguments Optional

## 99 - Configure Environment Variables in Applications
Three ways of setting env vars in k8s:
- plain key value pairs in pod definition file(env property)
- configMaps
- secrets

The difference between these when providing them in pod definition file, is in the two latter cases, we use `valueFrom` instead of `value`.

## 100 - Configuring ConfigMaps in Applications
When you have a lot of pod definition files, it will become difficult to manage the environment data stored within the various files.
We can take this info out of the pod definition file and manage it centrally using configuration maps.

Just like any other k8s object, there are two ways of creating a configMap:
1. the imperative way(without using a configMap definition file): `kubectl create configmap ...`
2. declarative way(using a configMap definition file)

To inject environment variables into a pod, there are multiple ways:
- using configmaps to inject whole data of configmap into pod
- inject a **single** variable of configmap(single env in the slide), ny specifying the `key` field
- inject whole data as files in a volume

## 101 - Practice Test Environment Variables
## 102 - Solution Environment Variables Optional
## 103 - Configure Secrets in Applications
The `--from-literal` option is used to specify the key value pairs in the command itself. Another way to input secret data is through a file by using
`--from-file` option.

While creating a secret with the declarative approach, you must specify the secret values in an encoded format in the `data` section.

But how do you convert the data from plain text to an encoded format?

A: On linux, run:
```shell
echo -n <text you wanna encode> | base64

# to decode
echo -n <text you wanna encode> | base64 --decode
```

### secrets in pods
If you were to mount the secret as a volume in the pod, each variable in the secret is created as a file with the value of the secret as it's content.

### Notes on secrets
- Secrets are not encrypted. Only encoded. Meaning anyone can look up the file that you created for secrets or get the secret object and then decode the
  variables in there. So not check in the secret definition files to github or ... .
- Secrets are not encrypted in etcd. So none of the data in etcd is encrypted by default. So consider enabling encryption at rest.
- once you create the secret in a particular namespace, if anyone with access to creating a pod or deployment in the same namespace, creates a pod or
  deployment and uses that secret, then they're able to see the secret objects mounted on to those pods. So you should consider configuring role-based 
  access control(RBAC) to restrict access.
- you can use external secret provider to not to store secrets in etcd of the cluster and those providers takes care of most of the security.

## 104 - A note about Secrets
Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets
can be considered as not very safe.

The concept of safety of the Secrets is a bit confusing in Kubernetes. The kubernetes documentation page and a lot of blogs out there refer to
secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally 
exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it.

https://kubernetes.io/docs/concepts/configuration/secret

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:
- Not checking-in secret object definition files to source code repositories. 
- Enabling Encryption at Rest for Secrets so they are stored encrypted in ETCD.

https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

Also the way kubernetes handles secrets. Such as:
- A secret is only sent to a node if a pod on that node requires it. 
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage. 
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Read about the protections and risks of using secrets here:

https://kubernetes.io/docs/concepts/configuration/secret/#protections

https://kubernetes.io/docs/concepts/configuration/secret/#risks

https://kubernetes.io/docs/concepts/configuration/secret/#risks

Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets,
HashiCorp Vault. I hope to make a lecture on these in the future.

## 105 - Practice Test Secrets
## 106 - Solution Secrets Optional
## 107 - Demo Encrypting Secret Data at Rest
TODO

## 108 - Scale Applications
We have already discussed about scaling applications in the Deployments and Rolling updates and Rollback sections.

## 109 - Multi Container PODs
At times, you may need two services to work together such as a web server and a logging service. You need one log agent instance per 
web server instance paired together. You don't want to merge and load the code of the two services, as each of them target different functionalities
and you would still like them to be developed and deployed separately. We want these two to scale up and down **together** and that is why you can have
multi-container pods which share the same lifecycle which means they are created together and destroyed together. They share the same network space,
which means they can refer to each other as localhost, and they have access to the same storage volumes. This way you don't have to
establish volume sharing or services between the pods to enable communication between them.

## 110 - Practice Test Multi Container PODs
## 111 - Solution MultiContainer Pods Optional
## 112 - Multicontainer PODs Design Patterns
There are 3 common patterns, when it comes to designing multi-container PODs. The first and what we just saw with the logging service example is
known as a side car pattern. The others are the adapter and the ambassador pattern.

But these fall under the CKAD curriculum and are not required for the CKA exam. So we will be discuss these in more detail in the CKAD course.

![](../img/112-1.png)

## 113 - InitContainers
In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle.
For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are 
expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running.
If any of them fails, the POD restarts.

But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that
will be used by the main web application. That is a task that will be run only one time when the pod is first created. Or a process that waits
for an external service or database to be up before the actual application starts. That's where initContainers comes in.

An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section,  like this:

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the
real container hosting the application starts.

You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is
run one at a time in sequential order.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.

Read more about initContainers here. And try out the upcoming practice test.

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

## 114 - Practice Test Init Containers
## 115 - Solution Init Containers Optional
## 116 - Self Healing Applications
Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in
ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of 
the application are running at all times.

Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness 
and Readiness Probes. However these are not required for the CKA exam and as such they are not covered here. These are topics for the
Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.

## 117 - If you like it Share it