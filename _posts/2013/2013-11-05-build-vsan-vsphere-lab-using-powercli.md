---
  title: Build vSAN vSphere Lab using PowerCLI
  date: 2013-11-05 13:28:32
---

Here is my first contribution as far as any PowerCLI scripts go. I am
myself just learning finally so I will share with everyone as I am
learning. Obviously feel free to educate me on any changes that could be
made to make this better or to add more functionality. It is not a
complete build from start to finish for vSAN but enables vSAN for the
cluster and creates vSAN Network and VMKernel ports.

This script is meant for an existing vCenter to exist and only two
standalone hosts, however it can be modified to adjust for your
deployment. It will reconfigure the hosts, join them to vCenter, migrate
VSS (vSphere Standard Switches) to VDS (vSphere Distributed Switches),
configure VMKernel ports for vMotion, vSAN, and iSCSI, also set port
bindings on iSCSI configuration. There are a few other things it will do
too but I included a few notes.

```powershell
# PowerCLI Script for building lab hosts
# @mrlesmithjr
# EverythingShouldBeVirtual.com

$vi_server = "192.168.58.130"
$vcuser = "root"
$vcpass = "vmware1"
$dc = "EverythingShouldBeVirtual Lab"
$dns_domain = "lab.local"
$cluster = "HA-DRS-Cluster"
$host1 = "192.168.58.128"
$host2 = "192.168.58.129"
$hostuser = "root"
$hostpass = "vmware1"
$gateway = "192.168.58.1"
$hostname1 = "esxi01"
$hostname2 = "esxi02"
$nas = "192.168.58.144"

Connect-VIServer -Server $vi_server -User $vcuser -Password $vcpass
New-DataCenter -Location (Get-Folder -NoRecursion) -Name $dc
New-Cluster $cluster -DRSEnabled -Location $dc -DRSAutomationLevel FullyAutomated -HAEnabled -VsanEnabled -HAAdmissionControlEnabled:$True -confirm:$False
New-AdvancedSetting -Entity $cluster -Type ClusterHA -Name 'das.isolationaddress1' -Value $gateway -confirm:$False -force
New-AdvancedSetting -Entity $cluster -Type ClusterHA -Name 'das.usedefaultisolationaddress' -Value false -confirm:$False -force

# Add hosts to Datacenter
Add-VMHost $host1 -Location $dc -User $hostuser -Password $hostpass -Force
Add-VMHost $host2 -Location $dc -User $hostuser -Password $hostpass -Force

# Change hostnames and dns domain
$vmHostNetworkInfo = Get-VmHostNetwork -Host $host1
Set-VMHostNetwork -Network $vmHostNetworkInfo -DomainName $dns_domain -HostName $hostname1 -DnsFromDhcp $false
$vmHostNetworkInfo = Get-VmHostNetwork -Host $host2
Set-VMHostNetwork -Network $vmHostNetworkInfo -DomainName $dns_domain -HostName $hostname2 -DnsFromDhcp $false

# Setup variable to use in script for all hosts in vCenter
$vmhosts = @(Get-VMHost)

# Rename local datastore
foreach ($vmhost in $vmhosts) {
$dsname = "$vmhost-local"
$vmhost | Get-Datastore -Name datastore* | Set-Datastore -Name $dsname
}

# Change numuplinkports to match the max number of uplink ports per host you want set
$numuplinkports = "2"
$vds_name = "LabVDS"
$mgmt_portgroup = "Management Network"
$vmotion_portgroup = "vMotion Network"
$vm_portgroup = "VM Network"
$storage_portgroup1 = "iSCSI-1"
$storage_portgroup2 = "iSCSI-2"
$vsan_portgroup = "vSAN Network"

# Create VDS and Portgroups
Write-Host "`nCreating new VDS" $vds_name
New-VDSwitch -Name $vds_name -Location $dc -NumUplinkPorts $numuplinkports
Write-Host "Creating new Management DVPortgroup"
Get-VDSwitch -Name $vds_name | New-VDPortgroup -Name $mgmt_portgroup | Out-Null
Write-Host "Creating new vMotion DVPortgroup"
Get-VDSwitch -Name $vds_name | New-VDPortgroup -Name $vmotion_portgroup | Out-Null
Write-Host "Creating new VM DVPortgroup`n"
Get-VDSwitch -Name $vds_name | New-VDPortgroup -Name $vm_portgroup | Out-Null
Write-Host "Creating new Storage DVPortgroup`n"
Get-VDSwitch -Name $vds_name | New-VDPortgroup -Name $storage_portgroup1 | Out-Null
Get-VDPortgroup $storage_portgroup1 | Get-VDUplinkTeamingPolicy | Set-VDUplinkTeamingPolicy -UnusedUplinkPort "dvUplink2"
Write-Host "Creating new Storage DVPortgroup`n"
Get-VDSwitch -Name $vds_name | New-VDPortgroup -Name $storage_portgroup2 | Out-Null
Get-VDPortgroup $storage_portgroup2 | Get-VDUplinkTeamingPolicy | Set-VDUplinkTeamingPolicy -UnusedUplinkPort "dvUplink1"
Write-Host "Creating new vSAN DVPortgroup`n"
Get-VDSwitch -Name $vds_name | New-VDPortgroup -Name $vsan_portgroup | Out-Null

# Add hosts to VDS and add vmnic1 for each host to VDS
foreach ($vmhost in $vmhosts) {
Write-Host "Adding" $vmhost "to" $vds_name
Get-VDSwitch -Name $vds_name | Add-VDSwitchVMHost -VMHost $vmhost | Out-Null
Write-Host "Adding" $vmhost "vmnic1 to" $vds_name
$vmhostNetworkAdapter = Get-VMHost $vmhost | Get-VMHostNetworkAdapter -Physical -Name vmnic1
Get-VDSwitch -Name $vds_name | Add-VDSwitchPhysicalNetworkAdapter -VMHostPhysicalNic $vmhostNetworkAdapter -Confirm:$false
}

# Move vmk0 (Management Port) to VDS
foreach ($vmhost in $vmhosts) {
$mgmt_portgroup = "Management Network"
$dvportgroup = Get-VDPortgroup -name $mgmt_portgroup -VDSwitch $vds_name
$vmk = Get-VMHostNetworkAdapter -Name vmk0
Set-VMHostNetworkAdapter -PortGroup $dvportgroup -VirtualNic $vmk -confirm:$false | Out-Null
}

# Move vmnic0 from VSS to VDS and remove vswitch0
foreach ($vmhost in $vmhosts) {
$vmhostNetworkAdapter = Get-VMHost $vmhost | Get-VMHostNetworkAdapter -Physical -Name vmnic0
Get-VDSwitch -Name $vds_name | Add-VDSwitchPhysicalNetworkAdapter -VMHostNetworkAdapter $vmhostNetworkAdapter -Confirm:$false
$vswitch = Get-VirtualSwitch -VMHost $vmhost -Name vSwitch0
Remove-VirtualSwitch -VirtualSwitch $vswitch -Confirm:$false
}

# Create vMotion, iSCSI  and vSAN VMKernel Ports
foreach ($vmhost in $vmhosts) {
$vs = Get-VDSwitch -Name $vds_name
New-vmhostnetworkadapter -VMHost $vmhost -PortGroup $vmotion_portgroup -VirtualSwitch $vs -VMotionEnabled $true
New-vmhostnetworkadapter -VMHost $vmhost -PortGroup $storage_portgroup1 -VirtualSwitch $vs
New-vmhostnetworkadapter -VMHost $vmhost -PortGroup $storage_portgroup2 -VirtualSwitch $vs
New-vmhostnetworkadapter -VMHost $vmhost -PortGroup $vsan_portgroup -VirtualSwitch $vs -VsanTrafficEnabled $true
}

# Enable the software ISCSI adapter if not already enabled and Bind VMKernel Ports to iSCSI Adapter
foreach ($vmhost in $vmhosts) {
$VMHostStorage = Get-VMHostStorage -VMHost $vmhost | Set-VMHostStorage -SoftwareIScsiEnabled $True
Start-Sleep -Seconds 30
$HBANumber = Get-VMHostHba -VMHost $vmhost -Type iSCSI | %{$_.Device}
 # Sets up PowerCLI to be able to access esxcli commands
 $esxcli = Get-EsxCli -VMHost $vmhost
 $esxcli.iscsi.networkportal.add($HBANumber, $true, "vmk2")
 $esxcli.iscsi.networkportal.add($HBANumber, $true, "vmk3")
}

# Attach NAS iSCSI LUN(s)
foreach ($vmhost in $vmhosts) {
Get-VMHostHba -VMhost $vmhost -Type iScsi | New-IScsiHbaTarget -Address $nas -Type Send
Get-VMHostStorage $vmhost -RescanAllHba
Get-ScsiLun –Hba $HBANumber | Set-ScsiLun –MultipathPolicy “RoundRobin”
}

# Move hosts into cluster
foreach ($vmhost in $vmhosts) {
Move-VMHost $vmhost -Destination $cluster
}

Disconnect-VIServer * -Confirm:$false
```

Enjoy!
