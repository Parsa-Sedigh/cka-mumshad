# Section 8 - Storage
## 179 - Storage Section Introduction

## 180 - Introduction to Docker Storage

## 181 - Storage in Docker
We're gonna learn about Docker storage drivers and file systems. We'll see where and how docker stores data and how it manages filesystems
of the containers.

### File system
How docker stores data on the local file system?

When you install docker on a system, it creates this folder structure at /var/lib/docker. It has multiple folders under it called aufs, containers,
image, volumes and ... . This is where docker stores all it's data by default. Data means files related to images and containers running on the
docker host.

Q: So how exactly does docker store the files of an image and a container?

A: To understand that, we need to understand docker's layered architecture. When docker builds images, it builds them in a layered architecture.
Each line of instruction in the Dockerfile creates a new layer in the docker image with just the changes from the previous layer.

Since each layer only stores the changes from the previous layer, it is reflected in the size as well. In img, the ubuntu image layer is 120MB,
the apt packages is 306MB and the next layers are very small.

When we wanna build an image from the right Dockerfile, since the first three layers of both the applications are the same, docker is not going to build
the first three layers(assuming we've build an image from left dockerfile in the past). Instead, it reuses the same three layers it built for
the first application from the cache and only creates the last two layers(new source and new ENTRYPOINT)

This way docker builds images faster and efficiently saves disk space.

![](../img/181-1.png)

This is also applicable if you were to update your application code. Whenever you update your application code, such as app.py in this case,
docker simply reuses all the previous layers from cache and quickly rebuilds the application image by updating the latest source code(COPY command in 
this case).

All of these layers are created when you run the `docker build` command to form th final docker image. So all of these are docker **image** layers.
Once the build is complete, you can't modify the contents of these layers. You can only modify them by initiating a new build.

When you run a container using `docker run` based on the image, docker creates a new writable layer on top of the image layer and it's named **container layer**.

The writable layer is used to store data created by the container such as log files written by the applications, any temporary files generated
by the container just any file modified by the user on that container. The life of this layer is only as long as the container is alive. When the container is
destroyed, this layer and all the changes stored in it are also destroyed.

Remember that the same image layers are shared by all containers created using this image.

![](../img/181-2.png)

### Copy-on-write

If I were to log into the newly created container and for example: create a new file called temp.txt , it will create that file in the container layer
which is read and write.

Since we baked our code into the image, the code is part of the image layer and therefore is readonly.

After running a container, what if I wish to modify the source code(application code) to test a change?

Remember, the same image layer is shared between multiple containers created from this image. So does it mean that I cannot modify this file inside
the container?

No, I can still modify this file but before I save the modified file, docker automatically creates a copy of the file in the read write layer and I will
then be modifying a different version of the file in the read write layer. All future modifications will be done on this copy of the file in the
read write layer. This is called **copy-on-write** mechanism. The image layers being read-only just means that the files in these layer will not be
modified in the image itself, so the image will remain the same all the time, until you rebuild the image using `docker build`.

What happens when we get rid of the container?

A: All of the data that was stored in the container layer, also gets deleted.
![](../img/181-3.png)

### Volumes
So what if we wish to persist the files in the container layer? For example we were working with a DB and we want to preserve the data created
by the container.

For this, we can add a persistent volume to the container. To do this, first create a volume using `docker volume create` command.
When you run `docker volume create data_volume` command, it creates a folder called data_volume under `var/lib/docker/volumes`. Then,
when we run the container with: docker run, we can mount this volume inside the docker container's read-write layer, using -v option:
`docker run -v <volume name like data_volume>:<location inside the container like /var/lib/mysql> <image name>`.
This will create a new container and mount the volume we created into /var/lib/mysql folder inside the container. So all data written by the
database, is in fact stored on the volume created on the docker host.

Note: /var/lib/mysql is the default location where mysql stores data, so we don't have to overwrite the place mysql stores data since we specified
the default location.

Now even if the container is destroyed, the data is still stored.

Q: Now what if we didn't run the `docker volume create` command to create the volume before `docker run` command like 
docker run -v data_volume2:/var/lib/mysql mysql?

A: Docker will **automatically** create a volume named `data_volume2` and mount it to the container.

This is called **volume mounting** as we're mounting a volume created by docker under /var/lib/docker/volumes folder.

Q: But what if we had our data already at another location for example let's say we have some external storage on the docker host at /data
and we want to store DB data on that volume and not in the default /var/lib/docker/volumes folder.

A: In this case, we would run the container using:
```shell
docker run -v <full path to the folder we wanna mount>:<folder inside container> <image name>
# for example:
docker run -v /data/mysql:/var/lib/mysql mysql
```
This mounts the folder(/data/mysql in this case) to the container(/var/lib/mysql in this case) is called **bind mounting**.

So there are two types of mounts:
- volume mount: mounts the volume from the `/var/lib/docker/volumes` folder to the container(left side of img)
- bind mount: mounts a folder from **any location on the docker host** to the container(right side of img)

Using `-v` is old style. The new way is to use `--mount` option, like:
```shell
# source is a location on the host and target is a location on the container
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

### Storage drivers
Q: Who is responsible for doing all of these operations? Maintaining the layered architecture, creating a writable layer, moving files
across layers(image layer to container layer) to enable copy and write and ... .

A: It's the storage drivers. So docker uses storage drivers to enable layered architecture.
- AUFS
- ZFS
- BTRFS
- device mapper
- overlay
- overlay2

![](../img/181-4.png)

The selection of the storage driver depends on the underlying OS being used. For example, with ubuntu, the default storage driver is AUFS, whereas
this storage driver is not available on other operating systems like fedora or centos. In that case, device mapper may be a better option.

Docker will choose the best storage driver available automatically, based on the OS.

The different storage drivers also provide different performance and stability characteristics.

## 182 - Volume Driver Plugins in Docker
### Docker volumes
Storage drivers help manage storage on images and containers.

Note: Volumes are not handled by storage drivers. Volumes are handled by volume driver plugins. The default volume driver plugin is **local**.
The local volume plugin helps create a volume on the docker host and store it's data under the var/lib/docker/volumes folder. There are other
volume driver plugins that allow you to create a volume on third party solutions, like azure file storage, convoy, digitalocean block storage,
flocker, google compute persistent disks, gce-docker, glusterFS, netapp, rexray, portworx, vmware vSphere storage.
Some of these volume drivers support different storage providers. For example rexray storage driver can be used to provision storage on
aws EBS, s3, EMC storage arrays like isilon and scaleIO or google persistent disk or openstack cinder.

### volume drivers
When you run a docker container, you can choose to use a specific volume driver such as rexray EBS to provision a volume from amazon EBS.
This will create a container and attach a volume from the aws cloud. When the container exits, your data is still persisted in the cloud.

![](../img/182-1.png)

## 183 - Container Storage Interface CSI
In the past, k8s used docker alone as the container runtime engine and all the code to work with docker, was embedded within the k8s source code.
With other container runtimes coming in like rkt and cri-o, it was important to open up and extend support to work with different container runtimes and
not be dependent on k8s source code and that's how container runtime interface came to be.

container runtime interface is a standard that defines how an orchestration solution like k8s, would communicate with container runtimes like docker.
So in the future, if any new container runtime is developed, they can follow the CRI standards and that new container runtime would work
with k8s without having to work with k8s team of developers or touch the k8s source code.

To extend support for different networking solutions, the container networking interface(CNI) was introduced.
Now any new networking vendors could develop their plugin based on the CNI standards and make it work with k8s.

We also have container storage interface to support multiple storage solutions to write our own drivers for our storage to work with k8s.

![](../img/183-1.png)

Note: CSI is not a k8s specific standard. It's meant to be universal standard and if implemented, allows any container orchestration tool to work
with any storage vendor with a supported plugin.

CSI defines a set of RPCs that will be called by the container orchestrator and these must be implemented by the storage drivers.
For example CSI says: When a pod is created and requires a volume, the container orchestrator(in this case k8s), should call the create volume RPC
and pass a set of details such as volume name. The storage driver should implement this RPC and handle that request and provision a new volume
on the storage array and return the result of the operation.

![](../img/183-2.png)

## 184 - Download Slide Deck
Please note that some slides are animated so content may not have exported correctly. Kindly use the slides as a reference for commands.

## 185 - Volumes
Docker containers are meant to be transient in nature, which means they are meant to last only for a short period of time.

To persist data processed by the containers, we attach a volume to the containers when they are created. The data processed by the container
is now placed in this volume. Even if the container is deleted, the data generated or processed by it, remains.

So how does that work in the k8s?

Just as in docker, the pods created in k8s are transient in nature. When a pod is created to process data, and then deleted,
the data processed by it, gets deleted as well(just like containers). To fix this, we attach a volume to the pod.
The data generated by the pod is now stored in the volume.

### Volumes & mounts
In 185-1 we create a simple pod that generates a random number between 1 and 100 and writes that to a file at /opt/numbe r.out .

To retain this number generated by the pod, we create a volume and the volume needs a storage. Once the volume is created, to access
it from a container, we mount the volume to a directory inside the container. We use the `volumeMount` field in each container to mount the
volume to the directory specified by mountPath within container.

The random number will now be written to /opt inside the container.

Volume storage options:

Using hostPath option to specify a folder on the host as storage space for the volume, works fine on a single node, but it is not recommended
for use in a multi node cluster. Because the pods would use the /data folder on all the nodes and expect all of them to be the same and have the
same data. Since they are on different servers, they're in fact not the same unless you configure some kind of
**external replicated cluster storage solution**. K8S supports several types of different storage solutions like NFS, CephFS and ... or public
cloud solution like aws, ebs and ... .

For example, to configure an aws elastic block store volume as the storage option for the volume, we replace hostPath of the volume with
`awsElasticBlockStore` and in there, specify volumeID and fsType. The volume storage will now be on aws ebs.

## 186 - Persistent Volumes
## 187 - Persistent Volume Claims
## 188 - Using PVCs in PODs
## 189 - Practice Test Persistent Volumes and Persistent Volume Claims
## 190 - Solution Persistent Volumes and Persistent Volume Claims
## 191 - Application Configuration