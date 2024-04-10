
# Communicate between Container Apps

## Step 1
```bash
#!/bin/bash

## Source: https://learn.microsoft.com/en-us/azure/container-apps/quickstart-code-to-cloud?tabs=bash%2Ccsharp&pivots=github-build

# First upgrade the local Azure command-line interface to te latest version
az upgrade

# Add the containerapp extension (or update it)
az extension add --name containerapp --upgrade

# Login to Azure
az login

# Once logged in, you can register these two namespaces.
# It is safe to run these commands multiple times... there is no
# impact if the namespaces are already registered.
# However, they require an active connection to Azure.
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights

RESOURCE_GROUP="album-containerapps"
LOCATION="canadacentral"
ENVIRONMENT="env-album-containerapps"
API_NAME="album-api"

# Fork this repository... and insert github repo
githubRepo="https://github.com/bobbyache/containerapps-albumapi-csharp"

# An all-in-one command that performs all the steps to go from
# code to a live Azure Container App instance.
#.. this command is also re-runnable.. the app will be updated
#.. to a new version if one already exists.
az containerapp up \
    --name $API_NAME \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --environment $ENVIRONMENT \
    --context-path ./src \
    --repo $githubRepo
```

## Step 2

```bash
mkdir $SUB_DIRECTORY
git clone https://github.com/$GITHUB_USERNAME/containerapps-albumui.git $SUB_DIRECTORY

cd $SUB_DIRECTORY/src

az acr build --registry $ACR_NAME --image albumapp-ui .

cd ../..

rm -rf $SUB_DIRECTORY
```

## Step 3
```bash
#!/bin/bash

## Source: https://learn.microsoft.com/en-us/azure/container-apps/communicate-between-microservices?tabs=bash&pivots=acr-remote#communicate-between-container-apps
## Important! YOU NEED TO FETCH THE NAME OF YOUR ACR and add it here...

RESOURCE_GROUP="album-containerapps"
LOCATION="canadacentral"
ENVIRONMENT="env-album-containerapps"
API_NAME="album-api"
FRONTEND_NAME="...-album-api"
ACR_NAME="cad485cde82eacr"


API_BASE_URL=$(az containerapp show \
    --resource-group $RESOURCE_GROUP \
    --name $API_NAME \
    --query properties.configuration.ingress.fqdn -o tsv)

az containerapp create \
  --name $FRONTEND_NAME \
  --resource-group $RESOURCE_GROUP \
  --environment $ENVIRONMENT \
  --image $ACR_NAME.azurecr.io/albumapp-ui  \
  --target-port 3000 \
  --env-vars API_BASE_URL=https://$API_BASE_URL \
  --ingress 'external' \
  --registry-server $ACR_NAME.azurecr.io \
  --query properties.configuration.ingress.fqdn
```

# Deploy a Container App

```bash
#!/bin/bash

# Source: Exercise - Deploy a container app
# https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/3-exercise-deploy-app

# Install the Azure Container Apps extension for CLI
az extension add --name containerapp --upgrade

az login

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
# Note the use of "secretref" in "env-vars" to hide a sensitive environment variable
az containerapp create \
    --name my-container-app \
    --resource-group $resourceGroup \
    --environment $appContainerEnv \
    --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
    --target-port 80 \
    --ingress 'external' \
    --secrets "queue-connection-string=$CONNECTION_STRING" "key1=value1" "key2=value2" \
    --env-vars "QueueName=myQueue" "ConnectionString=secretref:queue-connection-string" \
    --query properties.configuration.ingress.fqdn

# Take the domain name provided and browse to it... should get you to the welcome page!
```

# ARM Template Multiple resources with Volume Mounts
```json
    "containers": [
        {
          "name": "main",
          "image": "[parameters('container_image')]",
          "env": [
            {
              "name": "HTTP_PORT",
              "value": "80"
            },
            {
              "name": "SECRET_VAL",
              "secretRef": "mysecret"
            }
          ],
          "resources": {
            "cpu": 0.5,
            "memory": "1Gi"
          },
          "volumeMounts": [
            {
              "mountPath": "/myfiles",
              "volumeName": "azure-files-volume"
            }
          ]
          "probes":[
              {
                  "type":"liveness",
                  "httpGet":{
                  "path":"/health",
                  "port":8080,
                  "httpHeaders":[
                      {
                          "name":"Custom-Header",
                          "value":"liveness probe"
                      }]
                  },
                  "initialDelaySeconds":7,
                  "periodSeconds":3
      // file is truncated for brevity
```

