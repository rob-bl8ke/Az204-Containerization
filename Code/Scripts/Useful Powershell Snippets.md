# Useful Powershell Snippets

[Install Azure PowerShell on Windows](https://learn.microsoft.com/en-us/powershell/azure/install-azps-windows?view=azps-11.5.0&tabs=powershell&pivots=windows-psgallery) - See also if you have `AzureRM` PowerShell module installed as there may be complications.

Determine your PowerShell version.
```ps
$PSVersionTable.PSVersion
```
Determine that you have the right Az module installed.
```ps
Get-Module -Name Az -ListAvailable
```

Update the module
```ps
Update-Module -Name Az -Force
```

Get help with [Azure PowerShell Documentation](https://learn.microsoft.com/en-us/powershell/azure/?view=azps-9.6.0) v9.6.0

## Some typical commands

Use `Get-Command -AzResourceGroup` to find the commands for a specific item you might know something about. If we’re working with virtual machines so a good strategy would be to `Get-Command *-WebApp`.

```powershell
Connect-AzAccount
Disconnect-AzAccount

Get-Content csharpfunc.cs

# Create resource group and required standard SKU storage account
New-AzResourceGroup -Name ...resourcegroup -Location southafricanorth
Get-AzResource -ResourceGroupName ...resourcegroup | Format-Table
Get-AzResource -Name ...funcappstorage -ResourceGroupName ...resourcegroup

# Search locations
Get-AzLocation | Format-Table Location, DisplayName
Get-AzLocation | Where-Object { $_.DisplayName -like "Canada" } | Format-Table Location, DisplayName
Get-AzLocation | Where-Object { $_.DisplayName -like "*africa*" } | Format-Table Location, DisplayName

# Delete
Remove-AzResourceGroup -Name ...resourcegroup
```

```PowerShell
Get-Content .\typescriptfunc\index.ts
Get-AzResource -ResourceGroupName ...resourcegroup | Format-Table
Get-AzResource -Name ...funcappstorage -ResourceGroupName ...resourcegroup
```

- [PowerShell Documentation](https://learn.microsoft.com/en-us/powershell/azure/?view=azps-10.0.0&viewFallbackFrom=azps-9.6.0)

## Environment variables

- [Use Environment Variables with PowerShell](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables?view=powershell-7.3)
- Access environment variables by prefixing the name with `$Env:`.
- Create a user environment variable with a key of `AzureScriptSettings`.
- Add your custom settings like this
```
prefix|resourcegroup|location
```

- Access these settings from your environment settings instead of displaying them in the script.

```powershell

$settings = $Env:AzureScriptSettings
$arr
$arr = $settings.Split("|")
$prefix = $arr[0]

$resourceGroup = $arr[1]
$location = $arr[2]
$containerName = "$prefix-container-1"
$dnsNameLabel = "$prefix-aci-example"
$image =  "mcr.microsoft.com/azuredocs/aci-wordcount:latest"

Write-Output  "Resource Group: $resourceGroup"
Write-Output  "Location: $location"
Write-Output  "Web App Container Name: $containerName"
Write-Output  "DNS Name Label: $dnsNameLabel"
Write-Output  "Image: $image"

```

# Check for...

This returns a JSON response with high level tenant and subscription information.

```powershell
$tenantInfo = (az login)
```
### Resource Group

Create a resource group  if it doesn't exist.

```powershell
if ((az group exists --name $resourceGroup)) {
    Write-Host "Group $resourceGroup exists in $($tenantInfo.name)"
} else {
    Write-Host "Group DOES NOT exist... creating"
    az group create --name $resourceGroup --location eastus
}
```
