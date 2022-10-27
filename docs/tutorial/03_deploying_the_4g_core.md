# 3. Deploying the 4G core

## Bootstrapping a new Juju controller

First, list all the network interfaces that magma-access-gateway has:

```bash
multipass exec magma-access-gateway -- ip -br address show scope global
```

The result should look like so:

```bash
ubuntu@host:~$ multipass exec magma-access-gateway -- ip -br address show scope global
enp5s0           UP             10.24.157.76/24 fd42:c027:d54:8986:5054:ff:feb4:d6b2/64 
enp6s0           UP             10.209.93.17/24 fd42:74e4:d36e:d16d:5054:ff:feae:17d6/64
```

Note the two interfaces, **enp5s0** and **enp6s0** in my example. Yours may vary.

From the host, bootstrap a **new** juju controller:

```bash
juju bootstrap localhost virtual-machine
```
!!! info
    
    We have now bootstrapped two Juju controllers, one to manage our Kubernetes environment and one to 
    manage our virtual machines environment. From the host, you can always list the controllers by 
    running `juju controllers` and you can switch from one to the other using 
    `juju switch <controller name>`.

Copy the `magma-access-gateway` private ssh key to your host's home directory and change its permission
for it to be usable by juju:

```bash
sudo cp /var/snap/multipass/common/data/multipassd/ssh-keys/id_rsa .
sudo chmod 600 id_rsa
sudo chown $USER id_rsa
```

From the `virtual-machine` juju controller, add the access-gateway virtual machine to Juju:

```bash
juju add-machine ssh:ubuntu@10.24.157.76 --private-key=id_rsa
```

!!! warning

    Make sur you use the actual IP address of the magma-access-gateway machine!

Validate that the machine is available:

```bash
juju machines
```

The output should look like:

```bash
ubuntu@host:~$ juju machines
Machine  State    Address       Inst id              Series  AZ  Message
0        started  10.24.157.76  manual:10.24.157.76  focal       Manually provisioned machine
```

## Deploying Magma Access Gateway

Deploy Access Gateway with the interfaces listed earlier (enp5s0 and enp6s0):

```bash
juju deploy magma-access-gateway-operator --config sgi=enp5s0 --config s1=enp6s0 --channel=beta --to 0
```

You can see the deployment's status by running `juju status`. The deployment is completed when 
the application is in the `Active-Idle` state. This step can take a lot of time, expect at least 
10-15 minutes.

```bash
ubuntu@host:~$ juju status
Model    Controller        Cloud/Region         Version  SLA          Timestamp
default  virtual-machines  localhost/localhost  2.9.35   unsupported  15:46:28-04:00

App                            Version  Status  Scale  Charm                          Channel  Rev  Exposed  Message
magma-access-gateway-operator           active      1  magma-access-gateway-operator  beta      14  no       

Unit                              Workload  Agent  Machine  Public address  Ports  Message
magma-access-gateway-operator/0*  active    idle   0        10.24.157.241          

Machine  State    Address        Inst id               Series  AZ  Message
0        started  10.24.157.241  manual:10.24.157.241  focal       Manually provisioned machine
```
