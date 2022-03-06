Previous: [Generating the Data Encryption Config and Key](06-data-encryption-keys.md)

# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). In this lab you will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `etcd1`, `etcd2`, and `etcd3`. Login to each etcd:

```
ssh etcd1
```

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.




## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [etcd](https://github.com/etcd-io/etcd) GitHub project:

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.5.2/etcd-v3.5.2-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```
{
  tar -xvf etcd-v3.5.2-linux-amd64.tar.gz
  sudo mv etcd-v3.5.2-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
}
```

* alt create etcd user and run as etcd user
```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  groupadd -f -g 1501 etcd
  useradd -c "etcd user" -d /var/lib/etcd -s /bin/false -g etcd -u 1501 etcd
  chown -R etcd:etcd /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
}
```




The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Set the `INTERNAL_IP` env var below to the one belonging to the etcd node you are configuring.

```
export INTERNAL_IP='10.10.1.X'
```

or "automatically":
```
INTERNAL_IP=$(ip addr show eth0 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1)
```


Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
export ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster etcd1=https://10.240.0.31:2380,etcd2=https://10.240.0.32:2380,etcd3=https://10.240.0.33:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
  systemctl status -l etcd.service
}
```

> Remember to run the above commands on each etcd node: `etcd1`, `etcd2`, and `etcd3`.

## Verification


* set endpoint env vars
```
export ETCDCTL_API=3
HOST_1=10.240.0.31
HOST_2=10.240.0.32
HOST_3=10.240.0.33
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379
```

* check health
```
etcdctl --write-out="table" member list
etcdctl --write-out="table" endpoint health
etcdctl --write-out="table" endpoint status
```


* List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

> output

```
```

Next: [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)
