# Useful Powershell Snippets

Get help with [Azure PowerShell Documentation](https://learn.microsoft.com/en-us/powershell/azure/?view=azps-9.6.0) v9.6.0

Reference to Azure Functions Core Tools [at the command-line](https://learn.microsoft.com/en-us/azure/azure-functions/functions-core-tools-reference?tabs=v2). Here are some [full script examples](https://learn.microsoft.com/en-us/azure/azure-functions/create-resources-azure-powershell).

Use `Get-Command -AzResourceGroup` to find the commands for a specific item you might know something about. If we’re working with virtual machines so a good strategy would be to `Get-Command *-WebApp`.

```powershell
Connect-AzAccount
Disconnect-AzAccount

Get-Content csharpfunc.cs

# Create resource group and required standard SKU storage account
New-AzResourceGroup -Name ...resourcegroup -Location southafricanorth
Get-AzResource -ResourceGroupName ...resourcegroup | Format-Table
Get-AzResource -Name ...funcappstorage -ResourceGroupName ...resourcegroup

Get-AzLocation | Format-Table Location, DisplayName
Get-AzLocation | Where-Object { $_.DisplayName -like "Canada" } | Format-Table Location, DisplayName

# Delete
Remove-AzResourceGroup -Name ...resourcegroup
```

```PowerShell
Get-Content .\typescriptfunc\index.ts
Get-AzResource -ResourceGroupName ...resourcegroup | Format-Table
Get-AzResource -Name ...funcappstorage -ResourceGroupName ...resourcegroup
```

- [PowerShell Documentation](https://learn.microsoft.com/en-us/powershell/azure/?view=azps-10.0.0&viewFallbackFrom=azps-9.6.0)

## Environment Variables

- [Use Environment Variables with PowerShell](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables?view=powershell-7.3)
- Access environment variables by prefixing the name with `$Env:`.

```powershell
# $resourceGroup="myresourcegroup"
# $location="eastus"
# $appContainerEnv="my-app-container-env"
# $imageTag="namespace/app:1.0.0"

$resourceGroup=$Env:QikResourceGroup
$location="eastus"
$appContainerEnv=$Env:QikContainerEnvironment
$webApp=$Env:QikWebAppContainerPrefix
$imageTag="namespace/app:1.0.0"

Write-Output  "Resource Group: $resourceGroup"
Write-Output  "Location: $location"
Write-Output  "Web App Container Prefix: $webApp"
Write-Output  "App Container Env: $appContainerEnv"
Write-Output  "Image Tag: $imageTag"
```