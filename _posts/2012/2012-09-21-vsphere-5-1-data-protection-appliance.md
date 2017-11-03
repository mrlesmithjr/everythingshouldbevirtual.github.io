---
  title: vSphere 5.1 Data Protection Appliance
  date: 2012-09-21 16:19:01
---

I just installed and configured the new vSphere 5.1 VDP appliance. Very
easy installation and integrates well and works very well. But of course
I am now looking at the fact that I am capped at 2TB limit and I have
some servers that have nearly 16TB of storage. That is a problem of
course. From what I have found so far it is possible to run up to 10 VDP
appliances per vCenter Server. That is great, but how can I spread the
data across appliances. :) Now knowing that the builtin dedupe will help
on most cases this still looks like this is the catch to possibly engage
EMC to get a full blown version of [avamar](http://www.emc.com/backup-and-recovery/avamar/avamar.htm "http\://www.emc.com/backup-and-recovery/avamar/avamar.htm").
:) How nice is that. :) Maybe they will change the 2TB limit going
forward? Also I do understand that this is a solution that would be
useful for SMB or even a home lab. But even my home lab could outgrow
that very quick. :) I am not hating here by any means. I still think
this looks like a great solution which is included for us with vSphere
5.1, but it still makes [Veeam](http://www.veeam.com "http\://www.veeam.com") and
[PHDVirtual](http://www.phdvirtual.com/ "http\://www.phdvirtual.com/")
look appealing. But hey this is our choice and this is what makes this
stuff interesting. I will however still mess around with the VDP and see
how well it works over time.
