# Use PowerShell with Service Principal

Usually one can use [Add-AzureRmAccount](https://docs.microsoft.com/en-us/powershell/module/azurerm.profile/add-azurermaccount) to login to PowerShell, but this must be done every time a new PS instance is started and **the user needs to enter the password**. Some company requires a two-factor authentication, like smart card or phone call. This is not suitable for an automatated execution, like Task Scheduler. 

We can use **Service Principal** to automate this. 

## Create Service Principal in Azure Portal

In Portal Azure Portal, [create a Service Principal](Service-Principal-portal.md). 

## In PowerShell, sign in with the Service Principal

https://docs.microsoft.com/en-us/powershell/azure/authenticate-azureps?view=azurermps-6.7.0#sign-in-with-a-service-principal

```powershell
$user = "<Application ID>"
$password = ConvertTo-SecureString -String "<key>" -AsPlainText -Force
$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $user, $password
Connect-AzureRmAccount -ServicePrincipal -Credential $Credential -Tenant "72f988bf-86f1-41af-91ab-2d7cd011db47" 
```

Put **Application ID**, **Key** and **Tenant ID** in the PowerShell script.

## References

[Get-Credential](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-credential?view=powershell-6) shows how to create `Credential` without prompting the user with `New-Object`.

