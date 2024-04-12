# Container Instances

## Deploy a multi-container group using a YAML file

### Overview

- Configure a YAML file.
- Create a resource group.
- Create and deploy a container group.

A container group is useful when building an application sidecar for logging, monitoring, or any other configuration where a service needs a second attached process.

In this example, the sidecar periodically makes an HTTP request to the main web application via the group's local network to ensure that it is running. This sidecar example could be expanded to trigger an alert if it received an HTTP response code other than 200 OK.

- The docker images used for the following images are hosted here:
    - [ACI Hello World](https://hub.docker.com/_/microsoft-azuredocs-aci-helloworld)
    - [ACI Sidecar Image](https://hub.docker.com/_/microsoft-azuredocs-aci-tutorial-sidecar)

### Code

```yaml
apiVersion: 2019-12-01
location: eastus
name: container-group
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
      - port: 8080
  - name: aci-tutorial-sidecar
    properties:
      image: mcr.microsoft.com/azuredocs/aci-tutorial-sidecar
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: 80
    - protocol: tcp
      port: 8080
tags: {exampleTag: tutorial}
type: Microsoft.ContainerInstance/containerGroups
```
- To use a private container registry provide these details at the root level...
```yaml
apiVersion: 2019-12-01
location: eastus
name: container-group
...
imageRegistryCredentials:
  - server: imageRegistryLoginServer
    username: imageRegistryUsername
    password: imageRegistryPassword
```
- Note that the following can be established by looking at an existing resource. For instance if you `az group show --name $resourceGroup` you'll note that the "type" is displayed as one of the resource's properties.

```yaml
  type: Microsoft.ContainerInstance/containerGroups
```
- Use `--file .\_Sandbox\deploy-aci.yaml` to create your resource with the YAML file.
- The script has less lines of code because the configuration is in the file.

```PowerShell
$containerGroup = "container-group"

az group create --name $resourceGroup --location $location
az container create --resource-group $resourceGroup --file .\_Sandbox\deploy-aci.yaml

# Navigate to your resource in the browser using its IP address.
az container show --resource-group $resourceGroup `
    --name $containerGroup `
    --query "ipAddress.ip"

# or 
az container show --resource-group $resourceGroup --name $containerGroup --output table
```

#### Viewing the container logs

```PowerShell
az container logs --resource-group myResourceGroup --name $containerGroup --container-name aci-tutorial-app
az container logs --resource-group myResourceGroup --name $containerGroup --container-name aci-tutorial-sidecar
```

# References
- [Example tutorial](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-multi-container-yaml)
- [Container groups in Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-container-groups)
- [Docker Docs](https://docs.docker.com/get-started/overview/)
- [Your YAML syntax primer notes](https://www.notion.so/YAML-468ccca339294f94b771bbdc981ab3c4) from the [video sourced from here](https://www.youtube.com/watch?v=1uFVr15xDGg).