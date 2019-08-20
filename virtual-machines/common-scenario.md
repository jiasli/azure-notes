# Azure Virtual Machine Common Scenario

## Moving a VM to another VNET

Moving a VM to another VNET is **impossible**. We need to delete the VM first and recreate a new VM from the old **Disk**. Please note deleting a VM won't automatically delete the Disk.

Log in to Azure Portal,

1. **Virtual Machines** > Select a VM > **Delete**
1. **Disks** > Select a disk > **Create VM**
