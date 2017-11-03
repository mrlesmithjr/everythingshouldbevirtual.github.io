---
  title: vSphere 5.5 - Using vCenter Server Appliance
  date: 2013-10-21 10:10:19
---

I have been wanting to give the vCSA (vCenter Server Appliance) a go in
my main lab for quite some time but never got around to it. Well I
finally pulled the trigger with the latest 5.5 release. There was
however quite a bit of pre-work that I had to do in preparation of
making this move but all in all it went very smooth. The biggest thing
was to migrate all of my vSwitches from vDS (vSphere Distributed
Switches) to vSS (vSphere Standard Switches). Which was not too bad just
a little time consuming. However if you are a PowerCLI person this could
be really quick and easy.

So after getting all VMKernel ports and guest networks migrated from vDS
over to their corresponding vSS port groups. I was ready to proceed on.
The next step was to remove all vDS configurations form all of the hosts
by removing each host from the vDS groups (I did not want to leave any
remnants of the old configurations on the hosts). After this was done I
could then remove each host from my current Windows vCenter server. Now
I have 3 vSphere 5.1 hosts ready to be added to my new vCSA 5.5 and
start over from scratch. So I then proceeded on to deploy the vCSA by
connecting to one of my hosts and deploying a new virtual appliance and
powering it up and getting it configured. If you need a look at how to
set this up you can head over to
[this](https://everythingshouldbevirtual.com/deploying-vmware-vcenter-server-appliance-5-1 "http\://everythingshouldbevirtual.com/deploying-vmware-vcenter-server-appliance-5-1")
post and get a quick understanding of doing this for a vCSA 5.1 setup.
The process is pretty much the same with the new vCSA 5.5 so it should
work for you.

Now that I have my new vCSA up and running I can then begin adding my
hosts back to the new vCenter and continue on with creating a new
Datacenter, HA-DRS Cluster and building out the new vDS configurations.
Now if you do this methodically you _should_ not experience an outage at
all. This also assumes that you have plenty of pNICS to accomplish this
but if you were able to migrate from your previous vDS back to vSS
without an outage then you should be good by just reversing the same
process.

So overall this was a pretty seamless process but it did take some
planning and time. But the end result was a new shiny Linux based vCSA
(vCenter). I would like to figure out a way to migrate all of the
configurations from a Windows based vCenter over to a Linux based
vCenter but I did not come up with anything while I was researching this
but I am sure it can be done. The good thing is that with using the vCSA
all future upgrades you are able to do this and the good thing is that
it only takes about 5-10 minutes to deploy, configure and have a vCenter
instance running by using the vCSA.

New features of the vCSA 5.5 compared to the 5.1 vCSA.

-   5 Hosts (5.1) / 500 Hosts (5.5)

-   50 VMs (5.1) / 5000 VMs (5.5)

The new web ui is now HTML 5 which is much faster and cleaner than the
previous version. There is also a web plugin for VUM (vSphere Update
Manager).

Download the vCSA 5.5 from
[here](https://my.vmware.com/group/vmware/details?downloadGroup=VC550&productId=353&rPId=4283 "https\://my.vmware.com/group/vmware/details?downloadGroup=VC550&productId=353&rPId=4283").

Enjoy!
