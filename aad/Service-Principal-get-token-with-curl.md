# Get Service Principal token with curl

This article shows how to get Service Principal token with curl, following doc [Request an Access Token](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-service-to-service#request-an-access-token).

We need three important parameters. (Yes, they all have two names.) For example, one of my App has


Name in Azure Portal | Name in doc | Value
--- | --- | ---
Directory ID   | Tenant ID     | 72f988bf-86f1-41af-91ab-2d7cd011db47
Application ID | Client ID     | b0e35524-7612-42bd-ae7b-d7e74accb8cc
Key            | Client Secret | kY/0Ba48qUoC29ClLL+JMt0NdHiDPmI2naS6cZfxxxx=


## Create Service Principal in Azure Portal

In Portal Azure Portal, [create a Service Principal and assign it permissions](Service-Principal-portal.md). 

## Get Token with curl

**Key needs to be URL encoded**, such as `kY%2F0Ba48qUoC29ClLL%20JMt0NdHiDPmI2naS6cZfxxxx%3D`. Notepad++ plugin MIME Tools doesn't encode +, which perhaps is a bug. You may use https://meyerweb.com/eric/tools/dencoder/.

```Batchfile
curl https://login.microsoftonline.com/<tenant-ID>/oauth2/token -H "Content-Type: application/x-www-form-urlencoded" --data "grant_type=client_credentials&client_id=<Client ID>&client_secret=<Client Secret>=https%3A%2F%2Fmanagement.core.windows.net%2F"
```

To give it a prettier look (line break is added only for demonstration):


```Batchfile
curl https://login.microsoftonline.com/<tenant-ID>/oauth2/token 
	-H "Content-Type: application/x-www-form-urlencoded" 
	--data
	"
		grant_type=client_credentials&
		client_id=<Client ID>&
		client_secret=<Client Secret>&
		resource=https%3A%2F%2Fmanagement.core.windows.net%2F
	"
```

Replace `<tenant ID>`, `<client_id>`, `<client_secret>` with above values. So a real command is like

```Batchfile
curl https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db47/oauth2/token -H "Content-Type: application/x-www-form-urlencoded" --data "grant_type=client_credentials&client_id=b0e35524-7612-42bd-ae7b-d7e74accb8cc&client_secret=kY%2F0Ba48qUoC29ClLL%20JMt0NdHiDPmI2naS6cZfxxxx%3D&resource=https%3A%2F%2Fmanagement.core.windows.net%2F" 
```

Instead of using **Tenant ID** in URL, we can also use **Tenant Name** in URL, such as `Microsoft.onmicrosoft.com`.

```Batchfile
curl https://login.microsoftonline.com/Microsoft.onmicrosoft.com/oauth2/token -H "Content-Type: application/x-www-form-urlencoded" --data "grant_type=client_credentials&client_id=b0e35524-7612-42bd-ae7b-d7e74accb8cc&client_secret=kY%2F0Ba48qUoC29ClLL%20JMt0NdHiDPmI2naS6cZfxxxx%3D&resource=https%3A%2F%2Fmanagement.core.windows.net%2F" 
```

## Response


```json
{
  "token_type": "Bearer",
  "expires_in": "3599",
  "ext_expires_in": "0",
  "expires_on": "1519962650",
  "not_before": "1519958750",
  "resource": "https://management.core.windows.net/",
  "access_token": "eyJ0eX..."
}
```

`access_token` is what we want. We may decode it with https://jwt.io/ to see more info in the token.


Then the token can be used in HTTP **Authorization** header in any REST API call, such as [Invoke Azure REST API with curl](REST-curl.md).

```
Authorization:Bearer eyJ0eX...
```


## References

Client credentials grant: https://docs.microsoft.com/en-us/rest/api/#client-credentials-grant-non-interactive-clients

Request an Access Token: https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-oauth-service-to-service#request-an-access-token

curl download: https://curl.haxx.se/download.html

curl manpage: https://curl.haxx.se/docs/manpage.html