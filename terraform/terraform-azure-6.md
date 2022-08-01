# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)

Back to [page 5](terraform-azure-5.md)

## Before the cluster, know about identity
An Azure Kubernetes Service (AKS) cluster requires an identity to access Azure resources like load balancers and managed disks.
This identity can be either a managed identity or a service principal.

By default (which we are using in next steps), when you create an AKS cluster a system-assigned managed identity automatically created.
The identity is managed by the Azure platform and doesn't require you to provision or rotate any secrets. 

To use a service principal, you have to create one, as AKS does not create one automatically.
Clusters using a service principal eventually expire and the service principal must be renewed to keep the cluster working.
Managing service principals adds complexity, thus it's easier to use managed identities instead.
The same permission requirements apply for both service principals and managed identities.

Managed identities are essentially a wrapper around service principals, and make their management simpler.
Managed identities use certificate-based authentication, and each managed identities credential has an expiration of 90 days and it's rolled after 45 days.
AKS uses both system-assigned and user-assigned managed identity types, and these identities are immutable.


## The terraform files

Go to the ```production``` directory and create a new ```main.tf``` and ```variables.tf``` file.
```bash
cd ../production
touch main.tf variables.tf outputs.tf
```



Open the ```variables.tf``` file with your IDE and add:
```s
// Environment
variable "locat" {
    default = "australiaeast"
}

variable "rg" {
    default = "production"
}

variable env_tag {
    default = "production"
}


//Computing
variable "kc" {
    default = "aks_cluster"
}
```


Open the ```main.tf``` file:
```s
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "tfstate"
    storage_account_name = "tfstate<id>"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }

}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}

# Create a resource group
resource "azurerm_resource_group" "default" {
  name     = var.rg
  location = var.locat
  tags = {
    "env"     = var.env_tag
    "project" = "myapp"
  }
}


resource "azurerm_kubernetes_cluster" "default" {
  name                = "${var.locat}-${var.kc}"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name
  dns_prefix          = "exampleaks1"

  default_node_pool {
    name       = "default"
    node_count = 1
    vm_size    = "Standard_D2_v2"
  }

  identity {
    type = "SystemAssigned"
  }

  tags = {
    Environment = "Production"
  }
}

output "client_certificate" {
  value     = azurerm_kubernetes_cluster.default.kube_config.0.client_certificate
  sensitive = true
}

output "kube_config" {
  value = azurerm_kubernetes_cluster.default.kube_config_raw

  sensitive = true
}
```

And the ```outputs.tf``` file:
```s
resource "local_file" "default" {
  depends_on   = [azurerm_kubernetes_cluster.default]
  filename     = "kubeconfig"
  content      = azurerm_kubernetes_cluster.default.kube_config_raw
}
```

Run
```bash
terraform init
terraform plan
terraform apply
```

## Running kubectl
Now you already can use ```kubectl``` to manage your kubernetes.
See [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) to install it if you don't have it yet.
```bash
kubectl get nodes --kubeconfig kubeconfig
```

If you don't want to use every time the ```--kubeconfig``` parameter, you can run:
```bash
cp kubeconfig ~/.kube/config
```


## Next steps

Go to [page 7](terraform-azure-7.md) to deploy our application.


