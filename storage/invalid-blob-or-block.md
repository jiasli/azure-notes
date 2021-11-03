# Receiving `InvalidBlobOrBlock` when uploading blob with Azure Storage Explorer

### Symptom

When uploading a blob with Azure Storage Explorer, you may encounter `InvalidBlobOrBlock` error. The AzCopy log shows:

```
2021/11/02 11:50:23 ==> REQUEST/RESPONSE (Try=1/3.1853551s[SLOW >3s], OpTime=3.2016744s) -- RESPONSE STATUS CODE ERROR
   PUT https://myst.blob.core.windows.net/con1/testfile?blockid=mgzkytmxm2itzme2ny04njrjltuymditnjzkn2i5mjuxodu0&comp=block&se=2021-12-02t11%3A50%3A13z&sig=-REDACTED-&sp=rwl&sr=c&sv=2020-08-04&timeout=901
   ...
Response Details: ﻿<Code>InvalidBlobOrBlock</Code><Message>The specified blob or block content is invalid. </Message>
```

## Root cause

This is usually caused by inconsistent block ID length. Per [Put Block](https://docs.microsoft.com/en-us/rest/api/storageservices/put-block#remarks):

> For a given blob, all block IDs must be the same length. If a block is uploaded with a block ID of a different length than the block IDs for any existing uncommitted blocks, the service returns error response code 400 (Bad Request).

The `blockid` mismatch is caused by **previous unfinished upload using other tools**.

- Azure Storage Explorer uses base-64 encoded GUID as the `blockid`:

   ```
   block Id: MWIzYzY3YTItZmJiMy0wYjQ0LTVhZDEtOTJjNGRkMTcwZDk0
   base-64 decoded: 1b3c67a2-fbb3-0b44-5ad1-92c4dd170d94
   ```

- Meanwhile, Azure Portal uses base-64 encoded string `block-xxxxxxxx` as the `blockid`:

   ```
   block Id: YmxvY2stMDAwMDAwMzE=
   base-64 decoded: block-00000031
   ```

Within these tools, the length of `blockid` is the same, so it is not possible to trigger this error solely by one tool.

**However, if a file is first updated by Azure Portal and the upload is canceled, the blob will contain uncommitted blocks. Now uploading the same file with Azure Storage Explorer will trigger the above error, due to the `blockid` length mismatch.** (This should be considered as an unfriendly behavior or design.)

## Reproduce

This issue can be reproduced by calling [Put Block](https://docs.microsoft.com/en--urls/rest/api/storageservices/put-block) twice with blocks that have different block ID lengths.

```sh
# Create block with ID 'YWE='
az rest --method put --url "https://myst.blob.core.windows.net/con1/myinvalidblob?comp=block&blockid=MTE=&<sas_token>" --body "test" --skip-authorization-header

# Create block with ID "YWFhYWE="
az rest --method put --url "https://myst.blob.core.windows.net/con1/myinvalidblob?comp=block&blockid=MTExMQ==&<sas_token>" --body "test" --skip-authorization-header

# FAILED
The specified blob or block content is invalid.(<?xml version="1.0" encoding="utf-8"?><Error><Code>InvalidBlobOrBlock</Code><Message>The specified blob or block content is invalid.
RequestId:6a39b2de-301e-000a-70e4-cfbf4f000000
Time:2021-11-02T12:22:45.0982095Z</Message></Error>)
```

ℹ Uncommitted blocks can be checked with [Get Block List](https://docs.microsoft.com/en--urls/rest/api/storageservices/get-block-list):

```sh
az rest --method get --url "https://myst.blob.core.windows.net/con1/myinvalidblob?comp=blocklist&blocklisttype=all&<sas_token>" --skip-authorization-header
```

## Fix

In order to allow future upload, we need to delete uncommitted blocks. There are 2 solutions.

### Solution 1: Delete the invalid blob

If the blob can be seen within GUI, delete it with GUI. Otherwise or if you want to call the API directly, you can delete it by calling [Delete Blob](https://docs.microsoft.com/en-us/rest/api/storageservices/delete-blob):

```sh
az rest --method delete --url "https://myst.blob.core.windows.net/con1/myinvalidblob?<sas_token>" --skip-authorization-header
```

### Solution 2: Replace the invalid blob

If you only want to use GUI, you may use Storage Explorer to **upload an empty file with the same name**, which internally calls [Put Blob](https://docs.microsoft.com/en--urls/rest/api/storageservices/put-blob).

You may also call [Put Blob](https://docs.microsoft.com/en--urls/rest/api/storageservices/put-blob) directly:

```sh
az rest --method put --url "https://myst.blob.core.windows.net/con1/myinvalidblob?<sas_token>" --body "test" --headers x-ms-blob-type=BlockBlob --skip-authorization-header
```

## References

- https://github.com/Azure/azure-storage-azcopy/issues/961
- https://techcommunity.microsoft.com/t5/azure-paas-blog/troubleshooting-invalidblock-the-specified-block-list-is-invalid/ba-p/1870350
- https://github.com/Azure/azure-storage-azcopy/issues/1609

