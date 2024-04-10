# Deploy a Container App

```bash
#!/bin/bash

# Source: Exercise - Deploy a container app
# https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/3-exercise-deploy-app

az logout

az login

# Install the Azure Container Apps extension for CLI
az extension add --name containerapp --upgrade

# Register the Microsoft.App namespace
az provider register --namespace Microsoft.App

# Register Microsoft.OperationalInsights provier
az provider register --namespace Microsoft.OperationalInsights


resourceGroup=...-resourcegroup
location=eastus
appContainerEnv=...-app-container-env

az group create --name $resourceGroup --location eastus

# Create the App Container Environment
az containerapp env create \
    --name $appContainerEnv \
    --resource-group $resourceGroup \
    --location $location

# Create the Container App (Note the little trick to pass back the dns name)
az containerapp create \
    --name my-container-app \
    --resource-group $resourceGroup \
    --environment $appContainerEnv \
    --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
    --target-port 80 \
    --ingress 'external' \
    --query properties.configuration.ingress.fqdn

# Take the domain name provided and browse to it... should get you to the welcome page!
```

## Clean up

```bash
az group delete --name ...-resourcegroup --yes
```