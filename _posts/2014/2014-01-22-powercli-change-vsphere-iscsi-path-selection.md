---
  title: PowerCLI -- Change vSphere iSCSI Path Selection
  date: 2014-01-22 19:16:55
---

Here is just a quick PowerCLI script to change all of your iSCSI
datastores to "Round Robin" path selection. The default path selection
for all datastores is "Most Recently Used" which means that any given
time you will only be utilizing one storage path. So why would we want
to do this? The answer is that we want to utilize multiple paths to our
storage to increase our throughput which is called MPIO (MultiPath I/O).
So in order to automate these changes we can use a PowerCLI script which
is what is provided below. You will need to tweak the variables
($vi_server, $vcuser and $vcpass) to match your vCenter environment.

Updated because below caused a loop!

```powershell
$vi_server = "vcsa"
$vcuser = "root"
$vcpass = "vmware1"
Connect-VIServer -Server $vi_server -User $vcuser -Password $vcpass
foreach ($vmhost in $vmhosts) {
$HBANumber = Get-VMHostHba -VMHost $vmhost -Type iSCSI
Get-ScsiLun –Hba $HBANumber | Set-ScsiLun –MultipathPolicy “RoundRobin”
}
```

The correct way to do this is as below.

PowerCLI Script for building lab hosts

```powershell
# @mrlesmithjr
# EverythingShouldBeVirtual.com

$vi_server = "vcenter"
$vcuser = "root"
$vcpass = "vmware1"

Connect-VIServer -Server $vi_server -User $vcuser -Password $vcpass

# Setup variable to use in script for all hosts in vCenter
$vmhosts = @(Get-VMHost)

foreach ($vmhost in $vmhosts) {
    # Setup esxcli for use on each host
    $esxcli = Get-EsxCli -VMHost $vmhost
    # Get a list of all available LUNs, sorting them by device name
    $lunList = Get-VMHost $vmhost | Get-ScsiLun -CanonicalName “*” | % {$_.CanonicalName}
    foreach ($lun in $lunList) {
        $esxcli.storage.nmp.device.set($null,$lun,”VMW_PSP_RR”)
    }
}

Disconnect-VIServer * -Confirm:$false
```

Enjoy!
