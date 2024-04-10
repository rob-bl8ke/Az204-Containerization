
# Build and push from a local `Dockerfile`

- Exercise: [Build and run a container image by using Azure Container Registry Tasks](https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/6-build-run-image-azure-container-registry)

## Build and push to ACR

```bash
#!/bin/bash

# Source:
# Exercise: Build and run a container image by using Azure Container Registry Tasks
# https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/6-build-run-image-azure-container-registry

resourceGroup=...-acr-resourcegroup
containerRegistry=...acrregistry
location=eastus

az login

az group create --name $resourceGroup --location $location

az acr create --name $containerRegistry \
    --resource-group $resourceGroup --location $location \
    --sku Basic

echo FROM mcr.microsoft.com/hello-world > Dockerfile

az acr build --image sample/hello-world:v1 \
    --registry $containerRegistry \
    --file Dockerfile .

```

## Verify the results

```bash
#!/bin/bash



resourceGroup=...-acr-resourcegroup
containerRegistry=...acrregistry
location=eastus

# az login

az acr repository show-tags --name $containerRegistry \
    --repository sample/hello-world --output table
```

## Run Image

```bash
#!/bin/bash

# Not sure how this works yet...
az acr run --registry ...acrregistry --cmd '$Registry/sample/hello-world:v1' /dev/null
# az acr run --registry ...acrregistry oci://myregistry.azurecr.io/
```

## Clean up

```bash
#!/bin/bash

az group delete --name ...-acr-resourcegroup --no-wait --yes
rm Dockerfile
```

# References

- [Docker Docs](https://docs.docker.com/get-started/overview/)