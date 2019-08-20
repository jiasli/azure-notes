# Invoke Azure REST API with PowerShell

Sometimes an Azure REST API may not have corresponding PowerShell CmdLet. Instead, we can get the AAD token and directly invoke Azure REST API in PowerShell.

This below PowerShell script uses Service Principal to acquire token. For other ways to acquire token, see [Invoke Azure REST API with curl](rest-curl.md).

```powershell
# Get access token. https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal

$TenantId = "72f988bf-86f1-41af-91ab-2d7cd011db47"  # aka Directory ID. This value is Microsoft tenant ID
$ClientId = ""  # aka Application ID
$ClientSecret = ""  # aka key

$Resource = "https://management.core.windows.net/"
$RequestAccessTokenUri = "https://login.microsoftonline.com/$TenantId/oauth2/token"

# https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow#request-an-access-token
$Body = "grant_type=client_credentials&client_id=$ClientId&client_secret=$ClientSecret&resource=$Resource"
$Token = Invoke-RestMethod -Method Post -Uri $RequestAccessTokenUri -Body $Body -ContentType 'application/x-www-form-urlencoded'

# Build body with key-value pair
#$Body = @{'resource'= $Resource
#   'client_id' = $ClientId
#   'grant_type' = 'client_credentials'
#   'client_secret' = $ClientSecret
#}

Write-Host "Access Token JSON" -ForegroundColor Green
Write-Output $Token

# Show expiration date. https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-object
# $token | select *, @{Name='Expires';Expression={[timezone]::CurrentTimeZone.ToLocalTime(([datetime]'1/1/1970').AddSeconds($_.expires_on))}} | fl *


# Invoke REST API

$Headers = @{"Authorization" = "$($Token.token_type) "+ "$($Token.access_token)"}

$RunbookId = "/subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.Automation/automationAccounts/xxx/runbooks/xxx"
$ScheduleId = "/subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.Automation/automationAccounts/xxx/schedules/xxx"

# Link Runbook to Schedule
$Url = "https://s2.automation.ext.azure.com/api/Orchestrator/LinkRunbookToSchedule?runbookId=$RunbookId&scheduleId=$ScheduleId&runOn="
$Response = Invoke-WebRequest -Method Post -Uri $Url -Headers $Headers
Write-Output $Response.StatusCode

# Unlink Runbook from Schedule
$Url = "https://s2.automation.ext.azure.com/api/Orchestrator/UnlinkRunbookFromSchedule?runbookId=$RunbookId&scheduleId=$ScheduleId"
$Response = Invoke-WebRequest -Method Post -Uri $Url -Headers $Headers
Write-Output $Response.StatusCode
```