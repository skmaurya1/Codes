# Create a VM from a specialized VHD disk

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-vm-specialized-vhd%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## Prerequisite 
- VHD file that you want to create a VM from already exists in a storage account.

```
NOTE

This template will create an additional Standard_LRS storage account for enabling boot diagnostics each time you execute this template. To avoid running into storage account limits, it's best to delete the storage account when the VM is deleted.
```

This template creates a VM from a specialized VHD. The VHD file can be located in a storage account using a tool such as Azure Storage Explorer http://storageexplorer.com/

If you are looking to accomplish the above scenario through PowerShell instead of a template, you can use a PowerShell script like below

##### Variables
    ## Global
    $rgName = "hol-lab-sgpr"
    $location = "southeastasia"

    ## Storage
    $storageName = "sqlserverdwstoresg"
    $storageType = "Standard_LRS"

    ## Network
    $nicname = "sqlserverdwnw"
    $subnet1Name = "subnet1"
    $vnetName = "ssdwnet"
    $vnetAddressPrefix = "10.0.0.0/16"
    $vnetSubnetAddressPrefix = "10.0.0.0/24"

    ## Compute
    $vmName = "SqlServerDW"
    $computerName = "SqlServerDW"
    $vmSize = "Standard_Ds1_V2"
    $osDiskName = $vmName + "osDisk"

##### Resource Group
    New-AzureRmResourceGroup -Name $rgName -Location $location

##### Storage
    $storageacc = New-AzureRmStorageAccount -ResourceGroupName $rgName -Name $storageName -Type $storageType -Location $location

##### Network
    $pip = New-AzureRmPublicIpAddress -Name $nicname -ResourceGroupName $rgName -Location $location -AllocationMethod Dynamic
    $subnetconfig = New-AzureRmVirtualNetworkSubnetConfig -Name $subnet1Name -AddressPrefix $vnetSubnetAddressPrefix
    $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location -AddressPrefix $vnetAddressPrefix -Subnet $subnetconfig
    $nic = New-AzureRmNetworkInterface -Name $nicname -ResourceGroupName $rgName -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

##### Compute

    ## Setup local VM object
    $vm = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize

    $vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

    $osDiskUri = "https://storageatsea.blob.core.windows.net/storagecontainer-sea/sqlserverdwseadisk.vhd"
    $vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption attach -Windows

    ## Create the VM in Azure
    New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm -Verbose -Debug
