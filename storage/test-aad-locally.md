# Test AAD auth with Azure Storage locally

Create Managed identities and assign roles: <https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-msi?toc=%2fazure%2fstorage%2fblobs%2ftoc.json>

Acquire an access token: <https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token>

Note: Without connectionString, when running locally, `AzureServiceTokenProvider` will first try to get token 
from 169.254.169.25 and timeout. This step is unnecessary and slows down the program. This issue is discussed in <https://github.com/Azure/azure-sdk-for-net/issues/4645>. To solve it, add [connectionString](https://docs.microsoft.com/en-us/azure/key-vault/service-to-service-authentication#connection-string-support
) to force getting token from local. But make sure to remove it when deploying to Azure.

`AzureServiceTokenProvider` doesn't work with user-assigned managed identities. To use user-assigned managed identities, an [HTTP call](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token#get-a-token-using-http) is required.

```c#
using Microsoft.Azure.Services.AppAuthentication;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage.Blob;
using System;

namespace storage_msi
{
    class Program
    {
        static void Main(string[] args)
        {
            // Only use this connectionString for local development.
            var azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=Developer; DeveloperTool=VisualStudio");
            string accessToken = azureServiceTokenProvider.GetAccessTokenAsync("https://storage.azure.com/").Result;

            Console.WriteLine(accessToken);
            // Create storage credentials from your managed identity access token.
            TokenCredential tokenCredential = new TokenCredential(accessToken);
            StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

            // Create a block blob using the credentials.
            CloudBlockBlob blob = new CloudBlockBlob(new Uri("https://jlst.blob.core.windows.net/pic/Blob1.txt"), storageCredentials);
            blob.UploadText("test");
        }
    }
}
```