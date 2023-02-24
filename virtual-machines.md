# Virtual Machines

## Creating a Windows VM

### Create Resource Group

Azure CLI

```cmd
az group create --name myResourceGroup --location eastus
```

Azure PowerShell

```cmd
New-AzResourceGroup -Name 'myResourceGroup' -Location 'EastUS'
```

### Create the VM

Azure CLI

```cmd
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image Win2022AzureEditionCore \
    --public-ip-sku Standard \
    --admin-username azureuser
```

Azure PowerShell

```cmd
New-AzVm `
    -ResourceGroupName 'myResourceGroup' `
    -Name 'myVM' `
    -Location 'East US' `
    -VirtualNetworkName 'myVnet' `
    -SubnetName 'mySubnet' `
    -SecurityGroupName 'myNetworkSecurityGroup' `
    -PublicIpAddressName 'myPublicIpAddress' `
    -OpenPorts 80,3389
```

### Install web server

Azure CLI

```cmd
az vm run-command invoke -g MyResourceGroup -n MyVm --command-id RunPowerShellScript --scripts "Install-WindowsFeature -name Web-Server -IncludeManagementTools"
```

Azure PowerShell

```cmd
Invoke-AzVMRunCommand -ResourceGroupName 'myResourceGroup' -VMName 'myVM' -CommandId 'RunPowerShellScript' -ScriptString
'Install-WindowsFeature -Name Web-Server -IncludeManagementTools'
```

#### Open port 80 for web traffic

Azure CLI

```cmd
az vm run-command invoke -g MyResourceGroup -n MyVm --command-id RunPowerShellScript --scripts "Install-WindowsFeature -name Web-Server -IncludeManagementTools"
```

### Access the web server

With the port open, we can access the server using the public ip address that was returned by the create VM command.

### Clean up resources

Azure CLI

```cmd
az group delete --name myResourceGroup
```

Azure PowerShell

```cmd
Remove-AzResourceGroup -Name 'myResourceGroup'
```

## Creating a Linux VM

### Define Environment Variables

Azure CLI

```cmd
export RESOURCE_GROUP_NAME=myResourceGroup
export LOCATION=eastus
export VM_NAME=myVM
export VM_IMAGE=debian
export ADMIN_USERNAME=azureuser
```

### Create a resource group

Azure CLI

```cmd
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION
```

Azure PowerShell

```cmd
New-AzResourceGroup -Name 'myResourceGroup' -Location 'EastUS'
```

### Create virtual machine

Azure CLI

```cmd
az vm create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name $VM_NAME \
  --image $VM_IMAGE \
  --admin-username $ADMIN_USERNAME \
  --generate-ssh-keys \
  --public-ip-sku Standard
```

Azure PowerShell

```cmd
New-AzVm `
    -ResourceGroupName 'myResourceGroup' `
    -Name 'myVM' `
    -Location 'East US' `
    -Image Debian `
    -size Standard_B2s `
    -PublicIpAddressName myPubIP `
    -OpenPorts 80 `
    -GenerateSshKey `
    -SshKeyName mySSHKey
```

### Install web server

Azure CLI

```cmd
az vm run-command invoke \
   --resource-group $RESOURCE_GROUP_NAME \
   --name $VM_NAME \
   --command-id RunShellScript \
   --scripts "sudo apt-get update && sudo apt-get install -y nginx"
```

Azure PowerShell

```cmd
Invoke-AzVMRunCommand `
   -ResourceGroupName 'myResourceGroup' `
   -Name 'myVM' `
   -CommandId 'RunShellScript' `
   -ScriptString 'sudo apt-get update && sudo apt-get install -y nginx'
```

#### Open port 80 for web traffic

Azure CLI

```cmd
az vm open-port --port 80 --resource-group $RESOURCE_GROUP_NAME --name $VM_NAME
```

#### Get public IP address

Azure CLI

```cmd
export IP_ADDRESS=$(az vm show --show-details --resource-group $RESOURCE_GROUP_NAME --name $VM_NAME --query publicIps --output tsv)
```

Azure PowerShell

```cmd
Get-AzPublicIpAddress -Name myPubIP -ResourceGroupName myResourceGroup | select "IpAddress"
```

### Access the web server

With the port open, we can access the server using the public ip address that was returned by the previous command.

### Clean up resources

Azure CLI

```cmd
az group delete --name $RESOURCE_GROUP_NAME --no-wait --yes --verbose
```

Azure PowerShell

```cmd
Remove-AzResourceGroup -Name 'myResourceGroup'
```

### Additional Resources

- [Create and manage VM with PowerShell](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-manage-vm)
- [VM Management](https://learn.microsoft.com/en-us/previous-versions/azure/virtual-machines/windows/tutorial-govern-resources)
