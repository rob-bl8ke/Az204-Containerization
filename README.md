# Azure Containerization Study (focus on AZ-204)

- [Official Documentation - Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-overview)
- [Administer containers in Azure](https://learn.microsoft.com/en-us/training/paths/administer-containers-in-azure/) - Not specifically for AZ-204 course but worth following.

# Practicals

### Full Labs
- [Lab 05: Deploy compute workloads by using images and containers](https://microsoftlearning.github.io/AZ-204-DevelopingSolutionsforMicrosoftAzure/Instructions/Labs/AZ-204_lab_05.html) - This lab has 3 exercizes running through the basics of all three containerization topics including the creation of a simple Docker file. Always worth coming back to as a refresher.

#### Azure Container Instances
- [Exercise - Deploy a container instance by using the Azure CLI](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/3-run-azure-container-instances-cloud-shell) - on the Learning Path
    - [Quickstart: Deploy a container instance in Azure using the Azure portal](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-portal)
    - [Quickstart: Deploy a container instance in Azure using Azure PowerShell](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-powershell) - Uses the `Az` PowerShell module. `iis:nanoserver` image used in this lab is no longer supported. Rather use the `mcr.microsoft.com/azuredocs/aci-helloworld` image until the documentation is updated.
    - [Tutorial: Deploy a multi-container group using a YAML file](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-multi-container-yaml)
    - [Tutorial: Deploy a multi-container group using a Resource Manager template](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-multi-container-group) - Essentially the exact sames tutorial steps as the YAML file example. However, ARM templates can hold variables and parameters (demonstrated) and is more powerful.
    - [Quickstart: Deploy a container instance in Azure using an ARM template](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-template)
    - [Quickstart: Deploy a container instance in Azure using Bicep](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-bicep?tabs=CLI)
- [Exercise - Set environment variables in container instances](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/5-set-environment-variables-azure-container-instances) - on the Learning Path
    - [Set environment variables in container instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-environment-variables) - Microsoft Learn
- [Exercise - Mount an Azure file share in Azure Container Instances](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/6-mount-azure-file-share-azure-container-instances) - on the Learning Path.
    - [Mount an Azure file share in Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-volume-azure-files) - Microsoft Learn
- [Mount an emptyDir volume in Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-volume-emptydir)
- [Mount a gitRepo volume in Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-volume-gitrepo)
- [Mount a secret volume in Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-volume-secret)

#### Azure Container Registry
- [Exercise: Build and run a container image by using Azure Container Registry Tasks](https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/6-build-run-image-azure-container-registry)
- [Quickstart: Create a private container registry using the Azure CLI](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-azure-cli)
    - [Deploy to Azure Container Instances from Azure Container Registry using a service principal](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-using-azure-container-registry)

#### Azure Container Apps
- [Quickstart: Deploy your first container app with containerapp up](https://learn.microsoft.com/en-us/azure/container-apps/get-started?tabs%3Dbash)
- [Quickstart: Deploy to Azure Container Apps using Visual Studio Code](https://learn.microsoft.com/en-us/azure/container-apps/deploy-visual-studio-code?source%3Drecommendations)
- [Quickstart: Build and deploy your container app from a repository in Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/quickstart-code-to-cloud?tabs%3Dbash%2Ccsharp%26pivots%3Dgithub-build) and then do [Tutorial: Communication between microservices in Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/communicate-between-microservices?source%3Drecommendations%26tabs%3Dbash%26pivots%3Dacr-remote)
- [Tutorial: Deploy a Dapr application with GitHub Actions for Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/dapr-github-actions?source%3Drecommendations%26tabs%3Dazure-cli)

**Others**


### Videos

# References
- [Fantastic resource that covers much of Azure](https://github.com/Huachao/azure-content)
- [Use Environment Variables with PowerShell](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables?view=powershell-7.3)
- [OCI Image Format Specification](https://github.com/opencontainers/distribution-spec?tab=readme-ov-file) and its [spec](https://github.com/opencontainers/image-spec/blob/main/spec.md)
- [Helm Charts](https://helm.sh/) - Package manager for Kubernetes.

# Useful Visual Studio Extensions

- [Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)
- [Azure Resources](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureresourcegroups)
- [PowerShell](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell)
- [PlantUML](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml)
- [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) - Language support (Red Hat)
- [Bicep](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep) - Language support (Microsoft)
