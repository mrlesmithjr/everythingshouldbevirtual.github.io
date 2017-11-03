---
  title: PowerCLI Script to add Syslog Servers
  date: 2013-12-12 14:06:40
---

I was asked earlier about this. So I figured I would throw this together
real quick. There was a concern about adding syslog server settings to
an enormous number of hosts in a cluster for Log Insight (or other). So
here is a quick PowerCLI script to do this for us. Nothing too fancy but
it works.

```powershell
# PowerCLI Script for adding syslogserver to hosts
# @mrlesmithjr
# EverythingShouldBeVirtual.com
# Change the following to match your environment
# vi_server is your vCenter
$vi_server = “vcenterservername”
$vcuser = "vcenterserverusername"
$vcpass = "vcenterserverpassword"
# Make sure to tweak the protocol and ports below to match your environment
$syslogservers = “udp://graylog2:514,tcp://logstash:514,udp://loginsight:514”

Connect-VIServer -Server $vi_server -User $vcuser -Password $vcpass

# Setup variable to use in script for all hosts in vCenter
$vmhosts = @(Get-VMHost)

# Configure syslog on each host in vCenter
foreach ($vmhost in $vmhosts) {
Write-Host ‘$vmhost = ‘ $vmhost
Set-VMHostAdvancedConfiguration -Name Syslog.global.logHost -Value "$syslogservers" -VMHost $vmhost
$esxcli = Get-EsxCli -VMHost $vmhost
$esxcli.system.syslog.reload()
}

Disconnect-VIServer * -Confirm:$false
```

Enjoy!
