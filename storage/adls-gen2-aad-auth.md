# Use Azure Data Lake Storage Gen2 with AAD auth and REST in Python

## Get access token

https://docs.microsoft.com/en-us/python/azure/python-sdk-azure-authenticate?view=azure-python

For example, in App Service, get system-managed identity token with

```py
import os
import requests

# Get access token with REST：https://docs.microsoft.com/en-us/azure/app-service/overview-managed-identity#rest-protocol-examples
# https://stackoverflow.com/questions/386934/how-to-evaluate-environment-variables-into-a-string-in-python
url = f'{os.path.expandvars("MSI_ENDPOINT")}?resource="https://storage.azure.com/&api-version=2017-09-01'

# http://docs.python-requests.org/en/master/user/quickstart/#custom-headers
headers = {'secret', os.path.expandvars("MSI_SECRET")}
response = requests.get(url, headers=headers)
access_token = response.json()["access_token"]
```

## Invoke ADLS Gen2 REST API

```py
import io

stream = io.BytesIO()

# Fill the stream

stream_length = stream.getbuffer().nbytes

# https://docs.microsoft.com/en-us/rest/api/storageservices/authenticate-with-azure-active-directory
headers = {
    'Authorization': 'Bearer ' + access_token,
    'x-ms-version': '2018-11-09'} # API Version: 2018-11-09  https://docs.microsoft.com/en-us/rest/api/storageservices/datalakestoragegen2/path/update

# https://docs.microsoft.com/en-us/rest/api/storageservices/datalakestoragegen2/path/update

url = f'https://{adls_name}.dfs.core.windows.net/{adls_fs}/{blob_name}?resource=file'
response = requests.put(url, headers=headers)

url = f'https://{adls_name}.dfs.core.windows.net/{adls_fs}/{adls_file}?action=append&position=0'
response = requests.patch(url, stream, headers=headers)

url = f'https://{adls_name}.dfs.core.windows.net/{adls_fs}/{blob_name}?action=flush&position={stream_length}'
response = requests.patch(url, headers=headers)
```
