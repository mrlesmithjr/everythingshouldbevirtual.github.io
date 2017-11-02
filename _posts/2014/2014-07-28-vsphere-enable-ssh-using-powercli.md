---
  title: vSphere - Enable SSH using PowerCLI
  date: 2014-07-28
---

In case you have the need to enable SSH, set SSH service to start on
boot and also set the firewall rule to allow ALL IPs you can use the
following PowerCLI script. If you need to lock down the firewall to only
allow from a certain IP you can uncomment the appropriate lines and
comment out the others. This script will also disable the SSH alert for
every host in vCenter to get rid of the nagging alert. This script will
connect to vCenter and run against every host in vCenter as well.

```powershell
# PowerCLI Script for enabling SSH, setting firewall and disable SSH alert in vCenter
# @mrlesmithjr
# EverythingShouldBeVirtual.com

# Setup common variables to use
$vi_server = "vcsa.everythingshouldbevirtual.local" # Set to your vCenter hostname|IP
$vcuser = "vcenteruser" # Set to your vCenter username to connect
$vcpass = "vcenterpassword" # Set to your vCenter username password to connect
$ip = 10.0.101.122 # Set IP to your manamgement station if you are locking down the firewall

Connect-VIServer -Server $vi_server -User $vcuser -Password $vcpass

# Setup variable to use in script for all hosts in vCenter
$vmhosts = @(Get-VMHost)

foreach ($vmhost in $vmhosts) {
    write-host "Configuring SSH on host: $($vmHost.Name)" -fore Yellow
    if((Get-VMHostService -VMHost $vmHost | where {$_.Key -eq "TSM-SSH"}).Policy -ne "on"){
        Write-Host "Setting SSH service policy to automatic on $($vmHost.Name)"
        Get-VMHostService -VMHost $vmHost | where { $_.key -eq "TSM-SSH" } | Set-VMHostService -Policy "On" -Confirm:$false -ea 1 | Out-null
    }
    if((Get-VMHostService -VMHost $vmHost | where {$_.Key -eq "TSM-SSH"}).Running -ne $true){
        Write-Host "Starting SSH service on $($vmHost.Name)"
        Start-VMHostService -HostService (Get-VMHost $vmHost | Get-VMHostService | Where { $_.Key -eq "TSM-SSH"}) | Out-null
    }
    $esxcli = Get-EsxCli -VMHost $vmhost
    if($esxcli -ne $null){
        # Uncomment below to set firewall to allow only from a certain IP
        #if(($esxcli.network.firewall.ruleset.allowedip.list("sshServer") | select AllowedIPAddresses).AllowedIPAddresses -eq "All"){
            #Write-Host "Changing the sshServer firewall configuration"
                #$esxcli.network.firewall.ruleset.set($false, $true, "sshServer")
                #$esxcli.network.firewall.ruleset.allowedip.add("$ip", "sshServer")
                #$esxcli.network.firewall.refresh()
        #}
        # End Uncomment
        # Comment out below if using the above to set firewall to a specific IP
        if(($esxcli.network.firewall.ruleset.allowedip.list("sshServer") | select AllowedIPAddresses).AllowedIPAddresses -ne "All"){
            Write-Host "Changing the sshServer firewall configuration"
                $esxcli.network.firewall.ruleset.set($true, $true, "sshServer")
                $esxcli.network.firewall.refresh()
        }
        # End Comment
        if(($vmHost | Get-AdvancedSetting | Where {$_.Name -eq "UserVars.SuppressShellWarning"}).Value -ne "1"){
            Write-Host "Suppress the SSH warning message"
                $vmHost | Get-AdvancedSetting | Where {$_.Name -eq "UserVars.SuppressShellWarning"} | Set-AdvancedSetting -Value "1" -Confirm:$false | Out-null
        }
}
# Disconnect from vCenter
Disconnect-VIServer * -Confirm:$false
```

Enjoy!
