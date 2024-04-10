# Build a container registry

- [Build and run a container registry](https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/6-build-run-image-azure-container-registry)
- [Building and Running Container Images with ACR, ACI and the Azure CLI](https://markheath.net/post/build-container-images-with-acr)

```bash
az group create --name ...resourcegroup --location eastus
az acr create --resource-group ...resourcegroup --name ...containerregistry --sku Basic

cd /c/code/azure
mkdir container-docker-file
cd container-docker-file/

# Create a simple docker file
echo FROM mcr.microsoft.com/hello-world > Dockerfile
cat Dockerfile

az acr build --image sample/hello-world:v1 --registry ...containerregistry --file Dockerfile .

az acr repository list --name ...containerregistry --output table

az acr repository show-tags --name ...containerregistry --repository sample/hello-world --output table

# to get what you need for --cmd
az acr list

# example uses $Registry to specify the registry where you run the command
# When your using /dev/null it's a place where output goes to die. With the command > /dev/null it tells the program to direct any output to this so it doesn't come up on the screen
# Can't seem to get this to work...
az acr run --registry ...containerregistry --cmd '...containerregistry.azurecr.io/sample/hello-world:v1' /dev/null
```


- [Exercize - Deploy a container instance by using Azure CLI](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/3-run-azure-container-instances-cloud-shell)

# Deploy a container instance
### Legacy

This specific example is [sourced from this video location](https://youtu.be/qMzYRyHKydA?t=1096). It uses `docker compose` and is being retired. Show for information purposes only...
```bash
docker login azure
# Must create a context to link docker compose to azure
docker context create aci ...acicontext
docker context use ...acicontext
# Now can use docker compose
docker compose up
```

### Relevant
- [Deploy a container instance by using the Azure CLI](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/3-run-azure-container-instances-cloud-shell)

```bash
az group create --name ...resourcegroup --location eastus
DNS_NAME_LABEL=...coolwebapp

az container create --resource-group ...resourcegroup --name $DNS_NAME_LABEL --image mcr.microsoft.com/azuredocs/aci-helloworld --ports 80 --dns-name-label $DNS_NAME_LABEL --location eastus

az container show --resource-group ...resourcegroup --name $DNS_NAME_LABEL --query "{FQDN:ipAddress.fqdn, ProvisioningState:provisioningState}" --out table

curl ...coolwebapp.eastus.azurecontainer.io

az group delete --name ...resourcegroup --no-wait
```

# Build and Deploy Using Docker
- [Source](https://youtu.be/qMzYRyHKydA?t=976)
- [Node Sample Web App available here](https://github.com/Azure-Samples/acr-build-helloworld-node)
- Go to Settings -> Access keys. You'll see the login server and when Admin user is checked, the Username and two possible passwords will be shown.

```bash
# Build the image
cd /c/Code/azure/acr/acr-build-helloworld-node
docker build -t ...containerregistry.azurecr.io/acr-helloworld .

# Run it locally to test
docker run -d -p 8080:80 ...containerregistry.azurecr.io/acr-helloworld

# Login to your ACR.
# More secure to use --password-stdin
docker login ...containerregistry.azurecr.io --username ...containerregistry --password CLBBAuBy7e/RXL/J2NbHaBdeH4bzg...

# Push image up to the registry
docker push ...containerregistry.azurecr.io/acr-helloworld
```

# Environment Variables and Restart Values

Also see scripts folder...

- [Run containerized tasks with restart policies](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/4-run-containerized-tasks-restart-policies)
- [Set environment variables in container instances](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/5-set-environment-variables-azure-container-instances)

```bash
#!/bin/bash

az group create --name ...resourcegroup --location eastus

az container create \
    --resource-group ...resourcegroup \
    --file env.yaml \

az container show --resource-group ...resourcegroup \
    --name $DNS_NAME_LABEL \
    --query "{FQDN:ipAddress.fqdn, ProvisioningState:provisioningState}" \
    --out table

curl ...coolwebapp.eastus.azurecontainer.io

# az group delete --name ...resourcegroup --no-wait
```

- [Mounting file shares](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/6-mount-azure-file-share-azure-container-instances)
- [Deploy a container app](https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/3-exercise-deploy-app)


# Container Group

```
az container create --resource-group demo --file deployment.yml
```

# Build and Deploy a Container Instance and Container App Environment (Lab 5)

- [What is the difference between Container Apps and Container Instances?](https://www.youtube.com/watch?v=YU-MKS0rmTE)

```PowerShell
dotnet new console --output . --name ipcheck --framework net6.0

Get-Content ipcheck.csproj

New-Item Dockerfile
Get-Content Dockerfile

code .

dotnet run
```


```bash
#!/bin/bash

registryName=conregistry$RANDOM

# By redirecting the output of the check-name command to /dev/null, 
# any output produced by the command will be discarded and not displayed in the terminal.
if az acr check-name --name "$registryName" > /dev/null; then

    # Create and show
    az acr create --resource-group ContainerCompute --name $registryName --sku Basic
    # az acr list
    # az acr list --query "max_by([], &creationDate).name" --output tsv

    acrName=$(az acr list --query "max_by([], &creationDate).name" --output tsv)

    echo $acrName

    az acr build --registry $acrName --image ipcheck:latest .


else
    echo "The registry name is NOT available"
fi
```

### Create App Container Environment

```bash
az extension add --name containerapp --upgrade

# Note: Azure Container Apps resources have migrated from the Microsoft.Web namespace to the Microsoft.App namespace.
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights

# An environment in Azure Container Apps creates a secure boundary around a group of container apps. Container Apps 
# deployed to the same environment are deployed in the same virtual network and write logs to the 
# same Log Analytics workspace.
myRG=ContainerCompute
myAppConEnv=az204-env-$RANDOM

az containerapp env create --name $myAppConEnv --resource-group $myRG --location eastus

# By setting --ingress to external, you make the container app available to public requests. The command returns a link to access your app.
az containerapp create \
    --name my-container-app \
    --resource-group $myRG \
    --environment $myAppConEnv \
    --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
    --target-port 80 \
    --ingress 'external' \
    --query properties.configuration.ingress.fqdn

# Fetch the result from the terminal and curl it
# curl "result"
```

# Very Simple Docker File

`Dockerfile`
```
FROM mcr.microsoft.com/hello-world

```