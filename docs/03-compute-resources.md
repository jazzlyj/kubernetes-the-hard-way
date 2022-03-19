Previous: [Installing the Client Tools](02-client-tools.md)

# Pre-Requisites
* 1 "smart" configurable switch
* 1 MAAS node, which is: 
  * the DHCP server
  * PXE boot/imaging server  
  * DNS server
  * the router for the 10.32.0/24, 10.10.1.0/24, and 10.200.0.0/16 networks  



# Provisioning Compute Resources
* 3 Controllers
* 3 Etcd 
* 2 Load Balancers
* 3 Workers
ALL nodes are Ubuntu 20.04.4 LTS (GNU/Linux 5.13.0-35-generic x86_64)



## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

Critical Assumptions - All this setup lives behind some firewall or router that is NAT-ing a Private to Private or Public to Private Network

* There are 4 networks that need to be setup. The network #'s can be any as long as there are 4.
  * 10.10.1.0/24  - "External"/Public Net  (yes its a private address but for this tutorial it will serve as our PUBLIC network) 
  * 10.240.0.0/24  - "Internal"/Prvate Net  (the primary network that all the metal and vms will be built on)
  * 10.32.0.0/24 - Kubernetes Service network
  * 10.200.0.0/16  - Cluster CIDR Net
    * 10.200.$i.0/24 - POD instance i network
   
![image](https://user-images.githubusercontent.com/13760175/159127653-cbd7def2-1e8f-41dc-9e61-6cea2bf3d9ec.png)



A [subnet](https://cloud.google.com/compute/docs/vpc/#vpc_networks_and_subnets) must be provisioned with an IP address range large enough to assign a private IP address to each node in the Kubernetes cluster.

In this tutorial 1 host is used that serves as the router/gateway for the 3 of the networks. It could serve as the gateway for the fourth. 

 The same node is the MAAS controller node which is used to commission/deploy all metal vm nodes  


> The `10.240.0.0/24` IP address range can host up to 254 compute instances.

### Firewall Rules

No firewalls. Firewall is disabled in this tutorial.




> An [external load balancer](https://cloud.google.com/compute/docs/load-balancing/network/) will be used to expose the Kubernetes API Servers to remote clients.


### Kubernetes Public IP Address

Allocate a static IP address that will be attached to the external load balancer fronting the Kubernetes API Servers:

```
10.10.1.40
```


## Compute Instances

The compute instances in this lab will be provisioned using [Ubuntu Server](https://www.ubuntu.com/server) 20.04, which has good support for the [containerd container runtime](https://github.com/containerd/containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Kubernetes Controllers

Use 3 metal nodes or create 3 VMs which will host the Kubernetes control plane:

```
```

### Etcd hosts
* 3 hosts


### Kubernetes Workers

Each worker instance requires a pod subnet allocation from the Kubernetes cluster CIDR range. The pod subnet allocation will be used to configure container networking in a later exercise. 

> The Kubernetes cluster CIDR range is defined by the Controller Manager's `--cluster-cidr` flag. In this tutorial the cluster CIDR range will be set to `10.200.0.0/16`, which supports 254 subnets.

Three Kubernetes worker nodes:

```
```

### Verification



## Configuring SSH Access

SSH will be used to configure the controller and worker instances. When connecting to compute instances for the first time SSH keys will be generated for you and stored in the project or instance metadata as described in the [connecting to instances](https://cloud.google.com/compute/docs/instances/connecting-to-instance) documentation.

Test SSH access to the `controller1` compute instances:

```
ssh controller1  
```


Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
