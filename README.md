Drawbacks of docker
**Single Host:** Container is scoped to a single host. One containers resource consumption affects other
**No Auto Healing :** No automatic recovery of containers which went down or are facing issues or became dead somehow
**No Auto Scaling and Load Balancing:** No automatic provisioning of more containers and load balancing to meet the demand
**Not Feasible for Enterprise Environments:** No support for enterprise level standards like integration with Firewalls, API Gateways, etc. Hence never used in production
##### Kubernetes solves all these problems

______________________________________________________________

### Architecture of Kubernetes

![[b6e87f74-ef5c-42e2-a1e7-ded890a78030.jpg]]

**Pod :** 
* The smallest deployable unit in Kunernetes. 
* It is an abstraction over containers. Pods provide a environment on top of the container runtime so that theres no need to interact with the underlying container technologies. We only need to interact with the kubernetes layer.
* Esch pod is intended to run one application.
* While pods can run more than one conatiner, it is only intended when we need a side or helper container along with the main one to keep our application running. For Example: A database or caching service container running beside a backend API container 
* Kubernetes comes with its own internal network out of the box. Hence, each pod is assigned its own IP address (internal IP) using which they can communicate with each other (Example: Application pod communicating with Database pod). 
* Pods just like containers are ephemeral, meaning that they can die easily (for various reasons like application crashing or server running out of memory). And when one pod dies Kubernetes spins up a new pod in its place almost immediately.
 But everytime a pod dies and a new pod is created it is assigned a new IP address which is problematic if one of your pod uses IP address to communicate with other. 
 Here is where Service comes in.
**Service :** 
* Services provide a stable IP address for accessing a group of pods. 
* Service provides a fixed IP address for a pod, so that even if the pod goes down anad a new pod comes in its place, the IP address remains (lifecycle of pod and service are not connected).
* Services provide an endpoint for accessing our containers and even provides Load Balancing features across its pods.
* Services can be of internal and external type
**Node :**  A Server or Virtual Machine That runs our pods
**ConfigMap:** Contains all the Environment variables that are made accesible to applications inside containers
