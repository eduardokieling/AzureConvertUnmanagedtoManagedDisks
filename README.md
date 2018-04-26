# AzureConvertUnmanagedtoManagedDisks

DEFAULT VMs
----------------------------

1) Deallocate the VM by using the Stop-AzureRmVM cmdlet

```
$rgName = "myResourceGroup"
$vmName = "myVM"
Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName -Force
```

2) Convert the VM to managed disks by using the ConvertTo-AzureRmVMManagedDisk cmdlet.
```
ConvertTo-AzureRmVMManagedDisk -ResourceGroupName $rgName -VMName $vmName
```



VMs IN AN AVAILABILITY
----------------------------

1) Convert the availability set by using the Update-AzureRmAvailabilitySet cmdlet. 
```
$rgName = 'myResourceGroup'
$avSetName = 'myAvailabilitySet'

$avSet = Get-AzureRmAvailabilitySet -ResourceGroupName $rgName -Name $avSetName
Update-AzureRmAvailabilitySet -AvailabilitySet $avSet -Sku Aligned
```
2) If the region where your availability set is located has only 2 managed fault domains but the number of unmanaged fault domains is 3, this command shows an error similar to "The specified fault domain count 3 must fall in the range 1 to 2." To resolve the error, update the fault domain to 2 and update Sku to Aligned as follows:
```
$avSet.PlatformFaultDomainCount = 2
Update-AzureRmAvailabilitySet -AvailabilitySet $avSet -Sku Aligned
```

3) Deallocate and convert the VMs in the availability set.
```
$avSet = Get-AzureRmAvailabilitySet -ResourceGroupName $rgName -Name $avSetName

foreach($vmInfo in $avSet.VirtualMachinesReferences)
{
  $vm = Get-AzureRmVM -ResourceGroupName $rgName | Where-Object {$_.Id -eq $vmInfo.id}
  Stop-AzureRmVM -ResourceGroupName $rgName -Name $vm.Name -Force
  ConvertTo-AzureRmVMManagedDisk -ResourceGroupName $rgName -VMName $vm.Name
}
```
