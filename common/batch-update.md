# Batch Update Azure Resources by calling Azure REST API with PowerShell

This sample script updates the `timeWindowInMinutes` properties of all `scheduledQueryRules` resources in a subscription, by invoking 3 APIs:

* GET: [Scheduled Query Rules - List By Subscription](https://docs.microsoft.com/en-us/rest/api/monitor/scheduledqueryrules/listbysubscription)
* PUT: [Scheduled Query Rules - Create Or Update](https://docs.microsoft.com/en-us/rest/api/monitor/scheduledqueryrules/createorupdate)
* PATCH: [Scheduled Query Rules - Update](https://docs.microsoft.com/en-us/rest/api/monitor/scheduledqueryrules/update)

```powershell
# Get Access Token from https://resources.azure.com/api/token?plaintext=true
$Token = "eyJ0eX..."
$Headers = @{"Authorization" = "Bearer $Token"}
$SubscriptionID = "<subID>"

# Invoke API: Scheduled Query Rules - List By Subscription
$Url = "https://management.azure.com/subscriptions/$SubscriptionID/providers/microsoft.insights/scheduledQueryRules?api-version=2018-04-16"
$Response = Invoke-WebRequest -Method GET -Uri $Url -Headers $Headers
$RuleArray = ($Response.Content | ConvertFrom-JSON).Value

# Invoke API: Scheduled Query Rules - Create Or Update
foreach ($rule in $RuleArray) {
    $id = $rule.id;    
    $Url = "https://management.azure.com$id" + "?api-version=2018-04-16"
    $rule.properties.schedule.timeWindowInMinutes = 60
    Write-Output "PUT $Url"
    $Response = Invoke-WebRequest -Method PUT -Uri $Url -Headers $Headers -ContentType "application/json" -Body ($rule | ConvertTo-Json -Depth 10) 
    Write-Output "$($Response.StatusCode) $($Response.StatusDescription)"
}

# === Alternatively ===
# Invoke API: Scheduled Query Rules - Update
# Though the doc says only properties.enabled is allowed in the body, changing other property also seems to work.
foreach ($rule in $RuleArray) {
    $id = $rule.id;
    $Url = "https://management.azure.com$id" + "?api-version=2018-04-16"
    Write-Output "PATCH $Url"
    $Response = Invoke-WebRequest -Method PATCH -Uri $Url -Headers $Headers -ContentType "application/json" -Body '{"properties": { "schedule": {"frequencyInMinutes": 5, "timeWindowInMinutes": 60}}}'
    Write-Output "$($Response.StatusCode) $($Response.StatusDescription)"
}
```
