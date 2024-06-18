# Prerequisites

In this lab you will review the machine requirements necessary to follow this tutorial.

## Provisioning LXD Virtual Machines

This tutorial requires four (4) virtual machines running Debian 12 (bookworm). The following table list the four machines and their CPU, memory, and storage requirements.

| Name    | Description            | CPU | RAM   | Storage |
|---------|------------------------|-----|-------|---------|
| jumpbox | Administration host    | 1   | 512MB | 10GB    |
| server  | Kubernetes server      | 1   | 2GB   | 20GB    |
| node-0  | Kubernetes worker node | 1   | 2GB   | 20GB    |
| node-1  | Kubernetes worker node | 1   | 2GB   | 20GB    |

There are many ways to meet these requirements, but this tutorial is designed to be used with LXD VMs running locally, on an Ubuntu machine that has around 12 GB RAM at the minimum (the VMs will use 6.5GB). You can create the necessary VMs as follows:

If LXD is not yet set up on your machine, run the following command, and follow the prompts in the way that best suits your local environment (the defaults should be ok):
```bash
lxd init
```

Create a LXD profile to make provisioning the jumpbox with the correct allocations easy:
```bash
lxc profile create kthw-jumpbox
lxc profile edit kthw-jumpbox
```

Delete the profile entries after 'Name:' (Name cannot be edited, but it's already correct) and insert the following text:
```text
description: Kubernetes The Hard Way jumpbox profile
config:
  cloud-init.network-config: |
    version: 1
    config:
      - type: physical
        name: enp5s0
        subnets:
          - type: dhcp
            ipv4: true
  limits.cpu: "1"
  limits.memory: 512MiB
devices:
  eth0:
    name: eth0
    network: lxdbr0
    type: nic
  root:
    path: /
    pool: default
    size: 10GiB
    type: disk
```

Now, create the kthw-node profile the other three machines will use:
```bash
lxc profile create kthw-jumpbox
lxc profile edit kthw-jumpbox
```

Delete the profile entries after 'Name:'and add this:
```text
description: Kubernetes The Hard Way node profile
config:
  cloud-init.network-config: |
    version: 1
    config:
      - type: physical
        name: enp5s0
        subnets:
          - type: dhcp
            ipv4: true
  limits.cpu: "1"
  limits.memory: 2GiB
devices:
  eth0:
    name: eth0
    network: lxdbr0
    type: nic
  root:
    path: /
    pool: default
    size: 20GiB
    type: disk
```

Now, create the VMs:
```bash
lxc launch images:debian/12 --profile kthw-jumpbox kthw-jumpbox --vm
lxc launch images:debian/12 --profile kthw-node kthw-server --vm
lxc launch images:debian/12 --profile kthw-node kthw-node-0 --vm
lxc launch images:debian/12 --profile kthw-node kthw-node-1 --vm
```

Once you have all four machine provisioned, verify they have the correct system requirements and resource allocations by running `lxc list` with the following parameters:

```bash 
lxc list kthw- -c nsP,limits.cpu,limits.memory,devices:root.size
```

After running the above command you should see the following output:

```text
+----------------+---------+--------------+------------+---------------+-----------+
|      NAME      |  STATE  |   PROFILES   | LIMITS CPU | LIMITS MEMORY | ROOT SIZE |
+----------------+---------+--------------+------------+---------------+-----------+
| kthw-jumpbox   | RUNNING | kthw-jumpbox | 1          | 512MiB        | 10GiB     |
+----------------+---------+--------------+------------+---------------+-----------+
| kthw-node-0    | RUNNING | kthw-node    | 1          | 2GiB          | 20GiB     |
+----------------+---------+--------------+------------+---------------+-----------+
| kthw-node-1    | RUNNING | kthw-node    | 1          | 2GiB          | 20GiB     |
+----------------+---------+--------------+------------+---------------+-----------+
| kthw-server    | RUNNING | kthw-node    | 1          | 2GiB          | 20GiB     |
+----------------+---------+--------------+------------+---------------+-----------+
```


Next: [setting-up-the-jumpbox](02-jumpbox.md)
