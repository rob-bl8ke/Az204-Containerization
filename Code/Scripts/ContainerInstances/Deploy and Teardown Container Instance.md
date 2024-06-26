# Container Instances

## Create a Basic Container Instance (PowerShell and Azure CLI)

### Overview
- Create a resource group.
- Create a container.
    - Give it an image and a FQDN and expose it via port 80.

### Code

Set your variables. The `$dnsNameLabel` will become the fully qualified domain name (FQDN). Structure is `customlabel.azureregion.azurecontainer.io`.
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

Create the container using an image hosted at Microsoft. Expose port 80. If you wish to specify more than one port use a space seperated list such as "80 443".
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

Code originally [based on this lab](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-powershell). The image (`iis:nanoserver`) used in the lab is no longer supported and the closest thing you could find to it is [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver). Unsure as to whether this uses IIS as a web server or not. But it is Windows-based.

### Overview
- Create a new resource group (`New-AzResourceGroup`)
- Create a new container instance object (`New-AzContainerInstanceObject`)
- Create a new container group object (`New-AzContainerGroup`) and assign the container to it.

### Code
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

## Create two Container Instances (Restart policy, CPU, memory, and environment variables)

> Note: To create a group (so that the instances can communicate) one has to deploy using either YAML or ARM templates

### Overview

- Create a resource group.
- Create a container. Give it a FQDN (exposed on port 80 and port 443) and a restart policy. Request a single CPU and memory of 1 GB.
- Create a container with environment variables

> Although this exercise does successfully create a container group, unfortunately you can only see the properties of the other container, there is no way to interact with it as the two containers don't talk to each other and the word-count container is not exposed in any way. To view its environment variables one has to `az container show` it.

### Code
- The docker images used for the following images are hosted here:
    - [ACI Hello World](https://hub.docker.com/_/microsoft-azuredocs-aci-helloworld)
    - [ACI Word Count](https://hub.docker.com/_/microsoft-azuredocs-aci-wordcount)

> This script is to be run in bash and not PowerShell

```bash
#!/bin/bash

# Source:
# Exercise - Deploy a container instance by using the Azure CLI
# https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/3-run-azure-container-instances-cloud-shell

resourceGroup=...resourcegroup
container=...-hello-world-container
wordContainer=...-word-count-container
location=eastus

# az login

az group create --name $resourceGroup --location $location

DNS_NAME_LABEL=aci-example-$RANDOM
# FQDN will end up as DNS_NAME_LABEL.azureregion.azurecontainer.io

# Run container group and expose dns with ext. port
az container create --resource-group $resourceGroup \
    --name $container \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --dns-name-label $DNS_NAME_LABEL \
    --restart-policy OnFailure \
    --ports 80 443 \
    --cpu 1 \
    --memory 1

# Create a container with environment variables
# - in PowerShell should be "NumWords=5 MinLength=8"
# - at the old style windows command prompt should be "NumWords"="5" "MinLength"="8"
az container create \
    --resource-group $resourceGroup \
    --name $wordContainer \
    --image mcr.microsoft.com/azuredocs/aci-wordcount \
    --restart-policy OnFailure \
    --environment-variables 'NumWords'='5' 'MinLength'='8' \
    --secure-environment-variables 'ConnectionString'='...'


az container show --resource-group $resourceGroup \
    --name $container \
    --query "{ FQDN:ipAddress.fqdn, ProvisioningState:provisioningState }" \
    --out table

# Show the environment variables of the word-count container
az container show --resource-group $resourceGroup \
    --name $wordContainer \
    --query "{ EnvironmentVariables:containers[0].environmentVariables }" \
    --out yaml
```

## Tear down

```bash
#!/bin/bash

az group delete --name ...resourcegroup --no-wait --yes
```

# References

- [Docker Docs](https://docs.docker.com/get-started/overview/)