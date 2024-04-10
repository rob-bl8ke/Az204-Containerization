
# Logs and Metrics

```bash
az container create \
    --resource-group $resourcegroup \
    --file env.yaml \

az webapp create \
    --name $webApp \
    --resource-group $resourceGroup \
    --plan $appServicePlan  \
    --deployment-container-image-name $image

az container create \
    --resource-group $resourcegroup \
    --name hellofiles \
    --image mcr.microsoft.com/azuredocs/aci-hellofiles \
    --dns-name-label $hellofiles \
    --ports 80 \
    --azure-file-volume-account-name $funcappstorage \
    --azure-file-volume-account-key $KEY \
    --azure-file-volume-share-name $filehare \
    --azure-file-volume-mount-path /aci/logs/

az container show --resource-group $resourcegroup \
    --name $securetest \
    --query "containers[0].environmentVariables"

az container show --resource-group $resourcegroup \
  --name hellofiles --query ipAddress.fqdn --output tsv

# Container troubleshooting

# To troubleshoot a container you can pull logs with...
az container logs \
    --resource-group exampro \
    --name prod-web-app

# To get diagnostic information during container starup you can use...
az container attach \
    --resource-group exampro \
    --name prod-web-app

# To start an interactive container (write to the container and execute remote commands)
az container exec \
    --resource-group exampro \
    --name prod-web-app \
    --exec-command /bin/bash

# To get metrics such as CPU usage (space delimited metrics)
az monitor metrics list \
    --resource $containerId \
    --metrics CPUUsage \
    --output table

# Get a list of metrics to look at...
az monitor metrics list-definitions --resource $containerId
```
# Environment Variables, Querying FQDN, and mounting Azure Files

```bash
#!/bin/bash

## Source: https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/3-run-azure-container-instances-cloud-shell

az group create --name $resourceGroup --location $location

DNS_NAME_LABEL=aci-example-$RANDOM

# Start container as a service with an external port (for communication)
az container create --resource-group $resourceGroup \
    --name $containerName \
    --image mcr.microsoft.com/azuredocs/aci-helloworld  \
    --ports 80 \
    --dns-name-label $DNS_NAME_LABEL --location $location

# Start a container as a task (restart policy can be Never, Always, OnFailure)
# Restart on initialization failure
az container create \
    --resource-group $resourceGroup \
    --name $containerName \
    --image $containerImage \
    --restart-policy OnFailure \
    --environment-variables 'NumWords'='5' 'MinLength'='8' \

az container show --resource-group az204-aci-rg 
    --name mycontainer 
    --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" 
    --out table

# Mount Azure Files (note only supported for Linux containers)
az container create \
    --resource-group $resourceGroup \
    --name hellofiles \
    --image mcr.microsoft.com/azuredocs/aci-hellofiles \
    --dns-name-label aci-demo \
    --ports 80 \
    --azure-file-volume-account-name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --azure-file-volume-account-key $STORAGE_KEY \
    --azure-file-volume-share-name $ACI_PERS_SHARE_NAME \
    --azure-file-volume-mount-path /aci/logs/

    --azure-files-volume-account-name $volumeAccountName \
    --azure-file-volume-account-key $storageKey \
    --azure-file-volume-share-name $fileShareName \
    --azure-file-volumn-mount-path /aci/logs/
```

# Multiple WebApp Containers

```bash
# Source: https://learn.microsoft.com/en-us/azure/app-service/tutorial-multi-container-app

# In order for this to work it must be Linux
az appservice plan create \
    --name myAppServicePlan \
    --resource-group myResourceGroup \
    --sku S1 \
    --is-linux

# Use the "--multi-container-config-type compose" to create a web app
# that uses a Docker Compose file.
az webapp create --resource-group blogResourceGroup \
    --plan blogServicePlan \
    --name blog \
    --multi-container-config-type compose \
    --multi-container-config-file docker-compose.yml

# Set the container for your existing web app
az webapp config container set \
    --resource-group myResourceGroup \
    --name blog \
    --multicontainer-config-type compose \
    --multicontainer-config-file docker-compose-wordpress.yml
```