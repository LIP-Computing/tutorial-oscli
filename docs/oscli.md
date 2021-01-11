# Tutorial Openstack CLI

## Access to machine for the tutorial

The openstack clients are pre-installed in this machine, you can access with:

    ssh <USERNAME>@tut.ncg.ingrid.pt

You will find the openstack credentials for your user, so you can set the
environment variables with:

    source os-tut.sh

This will set the following environment variables:

    OS_PROJECT_DOMAIN_NAME=Default
    OS_USER_DOMAIN_NAME=Default
    OS_PROJECT_NAME=tutorial
    OS_USERNAME=<USERNAME>
    OS_PASSWORD=<PASSWORD>
    OS_AUTH_URL=https://stratus.ncg.ingrid.pt:5000/v3
    OS_IDENTITY_API_VERSION=3
    OS_IMAGE_API_VERSION=2

From here on you should be able to do the rest of the tutorial. Test with:

    openstack project list

## Create an ssh keypair

The access to VM instances is done through an ssh key pair, where your public
key is inserted into the VM, when it is instantiated.

To create an ssh keypair, you should issue the following command:

    ssh-keygen

By default it generates an RSA 2048 bit key that is stored in your `$HOME/.ssh/`
directory, you should set a strong `passphrase`

`.ssh/id_rsa` is your ssh private key, and `.ssh/id_rsa.pub` is your ssh public
key.

You can list all keypairs already in openstack:

    openstack keypair list

The next step is to insert your ssh public key in openstack, you should be careful
to choose a keypair name that does not yet exist:

    openstack keypair create --public-key .ssh/id_rsa.pub ${LOGNAME}-key

## Instantiate a VM

In order to instantiate a VM, the following information is needed:

* The image name
* The flavor name
* The network name
* The keypair name

List images and choose one:

    openstack image list

List flavors and choose one:

    openstack flavor list

List all networks:

    openstack network list

Now you can create a server, note the name of the server should not exist, the
list of servers can be checked with:

    openstack server list

    openstack server create --flavor svc1.s --key-name ${LOGNAME}-key --network tutorial_net --image centos7-x86_64-raw ${LOGNAME}-server

The status of the newly created server:

    openstack server show ${LOGNAME}-server

Where the following attributes can be checked:

    | OS-EXT-STS:power_state      | Running    |
    | OS-EXT-STS:vm_state         | active     |

At this point the VM only has a private IP, and is not accessible from a public
network:

    | addresses  | tutorial_net=192.168.1.157 

## Associate a floating public IP with the VM

In the previous section you have listed the available networks, it includes
the public network called `public_net`, you can create a public IP with:

    openstack floating ip create public_net

It will show in particular the attribute:

    | floating_ip_address | 194.210.120.240

Get the server ID with:

    openstack server show ${LOGNAME}-server -f value -c id

Now you can associate the floating public IP with the server:

    openstack server add floating ip <SERVER_ID> 194.210.120.240

Now you can confirm that this floating public IP has been associated to your VM:

    openstack server show ${LOGNAME}-server
    
    | addresses  | tutorial_net=192.168.1.157, 194.210.120.240

Since the base image of the VM is Centos7, the default user is `centos`,
for ubuntu base images the default user is `ubuntu`.

You can now access the VM with ssh:

    ssh centos@194.210.120.240

## Cleanup

To delete the server:

    openstack server delete u60-server



## Authors

* Mario David
* Jorge Gomes
* Alvaro Lopez Garcia
* (Add more contributers here)


