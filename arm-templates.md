# ARM Templates

- Repeatable way of deploying resources
- Declarative
- Can be automated

## Basic ARM Template

These are the mandatory properties in an ARM template. Rest are all optional.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": []
}
```

## Sample ARM Template

This is a sample ARM template for deploying the following resources:

- 1 Virtual Network with 2 subnets
- 1 Storage Account
- 1 Virtual Machine (Standard_B1s)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "my-new-storage-accountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_ZRS",
        "Standard_GRS",
        "Standard_RAGRS",
        "Premium_LRS"
      ]
    },
    "my-new-linux-vmName": {
      "type": "string",
      "minLength": 1
    },
    "my-new-linux-vmAdminUserName": {
      "type": "string",
      "minLength": 1
    },
    "my-new-linux-vmAdminPassword": {
      "type": "securestring"
    },
    "my-new-linux-vmUbuntuOSVersion": {
      "type": "string",
      "defaultValue": "14.04.2-LTS",
      "allowedValues": ["12.04.5-LTS", "14.04.2-LTS", "15.04"]
    }
  },
  "resources": [
    {
      "name": "my-new-vnet",
      "type": "Microsoft.Network/virtualNetworks",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-06-15",
      "dependsOn": [],
      "tags": {
        "displayName": "my-new-vnet"
      },
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["[variables('my-new-vnetPrefix')]"]
        },
        "subnets": [
          {
            "name": "[variables('my-new-vnetSubnet1Name')]",
            "properties": {
              "addressPrefix": "[variables('my-new-vnetSubnet1Prefix')]"
            }
          },
          {
            "name": "[variables('my-new-vnetSubnet2Name')]",
            "properties": {
              "addressPrefix": "[variables('my-new-vnetSubnet2Prefix')]"
            }
          }
        ]
      }
    },
    {
      "name": "[variables('my-new-storage-accountName')]",
      "type": "Microsoft.Storage/storageAccounts",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-06-15",
      "dependsOn": [],
      "tags": {
        "displayName": "my-new-storage-account"
      },
      "properties": {
        "accountType": "[parameters('my-new-storage-accountType')]"
      }
    },
    {
      "name": "[variables('my-new-linux-vmNicName')]",
      "type": "Microsoft.Network/networkInterfaces",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-06-15",
      "dependsOn": [
        "[concat('Microsoft.Network/virtualNetworks/', 'my-new-vnet')]"
      ],
      "tags": {
        "displayName": "my-new-linux-vmNic"
      },
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "subnet": {
                "id": "[variables('my-new-linux-vmSubnetRef')]"
              }
            }
          }
        ]
      }
    },
    {
      "name": "[parameters('my-new-linux-vmName')]",
      "type": "Microsoft.Compute/virtualMachines",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-06-15",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('my-new-storage-accountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('my-new-linux-vmNicName'))]"
      ],
      "tags": {
        "displayName": "my-new-linux-vm"
      },
      "properties": {
        "hardwareProfile": {
          "vmSize": "[variables('my-new-linux-vmVmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('my-new-linux-vmName')]",
          "adminUsername": "[parameters('my-new-linux-vmAdminUsername')]",
          "adminPassword": "[parameters('my-new-linux-vmAdminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "[variables('my-new-linux-vmImagePublisher')]",
            "offer": "[variables('my-new-linux-vmImageOffer')]",
            "sku": "[parameters('my-new-linux-vmUbuntuOSVersion')]",
            "version": "latest"
          },
          "osDisk": {
            "name": "my-new-linux-vmOSDisk",
            "vhd": {
              "uri": "[concat('http://', variables('my-new-storage-accountName'), '.blob.core.windows.net/', variables('my-new-linux-vmStorageAccountContainerName'), '/', variables('my-new-linux-vmOSDiskName'), '.vhd')]"
            },
            "caching": "ReadWrite",
            "createOption": "FromImage"
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('my-new-linux-vmNicName'))]"
            }
          ]
        }
      }
    }
  ],
  "variables": {
    "my-new-vnetPrefix": "10.0.0.0/16",
    "my-new-vnetSubnet1Name": "Subnet-1",
    "my-new-vnetSubnet1Prefix": "10.0.0.0/24",
    "my-new-vnetSubnet2Name": "Subnet-2",
    "my-new-vnetSubnet2Prefix": "10.0.1.0/24",
    "my-new-storage-accountName": "[concat('my-new-storage-account', uniqueString(resourceGroup().id))]",
    "my-new-linux-vmImagePublisher": "Canonical",
    "my-new-linux-vmImageOffer": "UbuntuServer",
    "my-new-linux-vmOSDiskName": "my-new-linux-vmOSDisk",
    "my-new-linux-vmVmSize": "Standard_B1s",
    "my-new-linux-vmVnetID": "[resourceId('Microsoft.Network/virtualNetworks', 'my-new-vnet')]",
    "my-new-linux-vmSubnetRef": "[concat(variables('my-new-linux-vmVnetID'), '/subnets/', variables('my-new-vnetSubnet1Name'))]",
    "my-new-linux-vmStorageAccountContainerName": "vhds",
    "my-new-linux-vmNicName": "[concat(parameters('my-new-linux-vmName'), 'NetworkInterface')]"
  }
}
```

When the above ARM template is deployed, this is what it creates

![ARM Deployment Visualization](images\arm-template-deploy-result.png)
