# Deploy a Container App

```bash
#!/bin/bash

# Source: Exercise - Deploy a container app
# https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/3-exercise-deploy-app

az logout

az login

# Install the Azure Container Apps extension for CLI, upgrade it if already installed...
az extension add --name containerapp --upgrade

# Register the Microsoft.App namespace
az provider register --namespace Microsoft.App

# Register Microsoft.OperationalInsights provider for the Azure Monitor Log Analytics workspace
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

## Updating your Container App

Modify environment variables, compute resources, scale parameters, and deploy a different image using...

```PowerShell
az container update `
    --name "album-api" `
    --resource-group $resourceGroup`
    --image  mcr.microsoft.com/azuredocs/containerapps-helloworld
```

```PowerShell
az container revision list `
    --name "album-api" `
    --resource-group $resourceGroup `
    -o table
```

## Defining Secrets

Note how you can access your secrets through environment variables by prepending the `secretref` prefix before the secret key name.

```PowerShell
az containerapp create `
    --resource-group $resourceGroup `
    --name $containerAppName `
    --environment $containerAppEnvironment `
    --image "mcr.microsoft.com/azuredocs/containerapps-helloworld" `
    --secrets "queue-connection-string=$connectionString" `
    --env-vars "QueueName=myqueue" "ConnectionString=secretref:queue-connection-string"
```

## Clean up

```bash
az group delete --name ...-resourcegroup --yes
```