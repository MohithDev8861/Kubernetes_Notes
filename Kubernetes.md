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
