
# Useful Azure CLI Snippets

```bash
# Create a random key
myKeyVault=...keyvlttest-$RANDOM

az login

# List resource groups that contain...
az group list --query "[?contains(name, '...')].{ name: name, id: id }"

myLocation=eastus
az group create --name az204-vault-rg34 --location $myLocation

# Don't wait, don't prompt...
az group delete --name az204-vault-rg34 --no-wait --yes

# # Get details for the current subscription
# az account list -o table # or
# az account show #note the "id" is the subscription id
# az account set --subscription a340a8e0-18f2-4355-bf29-beaad05b582e # try and switch account
```

To deploy an ARM template with [Azure CLI look here](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-cli).

Get help with the [Azure Command-Line Interface](https://learn.microsoft.com/en-us/cli/azure/) (CLI) documentation

For help with usage of JMESPath look [here](https://jmespath.org/examples.html) and [here](https://learn.microsoft.com/en-us/cli/azure/query-azure-cli?tabs=concepts%2Cbash).

Use `Get-Command -AzResourceGroup` to find the commands for a specific item you might know something about. If we’re working with functions so a good strategy would be to `Get-Command New-AzFunc`.

To look at a file `cat ./javascriptfunc/index.js`.

### Locations

`az account list-locations` will give you a list of locations you can choose from. Try this one to be less verbose: `az account list-locations --query "[].name"` or even this one `az account list-locations --query "[].{ name: name, displayName: displayName }"`.

## Environment Variables

- [How to set and use Environment Variables in a Bash script](https://ioflood.com/blog/bash-environment-variables/#:~:text=To%20set%20a%20bash%20environment,Output%3A%20%23%20Hello%2C%20World!)
- Using Bash, you can simply fetch an environment variable by refering to it. If its there, the value will be displayed.

```bash
echo $QikWebAppContainerPrefix
```

## Bash specific

Deleting a resource group if it exists (using Bash)

```bash
#!/bin/bash

# Set the name of the resource group you want to delete
resourceGroup="..."

# List tables
# az group list -o Table

# Check if the resource group exists
if [ $(az group exists --name $resourceGroup) = true ]; then
  # If the resource group exists, delete it
  echo "Deleting resource group $resourceGroup ..."
  az group delete --name $resourceGroup --yes
  echo "Resource group $resourceGroup deleted."
else
  # If the resource group doesn't exist, display a message
  echo "Resource group $resourceGroup does not exist."
fi
```