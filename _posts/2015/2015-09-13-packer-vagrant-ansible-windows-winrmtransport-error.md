---
  title: Packer - Vagrant - Ansible - Windows
---

While doing some Packer builds for Windows Server 2012 R2 to be used
with Vagrant in order to do some Ansible learning I stumbled across this
issue.

```bash
ansible-playbook -i hosts playbook.yml

PLAY [windows] ****************************************************************

GATHERING FACTS ***************************************************************
fatal: [vagrant-windows-2012-r2] => 500 WinRMTransport. [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:590)

TASK: [run ipconfig] **********************************************************
FATAL: no hosts matched or all hosts have already failed -- aborting


PLAY RECAP ********************************************************************
           to retry, use: --limit @/Users/larrysmith/playbook.retry

vagrant-windows-2012-r2    : ok=0    changed=0    unreachable=1    failed=0
```

After beating my head against the wall and just before giving up I came
across this [post](https://github.com/ansible/ansible/issues/10294#issuecomment-93494025).
Which goes over the related issue along with a temporary fix. Thank God!

So basically all you need to do is create a folder (callback_plugins)
within the folder where you are executing your Ansible playbook from and
create a file (fix-ssl.py) which will temporarily solve the issue.

```bash
mkdir callback_plugins
nano callback_plugins/fix-ssl.py
```

And paste the following within this file and save.

```python
import ssl
if hasattr(ssl, '_create_default_https_context') and hasattr(ssl, '_create_unverified_context'):
    ssl._create_default_https_context = ssl._create_unverified_context

class CallbackModule(object):
    pass
```

Now attempt to run your Ansible playbook once again and BOOM!

```raw
ansible-playbook -i hosts playbook.yml

PLAY [windows] ****************************************************************

GATHERING FACTS ***************************************************************
ok: [vagrant-windows-2012-r2]

TASK: [run ipconfig] **********************************************************
ok: [vagrant-windows-2012-r2]

TASK: [debug var=ipconfig] ****************************************************
ok: [vagrant-windows-2012-r2] => {
    "var": {
        "ipconfig": {
            "invocation": {
                "module_args": "ipconfig",
                "module_complex_args": {},
                "module_name": "raw"
            },
            "rc": 0,
            "stderr": "",
            "stdout": "\r\nWindows IP Configuration\r\n\r\n\r\nEthernet adapter Ethernet:\r\n\r\n   Connection-specific DNS Suffix  . : everythingshouldbevirtual.local\r\n   Link-local IPv6 Address . . . . . : fe80::acf2:968e:f06f:16f0%12\r\n   IPv4 Address. . . . . . . . . . . : 10.0.2.15\r\n   Subnet Mask . . . . . . . . . . . : 255.255.255.0\r\n   Default Gateway . . . . . . . . . : 10.0.2.2\r\n\r\nTunnel adapter isatap.everythingshouldbevirtual.local:\r\n\r\n   Media State . . . . . . . . . . . : Media disconnected\r\n   Connection-specific DNS Suffix  . : everythingshouldbevirtual.local\r\n",
            "stdout_lines": [
                "",
                "Windows IP Configuration",
                "",
                "",
                "Ethernet adapter Ethernet:",
                "",
                "   Connection-specific DNS Suffix  . : everythingshouldbevirtual.local",
                "   Link-local IPv6 Address . . . . . : fe80::acf2:968e:f06f:16f0%12",
                "   IPv4 Address. . . . . . . . . . . : 10.0.2.15",
                "   Subnet Mask . . . . . . . . . . . : 255.255.255.0",
                "   Default Gateway . . . . . . . . . : 10.0.2.2",
                "",
                "Tunnel adapter isatap.everythingshouldbevirtual.local:",
                "",
                "   Media State . . . . . . . . . . . : Media disconnected",
                "   Connection-specific DNS Suffix  . : everythingshouldbevirtual.local"
            ]
        }
    }
}

PLAY RECAP *****************************************************************
```

Below is my Ansible hosts file with details.

```bash
vagrant-windows-2012-r2 ansible_ssh_host=127.0.0.1

[windows]
vagrant-windows-2012-r2
```

And here is my group_vars/windows.yml

```yaml
---
ansible_ssh_user: vagrant
ansible_ssh_pass: vagrant
ansible_ssh_port: 55986
ansible_connection: winrm
```

And the simple Ansible playbook (playbook.yml) that I am running.

```yaml
---
- hosts: windows
  tasks:
    - name: run ipconfig
      raw: ipconfig
      register: ipconfig
    - debug: var=ipconfig
```

In case you are interested here is the Vagrantfile that I am
provisioning from.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.2"

Vagrant.configure("2") do |config|
    config.vm.define "vagrant-windows-2012-r2"
    config.vm.box = "windows_2012_r2"
    config.vm.communicator = "winrm"

    # Admin user name and password
    config.winrm.username = "vagrant"
    config.winrm.password = "vagrant"

    config.vm.guest = :windows
    config.windows.halt_timeout = 15

    config.vm.network :forwarded_port, guest: 3389, host: 3389, id: "rdp", auto_correct: true
    config.vm.network :forwarded_port, guest: 22, host: 2222, id: "ssh", auto_correct: true

    config.vm.provider :virtualbox do |v, override|
        #v.gui = true
        v.customize ["modifyvm", :id, "--memory", 2048]
        v.customize ["modifyvm", :id, "--cpus", 2]
        v.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
    end

    config.vm.provider :vmware_fusion do |v, override|
        #v.gui = true
        v.vmx["memsize"] = "2048"
        v.vmx["numvcpus"] = "2"
        v.vmx["ethernet0.virtualDev"] = "vmxnet3"
        v.vmx["RemoteDisplay.vnc.enabled"] = "false"
        v.vmx["RemoteDisplay.vnc.port"] = "5900"
        v.vmx["scsi0.virtualDev"] = "lsisas1068"
    end

    config.vm.provider :vmware_workstation do |v, override|
        #v.gui = true
        v.vmx["memsize"] = "2048"
        v.vmx["numvcpus"] = "2"
        v.vmx["ethernet0.virtualDev"] = "vmxnet3"
        v.vmx["RemoteDisplay.vnc.enabled"] = "false"
        v.vmx["RemoteDisplay.vnc.port"] = "5900"
        v.vmx["scsi0.virtualDev"] = "lsisas1068"
    end
    config.vm.provision "shell", path: "scripts/ConfigureRemotingForAnsible.ps1"
end
```

And the powershell provision script to configure WinRM in order to run
Ansible which can also be found
[here](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1).

```powershell
# Configure a Windows host for remote management with Ansible
# -----------------------------------------------------------
#
# This script checks the current WinRM/PSRemoting configuration and makes the
# necessary changes to allow Ansible to connect, authenticate and execute
# PowerShell commands.
#
# Set $VerbosePreference = "Continue" before running the script in order to
# see the output messages.
#
# Written by Trond Hindenes <trond@hindenes.com>
# Updated by Chris Church <cchurch@ansible.com>
#
# Version 1.0 - July 6th, 2014
# Version 1.1 - November 11th, 2014

Param (
    [string]$SubjectName = $env:COMPUTERNAME,
    [int]$CertValidityDays = 365,
    $CreateSelfSignedCert = $true
)


Function New-LegacySelfSignedCert
{
    Param (
        [string]$SubjectName,
        [int]$ValidDays = 365
    )

    $name = New-Object -COM "X509Enrollment.CX500DistinguishedName.1"
    $name.Encode("CN=$SubjectName", 0)

    $key = New-Object -COM "X509Enrollment.CX509PrivateKey.1"
    $key.ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
    $key.KeySpec = 1
    $key.Length = 1024
    $key.SecurityDescriptor = "D:PAI(A;;0xd01f01ff;;;SY)(A;;0xd01f01ff;;;BA)(A;;0x80120089;;;NS)"
    $key.MachineContext = 1
    $key.Create()

    $serverauthoid = New-Object -COM "X509Enrollment.CObjectId.1"
    $serverauthoid.InitializeFromValue("1.3.6.1.5.5.7.3.1")
    $ekuoids = New-Object -COM "X509Enrollment.CObjectIds.1"
    $ekuoids.Add($serverauthoid)
    $ekuext = New-Object -COM "X509Enrollment.CX509ExtensionEnhancedKeyUsage.1"
    $ekuext.InitializeEncode($ekuoids)

    $cert = New-Object -COM "X509Enrollment.CX509CertificateRequestCertificate.1"
    $cert.InitializeFromPrivateKey(2, $key, "")
    $cert.Subject = $name
    $cert.Issuer = $cert.Subject
    $cert.NotBefore = (Get-Date).AddDays(-1)
    $cert.NotAfter = $cert.NotBefore.AddDays($ValidDays)
    $cert.X509Extensions.Add($ekuext)
    $cert.Encode()

    $enrollment = New-Object -COM "X509Enrollment.CX509Enrollment.1"
    $enrollment.InitializeFromRequest($cert)
    $certdata = $enrollment.CreateRequest(0)
    $enrollment.InstallResponse(2, $certdata, 0, "")

    # Return the thumbprint of the last installed cert.
    Get-ChildItem "Cert:\LocalMachine\my"| Sort-Object NotBefore -Descending | Select -First 1 | Select -Expand Thumbprint
}


# Setup error handling.
Trap
{
    $_
    Exit 1
}
$ErrorActionPreference = "Stop"


# Detect PowerShell version.
If ($PSVersionTable.PSVersion.Major -lt 3)
{
    Throw "PowerShell version 3 or higher is required."
}


# Find and start the WinRM service.
Write-Verbose "Verifying WinRM service."
If (!(Get-Service "WinRM"))
{
    Throw "Unable to find the WinRM service."
}
ElseIf ((Get-Service "WinRM").Status -ne "Running")
{
    Write-Verbose "Starting WinRM service."
    Start-Service -Name "WinRM" -ErrorAction Stop
}


# WinRM should be running; check that we have a PS session config.
If (!(Get-PSSessionConfiguration -Verbose:$false) -or (!(Get-ChildItem WSMan:\localhost\Listener)))
{
    Write-Verbose "Enabling PS Remoting."
    Enable-PSRemoting -Force -ErrorAction Stop
}
Else
{
    Write-Verbose "PS Remoting is already enabled."
}

# Make sure there is a SSL listener.
$listeners = Get-ChildItem WSMan:\localhost\Listener
If (!($listeners | Where {$_.Keys -like "TRANSPORT=HTTPS"}))
{
    # HTTPS-based endpoint does not exist.
    If (Get-Command "New-SelfSignedCertificate" -ErrorAction SilentlyContinue)
    {
        $cert = New-SelfSignedCertificate -DnsName $env:COMPUTERNAME -CertStoreLocation "Cert:\LocalMachine\My"
        $thumbprint = $cert.Thumbprint
    }
    Else
    {
        $thumbprint = New-LegacySelfSignedCert -SubjectName $env:COMPUTERNAME
    }

    # Create the hashtables of settings to be used.
    $valueset = @{}
    $valueset.Add('Hostname', $env:COMPUTERNAME)
    $valueset.Add('CertificateThumbprint', $thumbprint)

    $selectorset = @{}
    $selectorset.Add('Transport', 'HTTPS')
    $selectorset.Add('Address', '*')

    Write-Verbose "Enabling SSL listener."
    New-WSManInstance -ResourceURI 'winrm/config/Listener' -SelectorSet $selectorset -ValueSet $valueset
}
Else
{
    Write-Verbose "SSL listener is already active."
}


# Check for basic authentication.
$basicAuthSetting = Get-ChildItem WSMan:\localhost\Service\Auth | Where {$_.Name -eq "Basic"}
If (($basicAuthSetting.Value) -eq $false)
{
    Write-Verbose "Enabling basic auth support."
    Set-Item -Path "WSMan:\localhost\Service\Auth\Basic" -Value $true
}
Else
{
    Write-Verbose "Basic auth is already enabled."
}


# Configure firewall to allow WinRM HTTPS connections.
$fwtest1 = netsh advfirewall firewall show rule name="Allow WinRM HTTPS"
$fwtest2 = netsh advfirewall firewall show rule name="Allow WinRM HTTPS" profile=any
If ($fwtest1.count -lt 5)
{
    Write-Verbose "Adding firewall rule to allow WinRM HTTPS."
    netsh advfirewall firewall add rule profile=any name="Allow WinRM HTTPS" dir=in localport=5986 protocol=TCP action=allow
}
ElseIf (($fwtest1.count -ge 5) -and ($fwtest2.count -lt 5))
{
    Write-Verbose "Updating firewall rule to allow WinRM HTTPS for any profile."
    netsh advfirewall firewall set rule name="Allow WinRM HTTPS" new profile=any
}
Else
{
    Write-Verbose "Firewall rule already exists to allow WinRM HTTPS."
}

# Test a remoting connection to localhost, which should work.
$httpResult = Invoke-Command -ComputerName "localhost" -ScriptBlock {$env:COMPUTERNAME} -ErrorVariable httpError -ErrorAction SilentlyContinue
$httpsOptions = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck

$httpsResult = New-PSSession -UseSSL -ComputerName "localhost" -SessionOption $httpsOptions -ErrorVariable httpsError -ErrorAction SilentlyContinue

If ($httpResult -and $httpsResult)
{
    Write-Verbose "HTTP and HTTPS sessions are enabled."
}
ElseIf ($httpsResult -and !$httpResult)
{
    Write-Verbose "HTTP sessions are disabled, HTTPS session are enabled."
}
ElseIf ($httpResult -and !$httpsResult)
{
    Write-Verbose "HTTPS sessions are disabled, HTTP session are enabled."
}
Else
{
    Throw "Unable to establish an HTTP or HTTPS remoting session."
}

Write-Verbose "PS Remoting has been successfully configured for Ansible."
```

Hopefully this will help out someone else as well as it took me a few
hours to finally get all of the pieces together in order to get this
working.

Enjoy!
