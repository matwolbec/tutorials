# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)
# Terraform on Azure Esssentials
Requirements and infos for essential terraform on azure usage  

## Initializing the Terraform file structure

To get providers documentations, use the [Terraform Registry](https://registry.terraform.io). You will find a lot of available providers there. In our case: [https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)

- Open the ```main.tf``` file with your IDE

- Like said on the guide above, insert the content below in the ```main.tf``` file:

```t
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}
```

- Run the command to install the provider package and set the workdir:

```bash
terraform init
```

## Workaround for WSL2
If you are using the WSL2 (linux on windows) like me, there is a DNS issue where the Terraform tries to resolute the ```management.azure.com``` without success.

For further info: [GitHub Microsoft WSL issues 8022](https://github.com/microsoft/WSL/issues/8022)  

I found this workaround, resolving the record with ```dig``` and adding to your ```/etc/hosts``` file. Another alternative is to set custom DNS server on your ```/etc/resolv.conf```.
```bash
sudo bash -c 'echo "$(dig management.azure.com | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}$") management.azure.com" >> /etc/hosts'
```

If you need to update the host IP address (usually if you have changed your region you will want it), run:
```bash
sudo bash -c "sed -i '/management.azure.com/d' /etc/hosts" ; sudo bash -c 'echo "$(dig management.azure.com | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}$") management.azure.com" >> /etc/hosts'
```

## Define our first resource
A resource group is a container that holds related resources for a solution.

You decide how you want to allocate resources to resource groups based on what makes the most sense for you.

A resource group could be all instances needed to run your application, like APP, DB and some Cache, or your can create a DB Cluster as one group and create another to your APP.

Generally, add resources that share the same lifecycle to the same resource group so you can easily deploy, update, and delete them as a group.

- PS: Microsoft Azure is limiting resources by location to free accounts, so we will use the ```australiaeast``` region where are available computing resources for testing purposes. 

In your ```main.tf``` file, add a code block:
```s
# Create a resource group
resource "azurerm_resource_group" "default" {
  name     = "staging-resources"
  location = "australiaeast"
}
```

Let's run terraform to see our plan (it could take some time):
```bash
terraform plan
```

And now let's apply and it will be provisioned on Azure. You have to confirm the action with ```yes```:
```bash
terraform apply
```

## tfstate file
After you run apply, the resource was generated. Now you should notice a new file on your workdir: ```terraform.tfstate```. Meanwhile, this file shouldn't be deleted once it's your resource control. When you edit your main.tf file and run ```plan``` or ```apply```, it will check this file, compare, and just then request the needed info to the provider API.

## Destroying our first resource
An important thing is to destroy your resources after you have done with your objectives, avoiding additional costs. Run the command and remember to type ```yes``` and confirm:
```bash
terraform destroy
```

## Adding resources
Tagging your resource is a good advice, you will have a clearer control of what is it and which environment it belongs. So let's add some tag to our resource group in ```main.tf```:
```s
  tags = {
    "env" = "staging"
    "project" = "myapp"
  }
```

The complete block should be like:
```s
resource "azurerm_resource_group" "default" {
  name     = "staging-resources"
  location = "australiaeast"
  tags = {
    "env" = "staging"
    "project" = "myapp"
  }
}
```

And run:
```bash
terraform apply
```

## Configure tfstate to a backend provider storage
For safety reasons, we don't want to store the state of our services on our computer, so we will configure a Azure Backend Service to store the terraform state, current the ```terraform.tfstate``` file.

For more info, visit [https://docs.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=terraform](https://docs.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=terraform)

Creating a terraform file to create an storage account
```bash
cd ../storage
touch main.tf
```

Open the file and add the content:
```s
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "random_string" "resource_code" {
  length  = 5
  special = false
  upper   = false
}

resource "azurerm_resource_group" "tfstate" {
  name     = "tfstate"
  location = "East US"
}

resource "azurerm_storage_account" "tfstate" {
  name                     = "tfstate${random_string.resource_code.result}"
  resource_group_name      = azurerm_resource_group.tfstate.name
  location                 = azurerm_resource_group.tfstate.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  allow_nested_items_to_be_public = true

  tags = {
    environment = "staging"
  }
}

resource "azurerm_storage_container" "tfstate" {
  name                  = "tfstate"
  storage_account_name  = azurerm_storage_account.tfstate.name
  container_access_type = "blob"
}
```

And run:
```bash
terraform init
terraform plan
terraform apply
```

You must store the id created. Get it:
```bash
terraform show | grep storage_account_name
```

## Adding backend

Now you've created the storage, we will configure our backend. Go back to staging directory:
```bash
cd ../staging/
```

And let's work on the ```staging/main.tf```.

Take a look on [https://www.terraform.io/language/settings/backends/azurerm](https://www.terraform.io/language/settings/backends/azurerm)

The backend is declared on the terraform section like below:
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
```

Remember to substitute the ```<id>``` value for ```storage_account_name``` field.

Save your file and run terraform:
```bash
terraform init
terraform plan
terraform apply
```

## Virtual Network

Create a virtual network for our services and subnets to isolate some resources.

[https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_network](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_network)

Create a new ```resource``` section on your ```staging/main.tf``` file:
```s
resource "azurerm_virtual_network" "default" {
  name                = "staging-network"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name
  address_space       = ["10.0.0.0/16"]

  subnet {
    name           = "internal"
    address_prefix = "10.0.1.0/24"
  }

  tags = {
    environment = "staging"
  }
}
```

And run terraform:
```bash
terraform plan
terraform apply
```

## Using variables
For a better maintanance, let's create a variable file and set our standard settings. The file must be named ```variables.tf```.
```bash
touch variables.tf
```

Open the file and add:
```s
variable "locat" {
    default = "australiaeast"
}

variable "rg" {
    default = "staging-resources"
}

variable "vnet" {
    default = "staging-network"
}
```

To use it on your ```main.tf```, just call: ```var.variable_name```

Ex:
```bash
location = var.locat
```

So our ```main.tf``` file will be like this:
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
    storage_account_name = "tfstatedye1v"
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
    "env"     = "staging"
    "project" = "myapp"
  }
}

resource "azurerm_virtual_network" "default" {
  name                = var.vnet
  location            = var.locat
  resource_group_name = var.rg
  address_space       = ["10.0.0.0/16"]

  subnet {
    name           = "internal"
    address_prefix = "10.0.1.0/24"
  }

  tags = {
    environment = "staging"
  }
}
```