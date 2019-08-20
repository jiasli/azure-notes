# Use Azure CLI with Service Principal

Usually we can use `az login` to login to Azure CLI. The token will be cached and refreshed for future uses. Some company requires a two-factor authentication, like smart card or phone call. This is not suitable for automatated executions, like Task Scheduler. We can use **Service Principal** to automate this. 

## Create Service Principal

https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac

```batch
az ad sp create-for-rbac
```
This automatically creates a Service Principal and assigns Contributor role on the scope of the subscription.
```batch
Creating a role assignment under the scope of "/subscriptions/00977cdb-163f-435f-9c32-39ec8ae61f4d"

{
  "appId": "318e1b5a-6997-40e2-b707-xxxxxxxxxxxx",     >> -u
  "displayName": "azure-cli-2019-08-14-12-19-29",
  "name": "http://azure-cli-2019-08-14-12-19-29",
  "password": "4682ac8f-3efa-430e-b409-xxxxxxxxxxxx",  >> -p
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"     >> -t
}
```

## Sign in with the Service Principal

Use `appId` for `-u`, `password` for `-p` and `tenant` for `-t`.

https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest#az-login

```batch
az login --service-principal -u "318e1b5a-6997-40e2-b707-xxxxxxxxxxxx" -p "4682ac8f-3efa-430e-b409-xxxxxxxxxxxx" --tenant "72f988bf-86f1-41af-91ab-2d7cd011db47"
```

Then call CLI commands as usual.