# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)

Back to [page 2](terraform-azure-2.md)

## Creating our VM

Add to your ```main.tf```
```s
resource "azurerm_linux_virtual_machine" "default" {
  name                = "${var.locat}-${var.vm}"
  resource_group_name = azurerm_resource_group.default.name
  location            = azurerm_resource_group.default.location
  size                = "Standard_F2"
  admin_username      = "adminuser"
  network_interface_ids = [
    azurerm_network_interface.default.id,
  ]

  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_azure_tf.pub")
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}
```

Add to ```variables.tf```
```s
//Computing
variable "vm" {
    default = "vmlinux"
}
```

Create a RSA KEY, the public key will be imported to your instance in the admin_ssh_key session. Change the name or the path of the file if needed.
```bash
ssh-keygen -t rsa -b 4096
# Type the path: ~/.ssh/id_azure_tf.pub
```

Run terraform
```bash
terraform apply
```



## Next steps

Go to [page 6](terraform-azure-6.md) to create our kubernetes cluster.
