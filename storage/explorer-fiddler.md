# Storage Explorer - Capture Fiddler trace

To capture Fiddler trace for Storage Explorer, we need to enable proxying and HTTPS decryption by adding environment variables.

CMD:

```batch
set https_proxy=http://127.0.0.1:8888 
set http_proxy=http://127.0.0.1:8888
set NODE_TLS_REJECT_UNAUTHORIZED=0
"C:\Program Files (x86)\Microsoft Azure Storage Explorer\StorageExplorer.exe"
```

PS: `set` doesn't work in PowerShell because PowerShell's `set` is an alias to `Set-Variable`, which only changes the script's variables. for PowerShell, use `$Env:<variable-name> = "<new-value>"`

PowerShell:

```powershell
$Env:HTTPS_PROXY = "http://127.0.0.1:8888"
$Env:HTTP_PROXY = "http://127.0.0.1:8888"
$Env:NODE_TLS_REJECT_UNAUTHORIZED = 0
"C:\Program Files (x86)\Microsoft Azure Storage Explorer\StorageExplorer.exe"
```

To display the values of all the environment variables, type:

```powershell
Get-ChildItem Env:
```

See Also: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables?view=powershell-6
