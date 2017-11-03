---
  title: Getting started with Hyper-V 2012
  date: 2013-07-10 10:29:23
---

I will be using this post for Hyper-V notes as I go through setting up
and using Hyper-V 2012. This may not be informative to all but I am
really just using it for some note taking.

Add hyper-v management tools on another server 2012 server.

Connect iSCSI storage.

From command line on Hyper-V 2012 Server run the following commands

GUI Based

```powershell
iscsicpl
```

Command Line

```powershell
Set-Service –Name MSiSCSI –StartupType Automatic
Start-Service MSiSCSI
```

List iSCSI IQN for the iSCSI Initiator

```powershell
(Get-InitiatorPort).NodeAddress
Get-IscsiConnection
```

Now, let's make the session for this iSCSI connection persist across
reboots with the following command:

```powershell
Get-IscsiSession | Register-IscsiSession
Get-IscsiSession
Get-Disk | Where-Object BusType –eq “iSCSI”
Initialize-Disk –Number <Disk_Number> –PartitionStyle GPT –PassThru | New-Partition –AssignDriveLetter –UseMaximumSize | Format-Volume
```

Install iSCSI MPIO

```powershell
Powershell
Install-WindowsFeature Multipath-IO
```

To allow remote management from another server 2012 the following
firewall rules need to be added

```powershell
netsh advfirewall firewall set rule group="Remote Administration" new enable=yes
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=yes
netsh advfirewall firewall set rule group="Remote Service Management" new enable=yes
netsh advfirewall firewall set rule group="Performance Logs and Alerts" new enable=yes
Netsh advfirewall firewall set rule group="Remote Event Log Management" new enable=yes
Netsh advfirewall firewall set rule group="Remote Scheduled Tasks Management" new enable=yes
netsh advfirewall firewall set rule group="Remote Volume Management" new enable=yes
netsh advfirewall firewall set rule group="Remote Desktop" new enable=yes
netsh advfirewall firewall set rule group="Windows Firewall Remote Management" new enable =yes
netsh advfirewall firewall set rule group="windows management instrumentation (wmi)" new enable =yes
sc config vds start= auto
net start vds
```
