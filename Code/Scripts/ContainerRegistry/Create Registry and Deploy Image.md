# Create a Container Registry and deploy a Container Image

This is a series of tutorials based on the above header. However there are lots more tutorials here and you should also be looking at "Deploy a multi-container group".

Follow the three part series from here:

- [1 Create container image](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-app)
- [2 Create container registry](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-acr)
- [3 Deploy application](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-deploy-app)

## [1 Create container image](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-app)

```bash
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

```bash
name=...containerregistry
resourceGroup=...resourcegroup
container=...container
location=eastus

# Create resource group
az group create --name $resourceGroup --location $location
```

```bash
# Create an Azure container registry
az acr create --resource-group $resourceGroup --name $name --sku Basic

# Login to the registry
az acr login --name "...containerregistry"
```
To push a container image to a private registry like Azure Container Registry, you must first tag the image with the full name of the registry's login server.

```bash
# get full login server name
az acr show --resource-group $resourceGroup --name $name --query loginServer --output table
# mycontainerregistry082.azurecr.io

# tag the image locally
docker images
docker tag aci-tutorial-app ...containerregistry.azurecr.io/aci-tutorial-app:v1
docker images

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

- [Authenticate with Azure Container Registry from Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-auth-aci). Also look here to see how you can use an existing service principal.

```bash
#!/bin/bash
servicePrincipal=...serviceprincipal

# Obtain the full registry ID
registryId=$(az acr show --name $name --resource-group $resourceGroup  --query "id" --output tsv)

# IMPORTANT!!! You will need to remove the forward slash from the $registryId otherwise the service principal creation statement won't work!!!
echo $registryId

# Create the service principal with rights scoped to the registry.
# Default permissions are for docker pull access. Modify the '--role'
# argument value as desired:
# acrpull:     pull only
# acrpush:     push and pull
# owner:       push, pull, and assign roles
password=$(az ad sp create-for-rbac --name $servicePrincipal --scopes $registryId --role acrpull --query "password" --output tsv)
userName=$(az ad sp list --display-name $servicePrincipal --query "[].appId" --output tsv)

# Output the service principal's credentials; use these in your services and
# applications to authenticate to the container registry.
echo "Service principal ID: $userName"
echo "Service principal password: $password"
```
Now with the service principal created, move on to 


```bash
az container create --resource-group $resourceGroup \
    --name aci-tutorial-app \
    --image ...containerregistry.azurecr.io/aci-tutorial-app:v1 \
    --registry-login-server ...containerregistry.azurecr.io \
    --registry-username $userName \
    --registry-password $password \
    --cpu 1 --memory 1 \
    --ip-address Public --dns-name-label ...dnsname --ports 80

az container show --resource-group $resourceGroup --name aci-tutorial-app --query instanceView.state
az container show --resource-group $resourceGroup --name aci-tutorial-app --query ipAddress.fqdn
```

To see the running application, navigate to the displayed DNS name in your favorite browser. Then view the logs.

```bash
az container logs --resource-group $resourceGroup --name aci-tutorial-app
```

(TO BE CONTINUED)


## Clean Up

```bash
az group delete --name $resourceGroup --no-wait --yes
```

# References

- [Docker Docs](https://docs.docker.com/get-started/overview/)