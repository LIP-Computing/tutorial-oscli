# tutorial-oscli
Tutorial Openstack CLI

## Access to machine for the tutorial

The openstack clients pre-installed in this machine, you can access with:

    ssh <USERNAME>@tut.ncg.ingrid.pt

You will find the openstack credentials for your user, so you can set the
environment variables with:

    source os-tut.sh

From here on you should be able to do the rest of the tutorial. Test with:

    openstack project list

## Instantiate a VM

List images:

    openstack image list

List flavors:

    openstack flavor list


## Authors

* Mario David
* Jorge Gomes


