# Environment Variables and Secure Values

```powershell
az container create `
    --resource-group $resourceGroup --name $wordContainer `
    --image $image `
    --restart-policy Always `
    --environment-variables "NumWords=5 MinLength=8" `
    --secure-environment-variables "ConnectionString=... Password=..."

az container show --resource-group $resourceGroup `
    --name $wordContainer `
    --query "{EnvironmentVariables:containers[0].environmentVariables}" `
    --out yaml
```

```bash
#!/bin/bash

az group create --name ...resourcegroup --location eastus

az container create \
    --resource-group ...resourcegroup \
    --file env.yaml \

az container show --resource-group ...resourcegroup \
    --name ...securetest \
    --query "containers[0].environmentVariables"

curl ...coolwebapp.eastus.azurecontainer.io

# az group delete --name ...resourcegroup --no-wait
```

```yaml
apiVersion: 2018-10-01
location: eastus
name: ...securetest
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'
          value: 'my-exposed-value'
        - name: 'SECRET'
          secureValue: 'my-secret-value'
      image: nginx
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```