# tutorial-oscli
Tutorial Openstack CLI

## Access to machine for the tutorial

The openstack clients pre-installed in this machine, you can access with:

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

List images:

    openstack image list

List flavors:

    openstack flavor list


## Authors

* Mario David
* Jorge Gomes
* (Add more contributers here)


