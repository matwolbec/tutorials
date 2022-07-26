# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)

Back to [page 3](terraform-azure-3.md)

## Creating Network

Add the following resources to your ```main.tf```
```s
resource "azurerm_virtual_network" "default" {
  name                = "${var.locat}-${var.vnet}"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name
  address_space       = [var.vnet_prefix]

  tags = {
    env = var.env_tag
  }
}

resource "azurerm_subnet" "default" {
  name                 = "${var.locat}-${var.subnet}"
  resource_group_name  = azurerm_resource_group.default.name
  virtual_network_name = azurerm_virtual_network.default.name
  address_prefixes     = [var.subnet_prefix]
}

resource "azurerm_public_ip" "default" {
  name                = "${var.locat}-${var.vm}-ip"
  resource_group_name = azurerm_resource_group.default.name
  location            = azurerm_resource_group.default.location
  allocation_method   = "Static"

  tags = {
    env = var.env_tag
  }
}

resource "azurerm_network_security_group" "default" {
  name                = "${var.locat}-${var.vm}-nsg"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name

  security_rule {
    name                       = "SSH"
    priority                   = 200
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "HTTP"
    priority                   = 201
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  tags = {
    env = var.env_tag
  }
}

resource "azurerm_network_interface" "default" {
  name                = "${var.locat}-${var.nic}"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name

  ip_configuration {
    name                          = "${var.locat}-${var.subnet}"
    subnet_id                     = azurerm_subnet.default.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id = azurerm_public_ip.default.id
  }
}

resource "azurerm_network_interface_security_group_association" "default" {
  network_interface_id      = azurerm_network_interface.default.id
  network_security_group_id = azurerm_network_security_group.default.id
}
```

And this variables to ```variables.tf```
```s
//Network
variable "vnet" {
    default = "stagingNetwork"
}

variable "vnet_prefix" {
    default = "10.0.0.0/16"
}

variable "subnet" {
    default = "stagingSubnet"
}

variable "subnet_prefix" {
    default = "10.0.1.0/24"
}

variable "nic" {
    default = "stagingNic"
}
```

## Next steps

Go to [page 5](terraform-azure-5.md) to create our VM.
