

## Importing Docker Images into the registry
```bash
# You've created a new instance of Azure Container Registry (ACR) named myreg1.
# Publish the most recent Windows Core image into myreg1.
az acr import \
    --name myreg1 \
    --source mcr.microsoft.com/windows/servercore:latest

# Source: https://learn.microsoft.com/en-us/azure/container-registry/container-registry-import-images?tabs=azure-cli
az acr import \
    --name myreg1 \
    --source mcr.microsoft.com/windows/servercore:latest

# Import using your Docker Hub user credentials (recommended)
az acr import \
    --name myregistry \
    --source docker.io/tensorflow:latest-gpu \
    --image tensorflow:latest-gpu \
    --username "docker-user-name" \
    --password "345fseIdk34"
```

## Example X

```bash
# build and deploy to Azure Container Registry
az acr build --registry $ACR_NAME --image albumapp-ui .

```