# [@matwolbec](https://matwolbec.github.io) - Terraform esssentials
Requirements and infos for essential terraform usage  

Terraform is an open-source infrastructure as code software tool that enables you to safely and predictably create, change, and improve infrastructure.

For further info, check [https://www.terraform.io/](https://www.terraform.io/)

## First step  
Install terraform: See [https://www.terraform.io/downloads](https://www.terraform.io/downloads), choose your OS and follow the guide.


## Basic commands  
```bash
terraform init #  Prepare your working directory for other commands
terraform plan # Show changes required by the current configuration
terraform apply # Create or update infrastructure
terraform destroy # Destroy previously-created infrastructure
terraform import # Associate existing infrastructure with a Terraform resource
```


## Setting directories and initial file

You must define your workdir. Usually the workdir will be the name of your Infra Project. In this case I will create a directory for this tutorial:

```bash
mkdir howto-terraform
cd howto-terraform
```

Now you can create another dir to define different environments, like ```staging``` and ```production```:
```bash
mkdir staging production
```


