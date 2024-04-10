# Mount Volumes

```bash
#!/bin/bash

az group create --name ...resourcegroup --location eastus

az storage account create \
	--name ...funcappstorage \
	--location southafricanorth \
	--resource-group ...resourcegroup \
	--sku Standard_LRS \
	--allow-blob-public-access false

az storage share create --account-name ...funcappstorage --name ...filehare

KEY=$(az storage account keys list --account-name ...funcappstorage --resource-group ...resourcegroup --query "[0].value" --out table)

# Currently fails here... documentation here: https://learn.microsoft.com/en-us/azure/container-instances/container-instances-volume-azure-files
# and here: https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/6-mount-azure-file-share-azure-container-instances
az container create \
    --resource-group ...resourcegroup \
    --name hellofiles \
    --image mcr.microsoft.com/azuredocs/aci-hellofiles \
    --dns-name-label ...hellofiles \
    --ports 80 \
    --azure-file-volume-account-name ...funcappstorage \
    --azure-file-volume-account-key $KEY \
    --azure-file-volume-share-name ...filehare \
    --azure-file-volume-mount-path /aci/logs/

az container show --resource-group ...resourcegroup \
  --name hellofiles --query ipAddress.fqdn --output tsv

  # ANYHOW !!! THE PREFERRED METHOD IS SUPPOSED TO BE EITHER A YAML OR A ARM FILE.
```