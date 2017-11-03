---
  title: How to install block based tools for platespin workloads manually
  date: 2012-08-22 10:08:43
---

Here is how you can control a reboot required for adding a server
workload to platespin which uses block based replication. Highly suggest
only using block based for DB servers or other servers that have a lot
of data, but maybe only a bit of changed data. All other workloads (Web,
App) should be file based. Below are the contents of KB article
**[7010094](http://www.novell.com/support/kb/doc.php?id=7010094) **which
was not working at the time I am posting this, but I have the contents.
Why would you want to do this? Well if you have a lot of workloads then
when you schedule downtime for a production DB server but have to
explain that it could take up to x number of hours for it to get
rebooted, but have no idea what x would be that doesn't work real well.
So schedule your production outage on an agreed upon time and manually
install the BBT (Block Based Tools) and then reboot the server at your
own will. Now after the server has been rebooted you can add the
workload to your platespin server and it will not require another reboot
because the tools will be detected.

\* **\*\*\*\*Disclaimer\*\*\*\* - The information provided below was
copied from Novell's site for Platespin support.\***

# How to manually install the PlateSpin Block-Based driver onto a Windows source workload

This document **(7010094)** _is provided subject to the
[disclaimer](https://kcs.innerweb.novell.com/contactcenter/viewContent.do?externalId=7010094&sliceId=1#disclaimer)
at the end of this document._

### Environment

PlateSpin Forge

PlateSpin Protect

Source workload is not a member of a Windows cluster

### Situation

After removing the block-based components manually or if the block-based
components need to be installed prior to adding a workload to PlateSpin
Forge or PlateSpin Protect, it is necessary to follow the steps outlined
in resolution section to ensure the block-based components are installed
correctly.

### Resolution

**If using Protect 10.1/Forge 3.1 or later**:

1. Download  and extract the platespin_bbt_install.zip file to the
source workload

2. Navigate to the extracted directory via command line and run this
command:

ProtectAgent.cli.exe /bbinstall

3. Reboot the source workload

**If using Protect 10.0.2/Forge 3.0.2 or earlier**:

1. Copy the PSMoninstall.zip file to the source workload.

2. Copy the appropriate .inf and .sys files for the source Operating
System by following these steps:

2a. Navigate to the following folder on the PlateSpin Protect server or
PlateSpin Forge Management VM:

..\\PlateSpin Protect
Server\\Packages\\C221392A-D69A-417C-B359-EBEBEFBBFBA0

D:\\Program Files\\PlateSpin Forge
Server\\Packages\\C221392A-D69A-417C-B359-EBEBEFBBFBA0

2b. Copy and paste 1.package.

2c. Rename the copy of 1.package to 1.zip and extract it

2d. Copy the files under 64bit or 32bit to the source workload

3. If the Microsoft VC++ Redistributable package is not installed on the
source workload, either install it or manually install the controller as
per TID
[7008520](http://www.novell.com/support/documentLink.do?externalID=7008520)

4. Remove any .bbvt files (if block driver was ever installed on this
machine) found on the root of every protect source volume (e.g.
C:\\platespin.bitmap.bbvt).

5. Extract psmoninstall.zip on the source. Move the psmon.sys and
psmon.inf files copied in step 2 to the extracted folder. If the
Microsoft VC++ Redistributable package is not installed, the extracted
folder shoould instead be the ..\\%PlateSpin Server Folder%\\Controller
folder.

6. Via command line, run the following command:

..\\extracted PsMonInstall folder\\PsMonInstall.exe /inf "..\\extracted
PsMonInstall folder\\psmon.inf" /install psmon /add

7. Reboot the source workload.

[Platespin BBT
Installs](http://everythingshouldbevirtual.com/wp-content/uploads/2012/08/Platespin-BBT-Installs.zip)
