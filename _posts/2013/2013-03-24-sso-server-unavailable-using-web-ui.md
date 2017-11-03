---
  title: SSO Server Unavailable Using Web UI
  date: 2013-03-24 10:43:55
---

I just ran into this issue. Tried to login to vSphere 5.1 web ui and
received the "SSO Server Unavailable". I tried restarting the vCenter
Single Sign On service and the VMware vSphere Web Client service without
any luck. A little bit of google searching and I found [this](http://pubs.vmware.com/vsphere-51/index.jsp?topic=%2Fcom.vmware.vsphere.vcenterhost.doc%2FGUID-321AB042-4F88-4A45-8FB2-36A343DFB6AB.html "http\://pubs.vmware.com/vsphere-51/index.jsp?topic=%2Fcom.vmware.vsphere.vcenterhost.doc%2FGUID-321AB042-4F88-4A45-8FB2-36A343DFB6AB.html") andit allowed me to get back in.

Below is an excerpt of the link from above.

Configure the vSphere Web Client to Bypass vCenter Single Sign On
You can configure the vSphere Web Client to bypass the vCenter Single
Sign On server. This allows the vSphere Web Client to connect directly
to vCenter Server 5.0 systems that are registered with the vSphere Web
Client.
If your environment contains both vCenter Server 5.0 and vCenter Server
5.1 system, you can use this setting to access the vCenter Server 5.0
systems if the vCenter Single Sign On Server becomes unavailable.
vCenter Single Sign On is required to connect to vCenter Server 5.1
systems. If you configure the vSphere Web Client to bypass Single Sign
On, you cannot use it to connect to any vCenter Server 5.1 systems until
you undo this configuration change.

Procedure

On the computer where the vSphere Web Client is installed, locate the
webclient.properties file. The location of this file depends on the
operating system on which the vSphere Web Client is installed.
Operating System File path

-   Windows 2003
    -   %ALLUSERPROFILE%Application Data\\VMware\\vSphere Web Client
-   Windows 2008
    -   %ALLUSERPROFILE%\\VMware\\vSphere Web Client
-   vCenter Server Appliance
    -   /var/lib/vmware/vsphere-client

Edit the file to include the line sso.enabled = false.
Restart the vSphere Web Client service.

On Windows operating systems, restart the VMware vSphere Web Client
service.
On the vCenter Server Appliance, restart the vSphere-client service by
typing service vsphere-client restart.
After you have restarted the vSphere Web Client, you can select the
vCenter Server 5.0 system to log in to from the login page.
What to do next
To return the vSphere Web Client to using vCenter Single Sign On, edit
the webclient.properties file to remove the sso.enabled = false line,
and restart the vSphere Web Client service.

Enjoy!
