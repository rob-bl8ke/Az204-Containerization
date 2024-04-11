# Container Instances

## Create a Basic Container Instance (PowerShell and Azure CLI)

Set your variables. The `$dnsNameLabel` will become the fully qualified domain name (FQDN).
```powershell
$resourceGroup = _ 
$location = southafricasouth
$image =  "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
$containerName = _
$dnsNameLabel = _
```
Login to Azure and create your group
```powershell
az login
az group create --name $resourceGroup --location $location
```

Create the container using an image hosted at Microsoft. Expose port 80.
```powershell
az container create --resource-group $resourceGroup `
    --name $containerName `
    --image $image `
    --ports 80 `
    --dns-name-label $dnsNameLabel `
    --location $location
```

Capture the FQDN. If you copy it you can navigate directly to the container instance landing page.
```powershell
az container show --resource-group $resourceGroup `
    --name $containerName `
    --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
```

## Create a Basic Container Instance (PowerShell `Az` Module)

```ps
$resourceGroup = "" 
$location = ""
$containerName = ""
$containerGroupName = ""
$dnsNameLabel = ""
$image =  "mcr.microsoft.com/azuredocs/aci-helloworld"
```

```ps
Connect-AzAccount

New-AzResourceGroup -Name $resourceGroup -Location $location
```

Create the container, adding port 80 to it
```ps
$port = New-AzContainerInstancePortObject -Port 80

$container = New-AzContainerInstanceObject `
    -Name $containerName `
    -Image $image `
    -RequestCpu 1 `
    -RequestMemoryInGb 1.5 `
    -Port @($port)
```

#### Alternate ports (not part of this exercise... skip)
You'd add TCP ports like this...
```ps
$port1 = New-AzContainerInstancePortObject -Port 8000 -Protocol TCP
$port2 = New-AzContainerInstancePortObject -Port 8001 -Protocol TCP

$container = New-AzContainerInstanceObject `
    -Name $containerName `
    -Image $image `
    -RequestCpu 1 `
    -RequestMemoryInGb 1.5 `
    -Port @($port1, $port2)
```

Finally, create the container group. Note that the way this is shown in the examples is incorrect and does not work. Things have changed this [this version of the Az module](https://learn.microsoft.com/en-us/powershell/module/az.containerinstance/new-azcontainergroup?view=azps-11.4.0) and some things have moved around. You updated before trying the examples and could not get it to work until you looked at the updated examples.

```ps
New-AzContainerGroup -ResourceGroupName $resourceGroup `
    -Name $containerGroupName `
    -Location $location `
    -Container $container `
    -OsType Linux `
    -RestartPolicy "Never" `
    -IpAddressType Public `
    -IPAddressDnsNameLabel $dnsNameLabel
```

At this point, you should be able to navigate to your container landing page.
```ps
Get-AzContainerGroup -ResourceGroupName $resourceGroup `
    -Name $containerGroupName `
    | Select-Object -Property  ResourceGroupName, IPAddressDnsNameLabel, Name 
```
To remove the resource and the container group and instance.
```ps
Remove-AzContainerGroup -ResourceGroupName myResourceGroup -Name myContainerGroup
```

## Create a container instance (Restart policy, CPU, memory, and environment variables)

```bash
#!/bin/bash

# Source:
# Exercise - Deploy a container instance by using the Azure CLI
# https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/3-run-azure-container-instances-cloud-shell

resourceGroup=...resourcegroup
container=...container
location=eastus

# az login

az group create --name $resourceGroup --location $location

DNS_NAME_LABEL=aci-example-$RANDOM

# Run container group and expose dns with ext. port
az container create --resource-group $resourceGroup \
    --name $container \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --dns-name-label $DNS_NAME_LABEL \
    --restart-policy OnFailure \
    --ports 80 \
    --cpu 1 \
    --memory 1

# Create a container with environment variables
az container create \
    --resource-group $resourceGroup \
    --name mywordcontainer \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure \
    --environment-variables 'NumWords'='5' 'MinLength'='8'


az container show --resource-group $resourceGroup \
    --name $container \
    --query "{ FQDN:ipAddress.fqdn, ProvisioningState:provisioningState }" \
    --out table
```

## Tear down

```bash
#!/bin/bash

az group delete --name ...resourcegroup --no-wait --yes
```

# References

- [Docker Docs](https://docs.docker.com/get-started/overview/)