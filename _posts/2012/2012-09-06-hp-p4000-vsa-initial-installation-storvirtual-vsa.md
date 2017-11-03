---
  title: HP P4000 VSA initial installation - StorVirtual VSA
  date: 2012-09-06 13:35:42
---

So I am getting settled after VMworld and thought I would attempt to
install the HP VSA that was being shown there. Well to say the least it
did not go well. Not sure what the issue is with the installer, but I
also attempted to run the setup as administrator (Win7 - :)  ) Below are
the snapshots from the initial install. It failed both using the command
line and gui as you can see from the screenshots below. After this
installation guide I plan on writing up some additional guides on how to
do basic configurations and some advanced configurations in the future.
I do use P4000 (HP Lefthand) SAN solutions on a day to day basis so I
will be able to provide some good stuff.

So to continue on with this I have a workaround. I will be installing
the VSA to run under VMware Workstation. You can also deploy the ovf to
ESX(i) through vCenter.

You first need to download the HP VSA. It can be downloaded from the
following link. [www.hp.com/go/tryvsa](http://www.hp.com/go/tryvsa)

After the download completes you will need to extract the zip file using
7-Zip or another unzip utility.

![](../../assets/16-47-45-300x115.png "16-47-45")

Browse to the following folder
"_HP_P4000_VSA_9.5_Full_Evaluation_SW_for_Vmware_ESX_requires_ESX_servers_AX696-10536\\Virtual_SAN_Appliance_Trial\\Virtual_SAN_Appliance\\vsatrial_"
and extract the files from within setup.exe.

![](../../assets/16-51-50-300x161.png "16-51-50")

Now within the following folder
"_HP_P4000_VSA_9.5_Full_Evaluation_SW_for_Vmware_ESX_requires_ESX_servers_AX696-10536\\Virtual_SAN_Appliance_Trial\\Virtual_SAN_Appliance\\vsatrial\\setup\\VSA_OVF_9.5.00.1215_"
double click on VSA.ovf

_YOU MUST_ be running VMware Workstation to complete this method.

![](../../assets/16-56-03-300x68.png "16-56-03")

This process will import into VMware workstation and you will be
presented with the following screens.

![](../../assets/16-58-30-300x199.png "16-58-30")

![](../../assets/17-04-42-300x244.png "17-04-42")

Once this completes you will have successfully imported the VSA. The
next step is to add some new virtual disks to the VSA. You can add as
many disks as you would like to, but you will need to change the Virtual
SCSI ID to start at 1:0, 1:1, 1:2 and so on. These new disks will be for
the data volume for the VSA, which we will configure here shortly.

Next thing is to install the HP CMC (Centralized Management Center).
Browse to the following folder
"_HP_P4000_VSA_9.5_Full_Evaluation_SW_for_Vmware_ESX_requires_ESX_servers_AX696-10536\\Virtual_SAN_Appliance_Trial\\Virtual_SAN_Appliance\\vsatrial\\setup\\CMC_Installer_"
and launch _CMC_9.5.00.1215_Installer.exe,_ this will install HP CMC.
Below are some screenshots of the installation.

![](../../assets/16-24-37-300x183.png "16-24-37")

![](../../assets/16-24-48-300x205.png "16-24-48")

![](../../assets/16-24-57-300x211.png "16-24-57")

![](../../assets/16-25-40-300x216.png "16-25-40")

![](../../assets/16-26-02-300x217.png "16-26-02")

![](../../assets/16-26-16-300x214.png "16-26-16")

![](../../assets/16-26-31-300x216.png "16-26-31")

![](../../assets/16-26-46-300x214.png "16-26-46")

![](../../assets/16-27-00-300x217.png "16-27-00")

![](../../assets/16-27-08-300x216.png "16-27-08")

![](../../assets/16-27-21-300x155.png "16-27-21")

After the CMC installation has completed we need to configure the VSA.

Edit the settings for the VSA as follows.

![](../../assets/08-35-35-300x176.png "08-35-35")

Now click add

![](../../assets/08-38-13-300x215.png "08-38-13")

Select hard disk and next...

![](../../assets/08-39-16-300x220.png "08-39-16")

Select create a new virtual disk and next...

![](../../assets/08-39-57-300x217.png "08-39-57")

Select SCSI and next...

![](../../assets/08-41-21-300x221.png "08-41-21")

Now enter the size of the disk you want to use and select store virtual
disk as a single file and next...

![](../../assets/08-44-02-300x221.png "08-44-02")

Click finish...

![](../../assets/08-48-38-300x223.png "08-48-38")

Now we need to change the SCSI ID of the new disk so that the VSA will
use it for virtual raid...

Highlight the new virtual disk, click advanced and in the drop down
select 1:0...

If you want to add more virtual disks make sure to change their SCSI ID
to 1:1, 1:2, etc. But for testing one virtual disk is fine.

![](../../assets/08-51-45-300x234.png "08-51-45")

Once you are complete with adding the virtual disks power on the VSA. It
will boot up and stop at a logon prompt.

![](../../assets/16-29-25-300x171.png "16-29-25")

Type start to login...

![](../../assets/16-29-40-300x172.png "16-29-40")

Hit enter on login screen...

![](../../assets/16-30-10-300x173.png "16-30-10")

![](../../assets/16-30-20-300x169.png "16-30-20")

Scroll down to Network TCP/IP Settings...

![](../../assets/16-30-35-300x171.png "16-30-35")

Hit enter...

![](../../assets/16-30-51-300x170.png "16-30-51")

Select eth0 and hit enter...

![](../../assets/16-31-09-300x172.png "16-31-09")

Enter a hostname and either select DHCP or enter a static IP. For this
walkthrough I am going to use DHCP...

![](../../assets/16-31-30-300x170.png "16-31-30")

\*\*Tab through fields then hit enter on ok...and enter on the next
popup...\*\*

You will now get a screen showing the IP address that was
obtained...Note this IP to add to CMC...

![](../../assets/13-14-53-300x171.png "13-14-53")

Now launch HP CMC if it is not already running and we are going to add
the VSA to CMC...

With CMC open select find, find systems, add...Enter the IP address
from the VSA screen above that you were given and then click ok and
close...

![](../../assets/13-19-05-300x207.png "13-19-05")

You should now see your new VSA listed under Available Systems...

![](../../assets/13-23-02-300x101.png "13-23-02")

Now double click on the VSA and expand the item and click on storage...
Verify that Raid Status is green and Normal...

![](../../assets/13-26-47-300x81.png "13-26-47")

If all looks good you are now ready to start using your HP P4000 VSA
(StorVirtual VSA)..

ENJOY!!!

And check back later for more information on configuration and use of
the VSA...
