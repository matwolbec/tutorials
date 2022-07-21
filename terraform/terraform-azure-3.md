# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)

Back to [page 2](terraform-azure-2.md)

## Configure tfstate to a backend provider storage
For safety reasons, we don't want to store the state of our services on our computer, so we will configure a Azure Backend Service to store the terraform state, current the ```terraform.tfstate``` file.

For more info, visit [https://docs.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=terraform](https://docs.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=terraform)

Creating a terraform file to create an storage account
```bash
cd ../tfstatus-storage
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

variable env_tag {
    default = "staging"
}
```

To use it on your ```main.tf```, just call: ```var.variable_name```

Ex:
```bash
location = var.locat
```

Furthermore, we need to create another resource group to store our instances, network, etc.
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
```

## Next steps

Go to [page 4](terraform-azure-4.md) to create our network.
