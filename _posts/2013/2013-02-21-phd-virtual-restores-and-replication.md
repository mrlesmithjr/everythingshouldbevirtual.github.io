---
  title: PHD Virtual Restores and Replication
  date: 2013-02-21 16:33:13
---

In this article we will be going through the process of using PHD
Virtual for doing file level restores, whole vm restores and
replications. The initial installation of PHD Virtual can be found
[here](https://everythingshouldbevirtual.com/installing-phd-virtual-backup-and-replication "https\://everythingshouldbevirtual.com/installing-phd-virtual-backup-and-replication") in
which we also created our first backup job. We will be using this to do
our first restore.

**File Level Restores**

Right click on any object and under PHD Virtual Backup select console.
Now follow the screenshots below to step you through this process. This
will basically be creating a CIFS share that is browseable and contains
the contents of the vmdk files of the server. In this case we are doing
an FLR on a Windows 2008 server.

Now you will go into File Recovery in the console and open the share and
browse into the disks and partitions and then into your folder structure
to find the files that you want to restore. You basically have to copy
the files/folders out of here to wherever you want to restore them to.
:( Not ideal, but I guess it works for some.

**Replications**

Right click on any object and under PHD Virtual Backup select console.
Go to configuration and then replication.

Now we will configure the appliance to do replications. You will first
need to configure the storage used for backups which in this case is the
local attached disk on the appliance.

Follow the screenshots below.

Now go to the replication tab on the left.

Click "Virtual Machines Available For Replication". This will list all
of your vms that are already backed up and ready to be configured for
replication. Choose the vms you want to replicate. Select the datastore
that you want to use for the replication. Select the default network to
use for the target (NOTE!!! If you are using vDS (Distributed Switches)
you will be presented with the error you will see in the screenshots.
You can still go forward, but keep in mind that you will need to
configure your replicated vm network settings when you are ready to
power up the vm, which will also require you to add a network adapter
because it does not add one. Even though you can get around this a word
of caution would be that if you are in a true disaster and you have
100's - 1000's of vm's you have a lot of work ahead of you. I truly
hope they fix this in upcoming releases.) Continue on and configure the
remaining screens for your replication configuration and then kick it
off.

Follow the screenshots below for the walk through of this section.

**Replications - Testing**

Now that we have our vms replicated you will want to test and verify
that it works right? Of course.

Right click on any object and under PHD Virtual Backup select console.
Go to replication and then "Replicated Virtual Machines", select the
vm you want to test and click start test. Now this will not
automatically power on the vm :(. You will need to go into vcenter and
manually power it on. And it worked. :)

Screenshots below of the process.

Now for the real fun!!! :)

The one thing we have not tested yet is recovering a complete vm from
backup. So let's do this.

**Full VM Recovery**

First thing I want to do is delete the original vm that was backed up in
our backup scenario. Then I am going to recover it. Remember if using
vDS you will need to add a network adapter to your recovered vm.

So here are the screenshots of this scenario.

AND IT WORKED !!! :) :)

**Ending thoughts**

I feel that PHD Virtual does a really good job at what it is meant for.
However there are some areas that need some tidying up, but I am sure
that these will be addressed in future releases. I look forward to doing
some additional testing of this product in the future for sure. Again
keep in mind this product is about half the cost of another product, but
YMMV. :)

Enjoy!!
