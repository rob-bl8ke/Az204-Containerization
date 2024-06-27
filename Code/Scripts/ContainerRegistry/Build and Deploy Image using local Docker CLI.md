# Create, build, deploy, and run a locally created Docker image to ACR

This is a series of tutorials based on the above header. However there are lots more tutorials here and you should also be looking at "Deploy a multi-container group".

Follow the three part series from here:

- [1 Create container image](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-app)
- [2 Create container registry](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-acr)
- [3 Deploy application](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-deploy-app)

## [1 Create container image](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-app)

- Clone the sample app into your local directory.
- Build the image from the Dockerfile using your local Docker engine.
- Run the Docker container locally specifying the port mapping and navigate to it to verify that it is running successfully
```bash
#!/bin/bash

az login
cd /c/Code/azure/acr

# Clone the sample app
git clone https://github.com/Azure-Samples/aci-helloworld.git

# Build the image from the Dockerfile and tag it
docker build ./aci-helloworld -t aci-tutorial-app

# See the built image
docker images

# Run the container locally
docker run -d -p 8080:80 aci-tutorial-app
# Navigate to http://localhost:8080

```
## [2 Create container registry](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-acr)

Since you've created the image using your local Docker engine, you must log in to an Azure Container Registry through the Docker CLI. Docker must be installed on your machine. Once done, use 'docker logout <registry url>' to
        log out.
```bash
# Create an Azure container registry
az acr create --resource-group $resourceGroup --name $name --sku Basic

# Login to the registry
az acr login --name "...containerregistry"
```
To push a container image to a private registry like Azure Container Registry, you must first tag the image with the full name of the registry's login server.

Get full login server name, you're going to need it to tag your image locally.
```bash
az acr show --resource-group $resourceGroup --name $name --query loginServer --output table
# mycontainerregistry082.azurecr.io
```

Now tag the image locally

```bash
# tag the image locally
docker images
docker tag aci-tutorial-app ...containerregistry.azurecr.io/aci-tutorial-app:v1
docker images
```
Finally, you'll want to push the image to your Azure Container Registry.

```bash
# Push the image
docker push ...containerregistry.azurecr.io/aci-tutorial-app:v1
```
List the images in ACI.

```bash
az acr repository list --resource-group $resourceGroup --name $name  --output table

# Show tags
az acr repository show-tags --resource-group $resourceGroup --name $name  --repository aci-tutorial-app --output table
```
At this point you'll be able to see the image and its tag in the Azure Portal in your Azure Container Registry.


## [3 Deploy application](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-deploy-app)

```PowerShell
az container show --resource-group $resourceGroup --name aci-tutorial-app --query instanceView.state
```

## Clean Up

```bash
az group delete --name $resourceGroup --no-wait --yes
```

# References

- [Docker Docs](https://docs.docker.com/get-started/overview/)