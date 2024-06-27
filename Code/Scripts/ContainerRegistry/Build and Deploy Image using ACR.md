
# Build and push from local `Dockerfile` using ACR

- Exercise: [Build and run a container image by using Azure Container Registry Tasks](https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/6-build-run-image-azure-container-registry)
- Tutorial: [Build and deploy container images in the cloud with Azure Container Registry Tasks](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tutorial-quick-task)
    - A much better tutorial. Use this one to configure registry authentication using KeyVault together with a service principal to store credentials. 

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
```
Create a new Azure Container Registry (ACR)
```bash
az acr create --name $containerRegistry \
    --resource-group $resourceGroup --location $location \
    --sku Basic\
```

Create and then view the contents of your new `Dockerfile`.

```bash
echo FROM mcr.microsoft.com/hello-world > Dockerfile
```
Assuming your terminal's current directory is the same as your `Dockerfile` and you are logged in and have access to the name of your ACR, run the following command to build and deploy your image to the registry.

```bash
az acr build --image sample/hello-world:v1 \
    --registry $containerRegistry \
    --file Dockerfile .
```

## Verify the results

```bash
az acr repository show-tags --name $containerRegistry \
    --repository sample/hello-world --output table
```

## Run Image

Run the `sample/hello-world:v1` container image from your container registry by using the az acr run command. The following example uses $Registry to specify the registry where you run the command.
This is the equivalent of `docker run`.

```bash
az acr run --registry $containerRegistry --cmd '$containerRegistry/sample/hello-world:v1' /dev/null
```
...or in PowerShell

```powershell
az acr run --registry $containerRegistry --cmd "${$containerRegistry}/sample/hello-world:v1"
```
The cmd parameter in this example runs the container in its default configuration, but cmd supports other docker run parameters or even other docker commands.

## Clean up

```bash
#!/bin/bash

az group delete --name ...-acr-resourcegroup --no-wait --yes
rm Dockerfile
```

# References

- [Docker Docs](https://docs.docker.com/get-started/overview/)