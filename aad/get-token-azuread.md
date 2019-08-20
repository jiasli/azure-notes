# Get AAD Token in PowerShell with AzureAD Module

We can get an AAD access token for REST API calls using [AzureAD Module](https://docs.microsoft.com/en-us/powershell/module/azuread).

```powershell
$TenantId = "72f988bf-86f1-41af-91ab-2d7cd011db47"  # aka Directory ID. This value is Microsoft tenant ID
$ClientId = ""  # aka Application ID
$ClientSecret = ""  # aka key

#Install-Module AzureAD -Force
Import-Module -Name "AzureAD"
$AadModule = Get-Module -Name "AzureAD" -ListAvailable
$adal = Join-Path $AadModule.ModuleBase "Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
[System.Reflection.Assembly]::LoadFrom($adal) | Out-Null

$authority = "https://login.windows.net/$TenantId"
$authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
$AdUserCred = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential" -ArgumentList $ClientId, $ClientSecret
$token = ($authContext.AcquireTokenAsync("https://management.azure.com/", $AdUserCred)).Result.AccessToken
Write-Output $token
$authHeader = @{
    "Content-Type" = "application/json"
    "Authorization"= "Bearer " + $token
    }
```

This corresponds to C# code

```csharp
// Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory

// https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/AuthenticationContext-the-connection-to-Azure-AD#authority-validation
var context = new Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext("https://login.microsoftonline.com/<tenantId>");
var credential = new ClientCredential("<clientId>", "<clientSecret>");
// https://management.azure.com/ may not work for some API
var token = context.AcquireTokenAsync("https://management.core.windows.net/", credential).Result.AccessToken;
```