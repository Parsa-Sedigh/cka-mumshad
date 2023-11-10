# Section 9 - Networking

## 196 - Networking Section Introduction
Required basics:
- setting and checking IP addresses on any given system
- knowledge on gateways and routes
- knowledge about DNS and configuring DNS servers

## 197 - Download Presentation Deck
Please note that some slides are animated so content may not have exported correctly. Kindly use the slides as a reference for commands.

## 198 - Prerequisite Switching Routing
### Linux networking basics
### Switching
What is a network?

We have two computers, A and B, laptops, desktops, VMs on the cloud, whatever, how does system A reach B? We connect them to a switch and
the switch creates a network containing the two systems. To connect them to a switch, we need an interface on each host. Physical or virtual,
depending on the host. To see the interfaces for the host, we use `ip link` command. In this case, we look at the interface named eth0, that
we will be using to connect to the switch. Let's assume it's a network with the address 192.168.1.0 .
We then assign the systems with IP addresses on the same network, for this, we use the `ip addr` command(do it for both systems):
```shell
ip addr add 192.168.1.10/24 dev eth0

ip addr add 192.168.1.11/24 dev eth0
```

Once the links are up and the ip addresses are assigned, the computers can now communicate with each other through the switch:
```shell
# on the other system run:
ping 192.168.1.11
```

The switch can only enable communication within a network. Which means it can receive packets from a host on the network and deliver it to other systems within
the same network.

Say we have another network containing systems C and D at address 192.168.2.0 . The systems have ip address 192.168.2.10 and 192.168.2.11 .
How does a system in one network reach a system in the other? How does the system B with the IP 192.168.1.11 reach system C with the IP 2.10 on the
other network?

That's where a router comes in. A router helps connect two networks together. It is an intelligent device, so think of it as another server with many
network ports. Since it connects to the two separate networks, it gets two IPs assigned, one on each network. In the first network,
we assign it an IP of 192.168.1.1 and in the second, we assign it an IP 192.168.2.1 .

Now we have a router connected to the two networks that can enable communication between them.

Q: When system B tries to send a packet to system C, how does it know where the router is on the network to send the packet through?

A: The router is just another device on the network. There could be many other such devices. That's where we configure the systems with a gateway
or a route.

### Gateway
If the network was a room, the gateway is a door to the outside world. To the other networks or to the internet. The systems need to know
where that door is to go through that. To see the existing routing configuration on a system, run the `route` command. It displays the kernel's routing table
and within that, for now, there are no routing configurations. So in this condition, your system B will not be able to reach system C.
It can only reach other systems within the same network in the range 192.168.1.0 .

To configure a gateway on system B, to reach the systems on network 192.168.2.0, run the `ip route add` command and specify that you can reach
the 192.168.2.0 network through the door or gateway at 192.168.1.1 :
```shell
# on system B run:
ip route add 192.168.2.0/24 via 192.168.1.1
```
By running the `route` command again shows that we have a route added to reach the 192.168.2.0 series network through the router(specified in `Gateway` col).

Now remember: This has to be configured on all the systems, for example if the system C is to send a packet to system B(the reverse action of what we did
previously), then you need to add a route on system C's routing table to access the network at 1.0(the network where system B lives) through the router
configured with the IP address 2.1 (192.168.2.1):
```shell
# on system c run(look at the slide of gateway as well):
ip route add 192.168.1.0/24 via 192.168.2.1

route
```

Now suppose these systems need access to the internet. Say they need access to google at 172.217.194.0 network on the internet. So you connect
the router to the internet and then add a new route in your routing table(for systems want to access
that network on internet) to route all traffic to the network 172.217.194.0 through your router:
```shell
ip route add 172.217.194.0/24 via 192.168.2.1
```

### Default gateway
But there are so many different sites on different networks on the internet. Instead of adding a routing table entry for the same router's IP address for
each of those networks, you can simply say for any network that you don't know a route to, use this router as the default gateway. This way any
request to any network outside of your existing network, goes to this particular router:
```shell
ip route add default via 192.168.2.1
```
With this, a Destination with value of `default` with the specified gateway will be added, so that any destination that is not in the entries,
will go to the specified gateway(192.168.2.1).

So in a simple setup like this, all you need is a single routing table entry with the default gateway set to the router's IP address.

**Instead of the word `default`, you could also say 0.0.0.0 which means any IP destination.**

So both of these lines mean the same thing:

Destination     Gateway

default         192.168.2.1

0.0.0.0         192.168.2.1

A 0.0.0.0 entry in the `Gateway` field indicates that you don't need a gateway. For example in this case, for system C to access any devices in the
192.168.2.0 network, it doesn't need a gateway because it is in it's own network. 

But say you have multiple routers in your network, one for the internet, another for internal private network, then you will need to have two separate
entries for each network. One entry for the internal private network and another entry with the default gateway for all **other** networks including
public networks. So if you're having issues reaching internet from your systems, this routing table and the default gateway configuration is a good
place to start.

![](../img/198-1.png)
In img above, we say for ip destination of range 192.168.1.0(the left network), go to 192.168.2.2 gateway which is the router for internal private network
and for any other ip destination(indicated with `default`), go to gateway with ip 192.168.2.1 which means public internet or any other network.

Q: How to set up a linux host as a router?

A: We have 3 hosts: A and B are connected to a network 192.168.1.0 (we read it 192.168.1 because that last one is the range) and B and C
to another on 192.168.2.0 (read 192.168.2). So host B is connected to both the networks using two interfaces eth0 and eth1.
`A` has ip 192.168.1.5, C has 192.168.2.5 and B has an IP on both the networks 1.6 and 2.6 .
How do we get A to talk to C?

If we try to ping 2.5 from A by saying: `ping 192.168.2.5`, it would say network is unreachable. This is because host A has no idea how to
reach to a network at 192.168.2.0 . We need to tell host A that the door or gateway to network 192.168.2, is through host B and we do that
by adding a routing table entry for host A.

We add a route to access network 192.168.2 via the gateway 192.168.1.6:
```shell
# on host A
ip route add 192.168.2.0/24 via 192.168.1.6
```

If the packets where to get through to host C, host C will have to send back responses to host A. When host C tries to reach host A at 192.168.1 network,
it will face the same issue: network is unreachable, so we need to let host C know that it can reach host A through host B which is acting as a router.
So add a similar entry to host C's routing table. This time we say to reach network 192.168.1.0, talk to host B at 192.168.2.6:
```shell
# on host B
ip route add 192.168.1.0/24 via 192.168.2.6
```

When we try to pint host B from host A, we no longer get the network unreachable error message. That means our routing entries are right. But we still
don't get any response back. Why?

By default, in linux, packets are not forwarded from one interface to the next. For example, packets received on eth0 on host B, are not forwarded
to elsewhere through eth1. This is for security reasons. For example, if you had eth0 connected to your private network and eth1 to a public network,
we don't want anyone from the public network to easily send messages to the private network, unless you explicitly allow that. But in this case since we
know that both are private networks and it is safe to enable communication between them, we can allow host B to forward packets from one
network to the other.

Whether a host can forward packets between interfaces is governed by a setting in `/proc/sys/net/ipv4/ip_forward`. By default the value in this file
is set to zero, meaning no forward.
```shell
cat /proc/sys/net/ipv4/ip_forward
```
Set this to 1 and you should see pings go through:
```shell
echo 1 > /proc/sys/net/ipv4/ip_forward
```
Now if you do ping 192.168.2.5, we would get the replies back!

Now remember simply setting this value, does not persist the changes across reboots. For that, you must modify the same value in the
/etc/sysctl.conf file. In that file, set `net.ipv4.ip_forward = 1`.

### Take aways
- `ip link` is to list and modify interfaces on the host. 
- `ip addr` is to see the IP addresses assigned to those interfaces 
- `ip addr add` is used to set ip addresses on the interfaces, for example: ip addr add `192.168.1.10/24 dev eth0`

Now remember changes made using these commands are only valid till a restart. If you want to persist these changes, you must set them
in the etc/network/interfaces file.

- `ip route or` `route` command is used to view the routing table. 
- `ip route add` command is to add entries into the routing table, like: `ip route add 192.168.1.0/24 via 192.168.2.1`
- `cat /proc/sys/net/ipv4/ip_forward` the command to check if IP forwarding is enabled on a host. If you're working with a host configured as a router

## 199 - Prerequisite DNS


200 - Prerequisite CoreDNS
201 - Prerequisite Network Namespaces
202 - FAQ
203 - Prerequisite Docker Networking
204 - Prerequisite CNI
205 - Cluster Networking
206 - Important Note about CNI and CKA Exam
207 - Practice Test Explore Kubernetes Environment
208 - Solution Explore Environment optional
209 - Pod Networking
210 - CNI in kubernetes
211 - CNI weave
212 - Practice Test Explore CNI
213 - Solution Explore CNI optional
214 - Practice Test Deploy Network Solution
215 - Solution Deploy Network Solution optional
216 - IP Address Management Weave
217 - Practice Test Networking Weave
218 - Solution Networking Weave optional
219 - Service Networking
220 - Practice Test Service Networking
221 - Solution Service Networking optional
222 - DNS in kubernetes
223 - CoreDNS in Kubernetes
224 - Practice Test Explore DNS
225 - Solution Explore DNS optional
## 226 - Ingress
The app is available at my-online-store.com . You build the application into a docker image and deploy it on the k8s cluster as a pod in a deployment.
Your app needs a DB, so you deploy a MySQL DB as a pod and create a service of type clusterIP called mysql-service to make it accessible to our application.
Your app is now working.

To make the app accessible to the outside world, you create another service called wear-service and it's of type NodePort and it makes your app available
on a high port on the nodes in the cluster. In this example, a port 38080 is allocated for the service. The users can now access your app using
the url: http://<ip of any of your nodes>:38080 . So users can access the app.

Whenever traffic increases, we increase the number of replicas of the pod to handle the additional traffic and the service takes care of splitting traffic
between the pods.

However, if you have deployed a production-grade app before, you know that there are many more things involved in addition to simply splitting the
traffic between the pods. For example, we don't want the users to have to type in the IP address everytime. So you configure your DNS server to point
to the IP of the nodes. Your users can now access your app using the url my-online-store.com:38080 .

Now your don't want your users to have to remember port number either. However, node port services can only allocate high numbered ports which are
greater than 30,000. For this, your bring in an additional layer between the DNS server and your cluster like a proxy server that proxies requests
on port 80 to port 38080 on your nodes. You then point your DNS to this server and users can now access your app by simply visiting my-online-store.com .

Now this would work if your app is hosted on-prem in your data center. Let's take a step back and see what you could do if you were on a public cloud env
like gcp?

In that case, instead of creating a service of type node port for your wear app, you could set it to type LoadBalancer. When you do that,
k8s would still do everything that it has to do for a nodePort which is to provision a high port for the service(38080), but in addition to that, k8s
also sends a req to gcp to provision a network load balancer for the service. On receiving the req, GCP will then automatically deploy a load balancer
configured to route traffic to the service ports on all nodes and return it's info to k8s. The load balancer has an external IP that can be provided
to users to access the app. In this case, we set the DNS to point to this IP of load balancer and users now access the app using the url
my-online-store.com .

Your company's business grows and you now have new services for your customers. For example, a video streaming service. We want the users to access
the video streaming service by going to my-online-store.com/watch. You want to make your old app accessible at my-online-store.com/wear.

The new video streaming app is developed as a completely different app as it has nothing to do with the existing app. However, to share the
cluster's resources, you deploy the new app as a separate deployment within the same cluster. You create a service called video-service of type
LoadBalancer, k8s provision port of 38282 for this service and also provisions a network load balancer on the cloud(gcp-load-balancer-2).
The new load balancer has a new IP. Remember you must pay for each of these LBs and having many such load balancers can inversely affect your cloud bill.

Q: So how do you direct traffic between each of these LBs based on the URL that the user types in?

A: You need yet another proxy or load balancer that can redirect traffic based on URLs to the different services. Everytime you introduce a new
service, you have to reconfigure the load balancer(the one that redirect traffic based on the url user requested, so that the req hits the right
lower load balancer layer) and finally you also need to enable SSL for your applications, so your users can access your app using https.
Where do you configure that?

It can be done at different levels, either at the application level itself or at the LB level or at the proxy server level, but which one?

Well you don't want your developers to implement it in their applications, as they would do it in different ways and it's an additional
burden for them to write additional code to handle that. You want it to be configured in one place with minimal maintenance.
That's a lot of different configuration and all of these becomes difficult to manage when your application scales.
It requires involving different individuals and different teams, you need to configure your firewall rules for each new service and
it's expensive as well as for each service a new cloud native LB needs to be provisioned. Wouldn't it be nice if you could manage all of that
within the k8s cluster and have all that configuration as just another k8s definition file?

That's where ingress comes in. Ingress helps your users access your app using a single externally accessible URL that you can configure to 
route traffic to different services within your cluster based on the URL path. At the same time, implement SSL security as well.

Think of ingress as a layer 7 load balancer built into the k8s cluster that can be configured using native k8s primitives.
Now remember, even with ingress, you still need to expose it to make it accessible outside the cluster. So you still have to either
publish it as a nodePort:
![](../img/226-1.png)

or with a cloud-native load balancer:
![](../img/226-2.png)

But that is just one-time configuration. Going forward, you're going to perform all your load balancing of SSL and 
url-based routing configurations on the ingress controller.

Q: Without ingress, how would you do all of this?

A: I would use a reverse proxy or a load balancing solution like nginx or HAproxy or traefik. I would deploy them on my k8s cluster and configure them
to route traffic to other services. The configuration involves defining URL routes, configuring SSL certificates and ... . 
Ingress is implemented by k8s in kind of the same way.

You first deploy a supported solution like nginx or ... and then specify a set of rules to configure ingress. The solution you deploy is
called as an **ingress controller** and the set of rules you configure are called as **ingress resources**. Ingress resources are created using
definition files like the ones we've been using to create pods and ... .

Note that a k8s cluster does not come with an ingress controller by default. So if you simply create ingress resources and expect them to work,
they won't.

As I mentioned, you don't have an ingress controller on k8s by default, so you must deploy one. What do you deploy?

There are a number of solutions available for ingress, a few of them: GCE which is googles layer 7 http load balancer, nginx, contour,
HAproxy, traefik and istio. Out of these, GCE and nginx are currently being supported and maintained by the k8s project and we will use nginx in this lecture.

These ingress controllers are not just another load balance or nginx server. The load balancer components are just a part of it. Ingress controllers
have additional intelligence built into them to monitor k8s cluster for new definitions or ingress resources and configure the nginx(in the case
of using nginx as ingress controller) server accordingly.

An nginx controller is deployed as just another deployment in k8s. So we start with a deployment definition file. Look at 226-1 code.
The image of pods used in that definition file, is a special built of nginx, built specifically to be used as an ingress controller in k8s.
So it has it's own set of requirements. Within the image, the nginx program is stored at location /nginx-ingress-controller. So you must pass that
as the command to start the nginx controller service. So we used the `args` section in that definition file.

We know nginx needs a set of config options like path to store the logs, the keep-alive threshold, ssl settings, session timeout and ... .
In order to decouple these configuration data from the nginx controller image, you must create a configMap object and pass those options in.
Now remember the configMap object doesn't need any entries at this point. A blank object will do, but creating one makes it easy for you to modify
a config setting in the future.

We must also pass in two env vars using `env` that carry the pod's name and namespace it is deployed to. The nginx service requires these to read
the configuration data from within the pod.

Then we need a service to expose the ingress controller to the external world. So create a service of type nodePort with the 
`name: nginx-ingress` selector to link the service to the deployment.

As mentioned, the ingress controllers have additional intelligence built into them to monitor the k8s cluster for ingress resources and configure
the underlying nginx server when sth is changed. But for the ingress controller to do this, it requires a `ServiceAccount` with the right set of permissions.
For that, create a ServiceAccount with the correct roles and roleBindings.

Summary:

With a deployment of the nginx ingress image, a service(nodePort) to expose it, a configMap to feed nginx configuration data and a service account
with the right permissions to access all of these objects, we should be ready with an ingress controller in it's simplest form.

### Ingress resource
An ingress resource is a set of rules and configurations applied on the ingress controller.

With rules, we can route traffic to different applications based on the URL of the request. Or you can route user based on the domain name itself.
For example, if the user visits wear.my-online-store.com, then route the user to the wear app, else, route the user to the video app.

An ingress resource is created with a k8s definition file. Look at ingress-wear.yaml (ingress resource for wear app).

**As with any other k8s object, we have apiVersion, kind, metadata and spec**.

The apiVersion for Ingress is `extensions/v1beta1` as of the time of recoding of the course, but it's expected to change with newer releases of k8s.

Note: The traffic is OFC router to the application services and not the pods directly. The `backend` section defines where the traffic will be routed to.
So if it's a single backend, then you don't really have any rules. You can simply specify the service name and port of the backend service.

### Ingress resource - rules
You use rules when you want to route traffic based on different conditions.

You can handle traffic based on what domain was requested by the user, for example a rule for my-online-store.com, second rule for wear.my-online-store.com ,
watch.my-online-store.com and forth rule to handle everything else.

Note: You can get different domain names to reach your cluster by adding multiple DNS entries all pointing to the same ingress controller service
on your k8s cluster.

Now within each rule, you can handle different paths. For example within rule 1, you can handle the /wear path to route that traffic to the cloths application
and a /watch path to route traffic to the video streaming app and a third path that routes anything other than first two paths, to a 404 not found page.

So we have rules at the top for each host or domain name and within each rule, you have different paths to route traffic based on the URL.

If you look at the output of `k describe ingress ingress-wear-watch`, in the output you see `Default backend: default-http-backend:80 (<none>)`.
What is this?

if a user tries to access a url that does not match any of these rules, then the user is directed to the service specified as the default backend.
In this case it happens to be a service named default-http-backend. So you must remember to deploy as such a service.

If user requests a url tht doesn't exist, we want to show him a nice 404 message. We can do this by configuring a default backend service.

To route traffic based on domain names, use `host` option in each rule, like: `- host: wear.my-online-store.com` to route traffic to appropriate backend.
So all traffic for each domain name will be routed to the specified list of backends irrespective of the url path of the req. You can have
multiple path specifications in each of these to handle different url paths.

Comparison: Splitting traffic by url path had just one rule and we split the traffic with two paths. To split traffic by host name,
we used two rules and one path specification in each rule.

227 - Article Ingress
228 - Practice Test Ingress 1
229 - Solution Ingress Networking 1 optional
230 - Ingress Annotations and rewritetarget
231 - Practice Test Ingress 2
232 - Solution Ingress Networking 2 optional