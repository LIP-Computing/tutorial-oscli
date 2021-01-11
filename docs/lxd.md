# HOWTO lxd

## Pre deplyoment steps

We will use a VM with Ubuntu 20.04 instantiated in an Openstack cloud. Enter in the VM and become root,
these steps will update the operating system and boot with the latest kernel:

```bash
ssh ubuntu@<Public_IP>
sudo -s
apt update
apt -y upgrade
apt -y dist-upgrade
shutdown -r now
apt -y autoremove
```

Install lxd: `apt -y install lxd net-tools `

## LXD initialization

Execute the lxd init to initialize LXD, for the purpose of this HOWTO, we will choose almost all default
options:

```
lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Name of the storage backend to use (btrfs, dir, lvm, zfs, ceph) [default=zfs]: 
Create a new ZFS pool? (yes/no) [default=yes]: 
Would you like to use an existing empty block device (e.g. a disk or partition)? (yes/no) [default=no]: 
Size in GB of the new loop device (1GB minimum) [default=19GB]: 50GB
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
Would you like LXD to be available over the network? (yes/no) [default=no]: no
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: yes

config: {}
networks:
- config:
    ipv4.address: auto
    ipv6.address: auto
  description: ""
  name: lxdbr0
  type: ""
storage_pools:
- config:
    size: 50GB
  description: ""
  name: default
  driver: zfs
profiles:
- config: {}
  description: ""
  devices:
    eth0:
      name: eth0
      network: lxdbr0
      type: nic
    root:
      path: /
      pool: default
      type: disk
  name: default
cluster: null
```

Now you can test with: `lxc list`, and some other commands to list and show other information:

```bash
lxc profile list
lxc profile show default
lxc network list
lxc network show lxdbr0
lxc storage list
```

## Instantiating a lxc container

The following command will instantiate a lxc container:

```bash
lxc launch images:ubuntu/focal/amd64 test-01
```

And you can check the status of all containers:

```bash
lxc list
+---------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
|  NAME   |  STATE  |         IPV4          |                     IPV6                      |   TYPE    | SNAPSHOTS |
+---------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| test-01 | RUNNING | 10.124.213.176 (eth0) | fd42:3292:3810:a761:216:3eff:fe15:aaad (eth0) | CONTAINER | 0         |
+---------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
```

## Customizing the container and making a new image

Now that the container is running, you can enter into the container and install packages or
do anyother type of costumization.

```bash
lxc exec test-01 -- bash
```

Now inside the container:

```bash
sudo -s; apt update; apt -y install openssh-server
exit
exit
```

After exiting the container you should stop it: `lxc stop test-01`, now you can publish the new container as an image,
that can be saved and used later:

```bash
lxc publish test-01 --alias ubuntu20.04-sshd
```

Confirm that you have your new image:

```bash
lxc image list

+------------------+--------------+--------+-------------------------------------+--------------+-----------+----------+------------------------------+
|      ALIAS       | FINGERPRINT  | PUBLIC |             DESCRIPTION             | ARCHITECTURE |   TYPE    |   SIZE   |         UPLOAD DATE          |
+------------------+--------------+--------+-------------------------------------+--------------+-----------+----------+------------------------------+
| ubuntu20.04-sshd | 37e5a988a6f3 | no     | Ubuntu focal amd64 (20210111_07:42) | x86_64       | CONTAINER | 163.71MB | Jan 11, 2021 at 4:57pm (UTC) |
+------------------+--------------+--------+-------------------------------------+--------------+-----------+----------+------------------------------+
|                  | e9b02852eba0 | no     | Ubuntu focal amd64 (20210111_07:42) | x86_64       | CONTAINER | 100.21MB | Jan 11, 2021 at 2:54pm (UTC) |
+------------------+--------------+--------+-------------------------------------+--------------+-----------+----------+------------------------------+
```

You can now instantiate a new container from your image:

```bash
lxc launch ubuntu20.04-sshd mycont-02
```

And list your containers:

```bash
lxc list

+-----------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
|   NAME    |  STATE  |         IPV4          |                     IPV6                      |   TYPE    | SNAPSHOTS |
+-----------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| mycont-02 | RUNNING | 10.124.213.101 (eth0) | fd42:3292:3810:a761:216:3eff:fe92:9089 (eth0) | CONTAINER | 0         |
+-----------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| test-01   | STOPPED |                       |                                               | CONTAINER | 0         |
+-----------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
```

You new container `mycont-02` is running, while the old container had been stoped.

To copy the authorized_keys present in the host into the container you can issue:

```bash
lxc file push -r .ssh mycont-02/home/ubuntu/
```

Now `exit` from the root user on the VM host and try to ssh into the container:

```
ubuntu@lxd-test:~$ ssh ubuntu@10.124.213.101
```

From your (lap)desktop you can use a ssh tunnel through the public IP of the host VM to get into the lxc container:

```bash
ssh -J ubuntu@PublicIP>:22 ubuntu@10.124.213.101
```

Now you are inside the container, you can do a `sudo -s` to gain root previleges.
