Deploys a group of Virtual Machines exposed to a public IP via a Load Balancer
==============================================================================

This Terraform module deploys a Virtual Machines Scale Set in Azure and opens the specified ports on the loadbalancer for external access and returns the id of the VM scale set deployed.

This module requires a network and loadbalancer to be provider separately. You can provision them with the "Azure/network/azurerm" and "Azure/loadbanacer/azurerm" modules.


Module Input Variables 
----------------------

- `resource_group_name` - The name of the resource group in which the resources will be created.
- `location` - The Azure location where the resources will be created.
- `vm_size` - The initial size of the virtual machine that will be used in the VM Scale Set.
- `vmscaleset_name` - The name of the VM scale set that will be created in Azure.
- `computer_name_prefix` - The prefix of the name of the VM that will be deployed as part of the VM scale set.
- `managed_disk_type` - The The type of storage to use for the managed disk for the VM Scale Set. Allowable values are `Standard_LRS` (default) or `Premium_LRS`. 
- `admin_username` - The name of the administrator to access the machines part of the virtual machine scale set. 
- `admin_password` - The password of the administrator account. The password must comply with the complexity requirements for Azure virtual machines.
- `ssh_key` - The path on the local machine of the ssh public key in the case of a Linux deployment.  
- `nb_instance` - The number of instances that will be initially deployed in the virtual machine scale set.
- `vnet_subnet_id` - The subnet id of the virtual network on which the vm scale set will be connected.
- `network_profile` - The name of the network profile that will be used in the VM scale set.
- `protocol` - A map representing the protocols and ports to open on the load balancer in front of the virtual machine scale set.

- `vm_os_simple`- This variable allows to use a simple name to reference Linux or Windows operating systems. When used, you can ommit the `vm_os_publisher`, `vm_os_offer` and `vm_os_sku`. The supported values are: "UbuntuServer", "WindowsServer", "RHEL", "openSUSE-Leap", "CentOS", "Debian", "CoreOS" and "SLES".

- `vm_os_id` - The ID of the image that you want to deploy if you are using a custom image. When used, you can ommit the `vm_os_publisher`, `vm_os_offer` and `vm_os_sku`. 

- `vm_os_publisher` - The name of the publisher of the image that you want to deploy, for example "Canonical" if you are not using the `vm_os_simple` or `vm_os_id` variables. 
- `vm_os_offer` - The name of the offer of the image that you want to deploy, for example "UbuntuServer" if you are not using the `vm_os_simple` or `vm_os_id` variables. 
- `vm_os_sku` - The sku of the image that you want to deploy, for example "14.04.2-LTS" if you are not using the `vm_os_simple` or `vm_os_id` variables. 
- `vm_os_version` - The version of the image that you want to deploy, default is "latest". 

- `load_balancer_backend_address_pool_ids` - The id of the backend address pools of the loadbalancer to which the VM scale set is attached.
- `lb_port` - Protocols to be used for the load balancer rules [frontend_port, protocol, backend_port]. Set to blank to disable.
- `tags` - A map of the tags to use on the resources that are deployed with this module.

Usage
-----

Using the `vm_os_simple`: 

```hcl 
provider "azurerm" {
  version = "~> 0.1"
}

variable "resource_group_name" {
    default = "terraform-test"
}

module "network" {
    source = "Azure/network/azurerm"
    location = "westus"
    resource_group_name = "${var.resource_group_name}"
  }

module "loadbalancer" {
  source = "Azure/loadbalancer/azurerm"
  resource_group_name = "${var.resource_group_name}"
  location = "westus"
  prefix = "terraform-test"
}

module "computegroup" { 
    source              = "Azure/computegroup/azurerm"
    resource_group_name = "${var.resource_group_name}"
    location            = "westus"
    vm_size             = "Standard_A0"
    admin_username      = "azureuser"
    admin_password      = "ComplexPassword"
    ssh_key             = "~/.ssh/id_rsa.pub"
    nb_instance         = 2
    vm_os_simple        = "UbuntuServer"
    vnet_subnet_id      = "${module.network.vnet_subnets[0]}"
    load_balancer_backend_address_pool_ids = "${module.loadbalancer.azurerm_lb_backend_address_pool_id}"
    lb_port             = { 
                            http = ["80", "Tcp", "80"]
                            https = ["443", "Tcp", "443"]
                          }
    tags                = {
                            environment = "dev"
                            costcenter  = "it"
                          }
}

output "vmss_id"{
  value = "${module.computegroup.vmss_id}"
}

```

Using the `vm_os_publisher`, `vm_os_offer` and `vm_os_sku` 

```hcl 

provider "azurerm" {
  version = "~> 0.1"
}

variable "resource_group_name" {
    default = "terraform-test"
}

module "network" {
    source = "Azure/network/azurerm"
    location = "westus"
    resource_group_name = "${var.resource_group_name}"
  }

module "loadbalancer" {
  source = "Azure/loadbanacer/azurerm"
  resource_group_name = "${var.resource_group_name}"
  location = "westus"
  prefix = "terraform-test"
}

module "computegroup" { 
    source              = "Azure/computegroup/azurerm"
    resource_group_name = "${var.resource_group_name}"
    location            = "westus"
    vm_size             = "Standard_A0"
    admin_username      = "azureuser"
    admin_password      = "ComplexPassword"
    ssh_key             = "~/.ssh/id_rsa.pub"
    nb_instance         = 2
    vm_os_publisher     = "Canonical"
    vm_os_offer         = "UbuntuServer"
    vm_os_sku           = "14.04.2-LTS"
    vnet_subnet_id      = "${module.network.vnet_subnets[0]}"
    load_balancer_backend_address_pool_ids = "${module.loadbalancer.azurerm_lb_backend_address_pool_id}"
    lb_port             = { 
                            http = ["80", "Tcp", "80"]
                            https = ["443", "Tcp", "443"]
                          }
    tags                = {
                            environment = "dev"
                            costcenter  = "it"
                          }
}

output "vmss_id"{
  value = "${module.computegroup.vmss_id}"
}

```


Outputs
=======

- `vmss_id` - Id ot the virtual machine scale set

Authors
=======
Originally created by [Damien Caro](http://github.com/dcaro)

License
=======

[MIT](LICENSE)
