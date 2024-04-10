
- [Source Tutorial](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tutorial-quick-task)

```bash
az login

# Create resource group and required standard SKU storage account
az group create --name ...resourcegroup --location southafricanorth

# Create an Azure Container Registry
az acr create --name ...containerregistry --resource-group ...resourcegroup --sku Standard --admin-enabled

az acr show -g ...resourcegroup -n ...containerregistry


# In order to run this command successfully you need your Docker Daemon running on your local machine.
# You'll use the token later, so record it.
az acr login --name ...containerregistry --expose-token

# pull an image
docker pull mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine

# Run the container locally (remove it on stop)
docker run -it --rm -p 8080:80 mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine

# Create an alias of your image using your Azure Container Registry name
docker tag mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine ...containerregistry.azurecr.io/samples/nginx

# Use the token that you exposed earlier to login to your registry
docker login ...containerregistry.azurecr.io --username 00000000-0000-0000-0000-000000000000 --password eyJhbGciO...

# Push your image to your Container Registry
docker push ...containerregistry.azurecr.io/samples/nginx

# You can also at some point pull it if you don't have the local copy
docker pull ...containerregistry.azurecr.io/samples/nginx
```
