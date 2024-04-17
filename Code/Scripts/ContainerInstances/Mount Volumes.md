# Azure CLI (PowerShell)

- [Source](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-volume-azure-files#create-an-azure-file-share)

Note how similar creating a file share is to creating a blob storage container... the commands are very similar. For a blob storage container it would be `az storage container create` with very similar properties.

However, when creating a file share it is `az storage share create`.

```PowerShell

$settings = $Env:AzureScriptSettings
$arr = $settings.Split("|")
$prefix = $arr[0]

$resourceGroup = $arr[1]
$location = $arr[2]

$storageAccount = "$($prefix)storageacc"
$fileShare = "$prefix-file-share"

# $containerName = "$prefix-container-1"
# $dnsNameLabel = "$prefix-aci-example"
$image =  "mcr.microsoft.com/azuredocs/aci-hellofiles"
$dnsNameLabel = "$($prefix)-aci-hellofiles"

az login

# Create a general purpose storage account and create a file share in that account
az group create --name $resourceGroup --location $location

az storage account create `
    --name $storageAccount `
    --location $location `
    --resource-group $resourceGroup `
    --sku Standard_LRS `
    --allow-blob-public-access false

az storage share create --account-name $storageAccount --name $fileShare

# Fetch the account access key
$storageAccessKey = (az storage account keys list --account-name $storageAccount --resource-group $resourceGroup --query "[0].value" --out tsv)

Write-Host $storageAccessKey

az container create `
    --resource-group $resourceGroup `
    --name hellofiles `
    --image $image `
    --dns-name-label $dnsNameLabel `
    --ports 80 `
    --azure-file-volume-account-name $storageAccount `
    --azure-file-volume-account-key $storageAccessKey `
    --azure-file-volume-share-name $fileShare `
    --azure-file-volume-mount-path /aci/logs/


    az container show --resource-group $resourceGroup --name hellofiles --query ipAddress.fqdn --output tsv
```

# Azure CLI (using Bash)

> Note there is no reason why this can't work since it works with the PowerShell script above... Redo this and it should work.

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
```

# A note on mounting volumes on Windows

This is rather unrelated but worth investigating because of the issues that surround mounting a Windows file system into a Linux container using a local Docker for Windows engine.

- [Source](https://stackoverflow.com/questions/60916317/using-docker-for-windows-to-volume-mount-a-windows-drive-into-a-linux-container)
- [Another Source](https://forums.docker.com/t/mounted-windows-share-doesnt-show-contents-within-docker-container/100343)

If you're just trying to mount a windows path to a Linux based container, here's an example using the basic docker run command, and a Docker Compose example as well:

```docker
docker run -d --name qbittorrent -v "/mnt/f/Fetched Media/Unsorted:/downloads" -v "/mnt/f/Fetched Media/Blackhole:/blackhole" linuxserver/qbittorrent
```

- This example shares the `f:\Fetched Media\Unsorted` and `f:\Fetched Media\Blackhole` folders on the Windows host to the container; and within the Linux container you'd see the files from those Windows folders in their respective Linux paths shown to the right of the colon(s). The f:\Fetched Media\Unsorted folder will be in the /downloads folder in the Linux container.
- Remember to use double quotes (to avoid issues with special characters like a dot in the case of a hidden folder).

> Unless you're using WSL/WSL 2... make sure you've shared those Windows folders within the Docker Desktop settings area in the GUI.

Here's a Docker Compose version of the above example, for the sake of thoroughness (includes how to set a path as read-write (rw), or read-only (ro)):

```yaml
qbittorrent:
  image: 'linuxserver/qbittorrent:latest'
  volumes:
    - '/mnt/f/Fetched Media/Unsorted:/downloads:rw'
    - '/mnt/f/Fetched Media/Blackhole:/blackhole:rw'
    - '/mnt/e/Logs/qbittorrent:/config/logs:rw'
    - '/opt/some-local-folder/you-want/read-only:/some-folder-inside-container:ro'
```
