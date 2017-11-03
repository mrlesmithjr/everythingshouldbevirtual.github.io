---
  title: Remove Platespin Block Based Drivers
  date: 2012-10-05 11:22:09
---

**Remove Platespin Block-Based drivers**
In order to increase a drive size you must first remove the block-based
drivers and reboot. Once rebooted you will be able to increase the
capacity of a disk. After doing so you will have to go to the platespin
server and remove the workload and then add it back.
Below is the link on removing these drivers.

Go [here](http://www.novell.com/support/php/search.do?cmd=displayKC&docType=kc&externalId=7005616&sliceId=1&docTypeID=DT_TID_1_1&dialogID=118906851&stateId=0%200%20242563522 "http\://www.novell.com/support/php/search.do?cmd=displayKC&docType=kc&externalId=7005616&sliceId=1&docTypeID=DT_TID_1_1&dialogID=118906851&stateId=0%200%20242563522"). [](http://www.novell.com/support/php/search.do?cmd=displayKC&docType=kc&externalId=7005616&sliceId=1&docTypeID=DT_TID_1_1&dialogID=118906851&stateId=0%200%20242563522)

The link is not working at this time.
So here is what to do.

Download and extract the Devcon.exe from
Microsoft [KB 311272](http://support.microsoft.com/kb/311272).

From a command prompt, execute:

```powershell
devcon stack storage\volume
```

Below is an example of the output. PsMon will be listed as a upper
filter.

```powershell
STORAGE\\VOLUME\\1&30A96598&0&SIGNATURE11781178OFFSET7E00LENGTH63F3DFE00\
Name: Generic volume\
Setup Class: {71A27CDD-812A-11D0-BEC7-08002BE2092F} Volume\
Class upper filters:\
Upper filters:\
psmon\
Controlling service:\
volsnap\
1 matching device(s) found
```

Download and extract [PsMonInstall.zip](http://everythingshouldbevirtual.com/wp-content/uploads/2012/08/Platespin-BBT-Installs.zip "http\://everythingshouldbevirtual.com/wp-content/uploads/2012/08/Platespin-BBT-Installs.zip")

Here is the command to run to remove the Platespin tools.

```powershell
PsMonInstall.exe /inf c:\windows\inf\psmon.inf /uninstall psmon /remove
```

Now reboot the workload and make sure that each drive does not contain
any files like platespin.bitmap.bbvt in the root of the drive. If they
do exist delete them.

Now if you need/want to install the Platespin tools manually head over
to [this](http://everythingshouldbevirtual.com/how-to-install-block-based-tools-for-platespin-workloads-manually "http\://everythingshouldbevirtual.com/how-to-install-block-based-tools-for-platespin-workloads-manually")link on how to do that.

Enjoy!
