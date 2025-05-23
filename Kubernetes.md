Drawbacks of docker
**Single Host:** Container is scoped to a single host. One containers resource consumption affects other
**No Auto Healing :** No automatic recovery of containers which went down or are facing issues or became dead somehow
**No Auto Scaling and Load Balancing:** No automatic provisioning of more containers and load balancing to meet the demand
**Not Feasible for Enterprise Environments:** No support for enterprise level standards like integration with Firewalls, API Gateways, etc. Hence never used in production
##### Kubernetes solves all these problems

______________________________________________________________

### Main components of Kubernetes

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

**ConfigMap:** Contains all the external configuration like database url's, etc that are made accesible to applications inside containers.

**Secret:** Just lke ConfigMap, but is used to store more secret info like credentials, passwords, API keys, etc

**Volumes:** Attach External storage to a pod to persist storage. Kubernetes doesnt explicitly manage storage.

**Replica Set:** A ReplicaSet is a process that runs multiple instances of a Pod and keeps the specified number of Pods constant. Its purpose is to maintain the specified number of Pod instances running in a cluster at any given time to prevent users from losing access to their application when a Pod fails or is inaccessible. ReplicaSet helps bring up a new instance of a Pod when the existing one fails, scale it up when the running instances are not up to the specified number, and scale down or delete Pods if another instance with the same label is created.

**Deployment:** Deployment is an abstraction over the pods (like a blueprint for creating pods). In reality you would never work with pods directly, we create deployments and these deployments manage the lifecycle of the pods, and the nuber of replicas we need and the policies for scaling up and scaling down the replicas.

**StateFul set:** Deployments are for stateless applications. Sometimes we need our application to maintain the state. Like for example if we have pods running database containers, we need to maintain the same state of the database across all the replicas to avoid data inconsistencies. this problem is solved by StateFul sets. They are just like deployments, but can maintain the state across pods. Deploying a StateFul sets is complicated so it is not preferred in most cases.

### Architecture of Kubernetes

**Node:**  A Server or Virtual Machine That holds and runs our pods. There are 2 types of nodes mainly, Master Node and Worker Node.

#### **Worker Node:**  

Worker nodes do the actual work, they contain and run our application. Each Node has multiple pods.
Each Node has 3 main processes installed by default, which are needed to run and manage pods.

* **Container Runtime:** A runtime to run containers, like Docker.
* **kubelet:** A process of Kubernetes that is responsible for taking the configuration then creating and running a pod with container inside it( Also responsible for allocating resources to the pods inside the node). Kubelet interacts with the Node and container runtime. 
* **kube-proxy:** A process that forwards the requests from services to pods.

#### **Master Node:** 

Master Node Manages all the worker nodes and the incoming traffic
Each master node has 4 processes primarily

**API Server:**  The Only Entry point into to the cluster (Acts as the gate keeper). Any entity should communincate with API server in order to interact with the cluster. Authenticates and validates the request, after which the request is allowed to move forward.

**Scheduler:** The process which schedules the creation, removal and placement of pods. Has an intelligent way of checking for the nodes with most available resources and scheduling the pod creation there. In reality, schedules just decides on which node a new pod should be created, after which it sends the request to kublet of the particular node which creates a new pod.

**Controller Manager:** Detects the cluster state changes and maintains the desired cluster state. Example: if a pod dies then controller manager detects it and notifies the scheduler for creation of a new pod in order to maintain the desired state.

**etcd:** Akey value store that stores all the info about the cluster. Every change in the cluster state (pod scheduling, creation and removal). All the info needed for the working of other 3 processes is stored in etcd (like amount of resources available, changes in cluster state, cluster health, etc). Application data is not stored in etcd.

In general a cluster can have multiple master nodes, where API servers are load balanced and etcd forms a distributed data storage. 

Basically, 

Deployment manages Replica Sets
I_ Replica Set manages pods 
   |_ which are an abstraction of container

## Kubectl

The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.

Some commonly used commands

```
kubectl get [component name]  # get info and status of the respective component in a cluster
kubectl get pods
```

## Kubernetes YAML configuration file

We can use YAML files to describe the desired state of our cluster.
Every configuration file in kubernetes has 3 parts
1) Metadata:  metadata of the specific component, Like name, label, etc
2) Specs: Specification describing the desired state of the component (example no of replicas)
3) Status: The current status of the component (not present initially but k8s adds it once we apply deployment). Status is stored in **etcd** and is continuously updated by kubernetes. Kubernetes compares this status with the specs in the .yaml file and if there are any discrepancies found then it will try to fix them. This is the backbone of kubernetes **auto healing** feature. 
Example of deployment.yaml file
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

**Template:** Template is the configuration of a pod. It has its own metadata and specs. This gives all specifications about the container that we intend to run like the image, port, etc.

_________________________________________________________________________
Secret Configuration file

All secret values must be base64 encoded

```
apiVersion: v1
kind: Secret
metadata:
    name: mongodb-secret
type: Opaque
data:
    mongo-root-username: dXNlcm5hbWU=
    mongo-root-password: cGFzc3dvcmQ=

```

Referencing Secret values 
![[{E1EB8056-20F4-4436-BB81-62FA3D5478C5}.png]]


________________________________________________________________________

### Namespaces

Kubernetes namespaces are the way of dividing the cluster resources between multiple users. They comes with a mechanism of creating logical isolated environments within the same kubernetes cluster. Each namespace has its own set of policies, resources, and access controls making them ideal for the environments such as development, staging and production. provides better resource management, security and maintaining an organized structure within large kubernetes deployments.

The following are the reasons to use kubernetes Namespaces:
- **Resource Isolation:** They provide logical separation of resources, ensuring that different applications or teams do not interfere with each other within the same cluster.
- **Access Control:** Namespaces enable fine-grained access controls, allowing administrators to define permissions and policies specific to each namespace.
- **Environment Segregation**: They facilitate the creation of separate environments (e.g., development, testing, production) within a single cluster, improving organization and management.
- **Efficient Resource Management:** Namespaces allow for better resource allocation and quota management, preventing any single application from consuming excessive resources at the expense of others.

When the kubernetes cluster is setup, at that time 4 kubernetes namespaces are created, each with some specific purpose. Those are as follows:

- **kube-system:** System processes like Master and kubectl processes are deployed in this namespace; thus, it is advised not to create or modify the namespace.
- **kube-public:** This namespace contains publicly accessible data like a configMap containing cluster information.
- **kube-node-lease:** This namespace is the heartbeat of nodes. Each node has its associated lease object. It determines the availability of a node.
- **default:** This is the namespace that you use to create your resources by default.

Although whatever resources you create will be created in the default namespace but you can also create your own new namespace and create resources there.
**Note:** Avoid creating namespaces with the prefix Kube-, since it is reserved for Kubernetes system namespaces and you should not try to modify them.
##### Points to remember:
**ConfigMap and Secret cannot be shared:** Each namespace must have its own ConfigMap and Secret file even though the variables refer the same value or component.

![[{562F389C-4B1A-4717-8522-71F9D830478A}.png]]

**Services can be shared across NameSpaces:** You can access services from other namespaces, by attaching the namespace name after the url of the service.

![[{7DD43B6B-E6D5-414C-B6C7-0F852B77E4A3}.png]]

**Some components cant be isolated with a namespace:** Components like volumes and nodes are accessible throughout the cluster irrespective of namespaces

![[{C0C133BB-1DD9-46A1-BD67-2CF4962AF72F}.png]]

### Ingress 

Ingress is a Kubernetes API object that is used to expose HTTP and HTTPS routes from outside the Kubernetes cluster to services inside the cluster. It provides a single entry point into a cluster hence making it simpler to manage applications and troubleshoot routing issues. The official definition of Ingress says ****"Ingress is an API object that manages external access to the services in a Cluster"****.

![[{8ED4515F-DE89-4BC7-AA6A-9DD6E9DE2119}.png]]

![[{D5F6F95D-3FD0-4694-B7FA-2BE96D4204BD}.png]]

More about ingress:  https://www.geeksforgeeks.org/what-is-kubernetes-ingress/
