# Change Access Tier of All Blobs in a Container

The script is based on answer from https://social.technet.microsoft.com/Forums/en-US/fb1c9421-be23-4521-83d7-967396052021/azure-blob-change-tier-to-archive-via-powershell?forum=windowsazuredata

The `$blob.ICloudBlob.SetStandardBlobTier()` operation uses [Methods of scalar objects and collections](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_methods?view=powershell-6#methods-of-scalar-objects-and-collections) to retrieve properties and execute methods for each item in the `$blob` collection.

```powershell
$StorageAccount = ""
$StorageKey = ""
$Container = ""

$ctx = New-AzureStorageContext -StorageAccountName $StorageAccount -StorageAccountKey $StorageKey

# Get all the blobs in container
$blob = Get-AzureStorageBlob -Container $Container -Context $ctx

# Set tier of all the blobs to Archive
$blob.ICloudBlob.SetStandardBlobTier("Cool")
```

## See Also

* [Azure Blob storage: hot, cool, and archive access tiers](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-storage-tiers)
* [about_Arrays](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_arrays?view=powershell-6)
* [about_Methods](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_methods?view=powershell-6)
