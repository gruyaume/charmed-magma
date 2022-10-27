# 1. Getting Started

## Requirements

This tutorial has been written to work on Ubuntu:

- **:material-ubuntu: Operating system**: Ubuntu 20.04
- **:octicons-cpu-16: CPU**: 10 cores
- **:fontawesome-solid-memory: Memory**: 16 GB
- **:material-ethernet: Storage**: 100 GB

## Installing dependencies

First, open a terminal window and install [Juju](https://juju.is/), [Multipass](https://multipass.run/), [MicroK8s](https://microk8s.io/) and [Kubectl](https://kubernetes.io/docs/reference/kubectl/) using snap.

```bash
sudo snap install juju
sudo snap install multipass
sudo snap install kubectl --classic
sudo snap install microk8s --channel=1.22/stable --classic
```

## Creating the environment

On our Ubuntu machine, we will create two virtual machines and a Kubernetes cluster.

### Virtual machines environment

First, let's make sure multipass is configured to use the `lxd` driver:

```bash
multipass set local.driver=lxd
```

Now, create 2 virtual machines using multipass. The first one is for the access gateway component:

```bash
multipass launch --name magma-access-gateway --mem=4G --disk=40G --cpus=2 --network=lxdbr0 20.04
```

The second one is for the srsRAN radio component

```bash
multipass launch --name srsran --mem=4G --disk=20G --cpus=2 20.04
```

List the created virtual machines and their addresses:

```bash
multipass list
```

The result should look like:
```bash
ubuntu@host:~$ multipass list
Name                    State             IPv4             Image
magma-access-gateway    Running           10.24.157.76     Ubuntu 20.04 LTS
                                          10.209.93.17
srsran                  Running           10.24.157.37     Ubuntu 20.04 LTS
```

### Kubernetes environment

Add the ubuntu user to the microk8s group:

```bash
sudo usermod -a -G microk8s ubuntu
```

Modify the ownership of the `~/.kube` directory:
```bash
sudo chown -f -R ubuntu ~/.kube
```

Log in to the `microk8s` group:

```bash
newgrp microk8s
```

Enable the following MicroK8s add-ons:

```bash
microk8s enable dns storage
```

Enable the metallb add-on with a range of any 5 IP addresses:

```bash
microk8s enable metallb:10.0.1.1-10.0.1.5
```

Output the kubernetes configuration to a file:

```bash
microk8s config > config
```

Move the configuration file to your `~/.kube/` directory:

```bash
mv config ~/.kube/
```

Validate that you can run kubectl commands:

```bash
kubectl get nodes
```

The result should look like:

```bash
ubuntu@host:~$ kubectl get nodes
NAME                 STATUS   ROLES    AGE    VERSION
magma-orchestrator   Ready    <none>   104s   v1.25.2
```
