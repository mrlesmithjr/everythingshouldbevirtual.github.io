---
  title: How to install vShield 5.1 Manager, App and Endpoint
  date: 2012-11-15 17:24:27
---

In this guide we will be installing the vShield Manager appliance and
the installing vShield App and vShield Endpoint (Now called vShield
Networking and Security). You can reference the VMware quick start guide
[here](http://www.vmware.com/pdf/vshield_51_quickstart.pdf).

The first step in this is to install the vShield Manager appliance. The
vShield Manager is the core centralized network management component of
vShield. The vShield manager can be installed on a different host than
your vShield agents. This is a good candidate to run in a separate
management vSphere cluster, but not required. There is one vShield
Manager required per vCenter instance.

Taken from the quick start guide here are the hardware requirements for
the vShield components.

-   **_Memory_**
    -   **_vShield Manager: 8GB allocated, 3GB reserved_**
    -   **_vShield App: 1GB allocated, 1 GB reserved_**
    -   **_vShield Edge compact: 256 MB, large: 1 GB, x-large: 8 GB_**
    -   **_vShield Data Security: 512 MB_**
-   **_Disk Space_**
    -   **_vShield Manager: 60 GB_**
    -   **_vShield App: 5 GB per vShield App per ESX host_**
    -   **_vShield Edge compact and large: 320 MB, lx-Large: 4.4 GB
        (with 4 GB swap file)_**
    -   **_vShield Data Security: 6GB per ESX host_**
-   **_vCPU_**
    -   **_vShield Manager: 2_**
    -   **_vShield App: 2_**
    -   **_vShield Edge compact: 1, large and x-Large: 2_**
    -   **_vShield Data Security: 1_**

So let's deploy the appliance now.

Open the vSphere client and select file, Deploy OVF Template... And
follow the screenshots.

![16-46-54](../../assets/16-46-54_thumb.png "16-46-54")

![16-47-09](../../assets/16-47-09_thumb.png "16-47-09")

![16-47-22](../../assets/16-47-22_thumb.png "16-47-22")

![16-48-26](../../assets/16-48-26_thumb.png "16-48-26")

![16-48-33](../../assets/16-48-33_thumb.png "16-48-33")

![16-48-44](../../assets/16-48-44_thumb.png "16-48-44")

![16-48-56](../../assets/16-48-56_thumb.png "16-48-56")

![16-49-16](../../assets/16-49-16_thumb.png "16-49-16")

![16-49-30](../../assets/16-49-30_thumb.png "16-49-30")

Now the appliance will be deployed.

Once the appliance has been successfully deployed go ahead and power it
on. And we will now configure the appliance to begin using it.

Once the appliance has booted up you will need to login at the console.

![15-17-08](../../assets/15-17-08_thumb.png "15-17-08")

The default username is **_admin_** and the default password is
**_default_**.

Now at the command prompt type **_enable_** and enter the password from
above again.

![15-17-46](../../assets/15-17-46_thumb.png "15-17-46")

Now at the manager prompt type **_setup_**.

![15-17-58](../../assets/15-17-58_thumb.png "15-17-58")

Now you will need to enter the following information

-   **_IP Address_**
-   **_Subnet Mask_**
-   **_Default gateway_**
-   **_Primary DNS IP_**
-   **_Secondary DNS IP_**
-   **_DNS domain search list_**

And then save the new configuration and reboot the appliance.

After the appliance reboots open your web browser up and connect to
<https://appliance_ip>

![15-22-34](../../assets/15-22-34_thumb.png "15-22-34")

Login with **_admin_** and password is **_default_**.

![15-22-56](../../assets/15-22-56_thumb.png "15-22-56")

Click on settings and reports.

![15-23-06](../../assets/15-23-06_thumb.png "15-23-06")

The first thing we will configure is the Lookup Service

![15-24-12](../../assets/15-24-12_thumb.png "15-24-12")

Click edit and enter the following information

**_Lookup Service Host (Whatever server is configured as your SSO server
for vCenter Server 5.1)_**

**_Port (Leave default)_**

**_SSO Administrator Username
(_**[_admin@System-Domain_](mailto:admin@System-Domain)**_) (This is the
default unless you changed it during the SSO installation)_**

**_Password (Use the password that you configured during the SSO
installation when installing vCenter Server 5.1)_**

\*\*\*NOTE\*\*\* If you are using the vCenter Server Appliance you will
need to check
[this](http://www.virtuallyghetto.com/2012/09/default-password-for-vcenter-sso-admin.html "http\://www.virtuallyghetto.com/2012/09/default-password-for-vcenter-sso-admin.html")
link out for the password of admin@System-Domain. William Lam's post saved
me on this.

![15-27-15](../../assets/15-27-15_thumb.png "15-27-15")

Click OK and then choose "**_Yes_**" to accept the certificate

![15-27-30](../../assets/15-27-30_thumb.png "15-27-30")

Here is what the Lookup Service will look like when complete

![15-27-41](../../assets/15-27-41_thumb.png "15-27-41")

Now we will configure the vCenter Server section

![15-29-46](../../assets/15-29-46_thumb.png "15-29-46")

Click edit and enter the following information

**_vCenter Server (IP or hostname of your vCenter Server)_**

**_Administrator Username_**

**_Administrator Password_**

![15-30-17](../../assets/15-30-17_thumb.png "15-30-17")

Click OK and then choose "**_Yes_**" to accept the certificate

![15-30-30](../../assets/15-30-30_thumb.png "15-30-30")

Check Install this certificate and then click ignore

![15-30-55](../../assets/15-30-55_thumb.png "15-30-55")

Here is what the vCenter Server section will look like when completed

![15-31-09](../../assets/15-31-09_thumb.png "15-31-09")

Now we will configure the NTP Server section

![15-33-24](../../assets/15-33-24_thumb.png "15-33-24")

Enter IP or hostname of NTP server and click OK

![15-33-44](../../assets/15-33-44_thumb.png "15-33-44")

Now we have completed the initial setup and your settings and reports
will look like below which will show your clusters, hosts and vms.

![15-36-49](../../assets/15-36-49_thumb.png "15-36-49")

![15-37-34](../../assets/15-37-34_thumb.png "15-37-34")

Now open up Internet Explorer and add \*:\\\\vshield_IP to your trusted
security zone. If you do not do this you will not be able to open up the
vShield pages within vCenter.

![15-38-47](../../assets/15-38-47_thumb.png "15-38-47")

Now within vCenter Home screen you will see vShield listed at the bottom

![15-39-31](../../assets/15-39-31_thumb.png "15-39-31")

Click vShield and you will see the same login window as you did when
using your browser

Username **_admin_** and the password is **_default_**

Once logged in you will see the same interface as you did when using
your browser

Select datacenters

![15-40-14](../../assets/15-40-14_thumb.png "15-40-14")

Select your first host in the correct datacenter cluster that you want
to install vShield App on and select install vShield App. When you
install vShield app it will deploy a vm instance to the host that will
control all vms networking that are part of vShield. You cannot shut
down these vms unless the host is in maintenance mode. This is for
obvious reasons.

![15-44-14](../../assets/15-44-14_thumb.png "15-44-14")

Now select the datastore to use for the vShield App appliance, the
management port group to use for the appliance, enter the IP address,
Netmask and Default Gateway to use.

Click Install

![15-46-05](../../assets/15-46-05_thumb.png "15-46-05")

![15-46-21](../../assets/15-46-21_thumb.png "15-46-21")

The installation is complete

![16-13-53](../../assets/16-13-53_thumb.png "16-13-53")

Now install vShield Endpoint

![16-14-10](../../assets/16-14-10_thumb.png "16-14-10")

![16-14-46](../../assets/16-14-46_thumb.png "16-14-46")

![16-15-01](../../assets/16-15-01_thumb.png "16-15-01")

![16-15-34](../../assets/16-15-34_thumb.png "16-15-34")

Now follow the same process for vShield App and vShield Endpoint on each
host in your cluster.

That is all for the initial setup. Now you can click through some of the
sections within vShield and see additional areas that will need to be
setup when you are ready to start creating firewall rules and etc. But
for this is it for this guide. I will be creating another guide for
further configurations and use very soon. Which will include configuring
the vShield Edge devices. These are external connections within the
datacenter.

Below are some additional screenshots of information contained within
vShield. You should also start seeing some traffic details starting to
populate.

![16-28-11](../../assets/16-28-11_thumb.png "16-28-11")

![16-28-37](../../assets/16-28-37_thumb.png "16-28-37")

![16-29-01](../../assets/16-29-01_thumb.png "16-29-01")

![16-30-08](../../assets/16-30-08_thumb.png "16-30-08")

![16-30-30](../../assets/16-30-30_thumb.png "16-30-30")

![16-30-51](../../assets/16-30-51_thumb.png "16-30-51")

![16-31-16](../../assets/16-31-16_thumb.png "16-31-16")

![16-31-29](../../assets/16-31-29_thumb.png "16-31-29")
