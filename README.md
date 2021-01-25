# azurestorageimmutabilitypolicy

# Azure Storage Immutability Policy

When trying to set up a new policy on Azure to audit storage containers that do or do not have immutability policy assigned the policy evaluation doesn't work properly and is unnable to evaluate whether the condition is true or not. According to the following documentation is an expected behavior [azure-policy](https://github.com/Azure/azure-policy), so as a workaround to get this information you can use powershell instead by executing the following script.

```powershell
#login into azure
az login
# get all storage accounts 
$AccountName = az storage account list --query "[].{name:name}" -o tsv
# get storage accounts keys
$keys = $AccountName | ForEach-Object { @{key= (az storage account keys list -n $_ --query "[0].{value:value}" -o tsv); name = $_ } }
# get container names
$containerName = $keys | ForEach-Object { @{containerName=(az storage container list --account-name $_.name --account-key $_.key --query "[].{name:name}" -o tsv);accountName=$_.name } }
# new array to store results
$policy = @()
# get containers with and without inmmutability policies applied
$containerName | ForEach-Object { if ($_.containerName) { if ($_.containerName -is [array]) { foreach ($c in $_.containerName) { $policy += @{contName=$c; accountName=$_.accountName; policy=(az storage container immutability-policy show --account-name $_.accountName --container-name $c) } } } else { $policy += @{contName=$_.containerName; accountName=$_.accountName; policy=(az storage container immutability-policy show --account-name $_.accountName --container-name $_.containerName) } } } }
foreach ($c in $policy) { if ($c.policy) { "Storage Account Name: " + $c.accountName + ", Container: " + $c.contName + ", Policy: TRUE" } else { "Storage Account Name: " + $c.accountName + ", Container: " + $c.contName + ", Policy: FALSE" } }

# show results
ForEach-Object -InputObject $policy -Process { $_.policy }
```
