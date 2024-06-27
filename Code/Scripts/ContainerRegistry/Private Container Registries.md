
# Private Container Registries
- [Authenticate with Azure Container Registry from Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-auth-aci). Also look here to see how you can use an existing service principal.

The following scripts show how you can use a Microsoft Entra service principal to provide access to your private container registries in Azure Container Registry.
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
