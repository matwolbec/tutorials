# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)
# Terraform on Azure Esssentials
Requirements and infos for essential terraform on azure usage  

## About Terraform
Terraform is an open-source infrastructure as code software tool that enables you to safely and predictably create, change, and improve infrastructure.

For further info, check [https://www.terraform.io/](https://www.terraform.io/)

## Install requirements
- Terraform: Choose your OS and follow the guide [https://www.terraform.io/downloads](https://www.terraform.io/downloads)
- Azure CLI: Same as above [https://docs.microsoft.com/en-us/cli/azure/install-azure-cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- Azure Account: Create and earn U$200,00 to try explore it [https://azure.microsoft.com/en-us/free/](https://azure.microsoft.com/en-us/free/)


## Basic terraform commands  
```bash
terraform init #  Prepare your working directory for other commands
terraform plan # Show changes required by the current configuration
terraform apply # Create or update infrastructure
terraform destroy # Destroy previously-created infrastructure
terraform import # Associate existing infrastructure with a Terraform resource
```


## Setting directories and initial file

You must define your workdir. Usually the workdir will be the name of your Infra Project. 

In this case I will create a directory for this tutorial:
```bash
mkdir howto-terraform
cd howto-terraform
```
  
Now you can create another dir to define different environments, like ```staging``` and ```production```:
```bash
mkdir staging production tfstatus-storage
```
  
Create a ```main.tf``` file. We will use the ```staging``` directory first
```bash
cd staging
touch main.tf
```

## Logging into the Azure CLI

Before we can start to use Terraform, we have to login on Azure CLI:
```bash
az login
```

Get the subscription list (we don't need it right now, but you can have more subscriptions)
```bash
az account list
```

## Next steps
Go to [page 2](terraform-azure-2.md) to start using terraform
