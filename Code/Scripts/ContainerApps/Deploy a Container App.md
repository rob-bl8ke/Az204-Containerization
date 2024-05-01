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

### Key Vault Concerns (and Managed Identity)

Since there is no direct integration with Azure Key Vault, it is recommended that you enable managed identity for your Container App. You can then use the Azure Key Vault SDK to access secrets.

Here is how you would assign a system assigned identity.
```Bash
# Assign a system-assigned managed identity
az containerapp identity assign \
	--name $containerAppName \
	--resource-name $resourceGroup
```

Here is how you would assign a user-assigned managed identity. 

```Bash
# Create a user-assigned managed identity
az identity create --resource-group $resourceGroup --name $userAssignedIdentity

# Assign a user-assigned managed identity
az containerapp identity assign \
	--name $containerAppName \
	--resource-group $resourceGroup \
	--identities /subscriptions/$subsriptionId/resourcegroups/$resourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$userAssignedIdentity
```

Similarly you can assign a managed identity the the App Service Environment.

```Bash
az containerapp env identity assign \
	--name $containerAppName \
	--resource-name $resourceGroup
```

## Clean up

```bash
az group delete --name ...-resourcegroup --yes
```