# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

## The Routing Table

In this section you will gather the information required to create routes in the `kubernetes-the-hard-way` VPC network.

Print the internal IP address and Pod CIDR range for each worker instance:

```

```

> output

```
10.240.0.61 10.200.1.0/24
10.240.0.62 10.200.2.0/24
10.240.0.63 10.200.3.0/24
```

## Routes

Create network routes for each worker instance:

```
```

List the routes VPC network:

```
```

> output

```
```

Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
