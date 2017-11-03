---
  title: Adding Veeam v7 vCenter Plugin
  date: 2013-10-09 15:26:52
---

As I have been testing out vSphere 5.5 and Veeam v7 the past week or so.
I wanted to go ahead and check out the new vCenter webui plugin for
Veeam v7 so I figured I would throw this together for others in case you
are looking for this info. Overall this plugin is fairly easy to install
and use which is great.

This plugin is actually installed from Veeam Backup Enterprise Manager
so you will need to make sure that you have this installed first. The
reason for this is that the vCenter webui plugin for Veeam links and
integrates with Veeam Backup Enterprise Manager.

So launch your browser of choice and connect to
<https://{enterprisemanagerhostname/ip}:9443> and go to the configuration
tab and vCenter Servers.

![13-10-18](../../assets/13-10-18-300x120.png)

Select your vCenter server that you want to install the plugin to and
click the install button and then enter your account information that
has administrative permissions in vCenter.

![13-10-47](../../assets/13-10-47-300x123.png)

Now click OK and off it goes.

![13-11-48](../../assets/13-11-48-300x129.png)

All done.

![13-12-06](../../assets/13-12-06-300x123.png)

Now using your browser of choice and connect to your vCenter webui
<https://{vcenterhostname/ip:9443}> and login using an account that has
access to vCenter and also a member of the Administrators group on your
Veeam Enterprise Manager Server otherwise you will not be able to view
the plugin.

![13-13-15](../../assets/13-13-15-300x255.png)

Once logged in you should see the Veeam Backup & Replication plugin
listed on the left.

![13-22-40](../../assets/13-22-40-300x139.png)

![13-23-39](../../assets/13-23-39-300x172.png)

![13-25-39](../../assets/13-25-39-300x123.png)

So there you have it. The new Veeam Backup & Replication v7 plugin for
vCenter webui is now installed and usable. Unless you have Veeam One
installed as well the capacity planning link will not work for you.
However the other links for Protected VMs Report and Latest Backup Job
Status Report will work. This will link back to your Veeam Enterprise
Manager server which provides a ton of information as well as being able
to restore VMs.

Enjoy!
