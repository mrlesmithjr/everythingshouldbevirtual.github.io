---
  title: Getting Started with vSAN in vSphere 5.5
  date: 2013-11-04 07:00:13
---

As you may or may not know VMware announced the new vSAN with vSphere
5.5. vSAN is currently in public beta but it is very easy to get started
testing it out in your new vSphere 5.5 environment. First thing you
should do is head over
[here](http://www.vmware.com/vsan-beta-register.html "http\://www.vmware.com/vsan-beta-register.html")
and get signed up to get your license key which is required to use vSAN.
All functionality is already built into vSphere 5.5 but you will need to
enter a license key.

So to get you started I wanted to throw this post together so you can
start testing vSAN out. There are a few things you must do in order to
get vSAN running properly which I will go over in this post. Just an
FYI, all vSAN options and functionality are only visible in the web ui
so the traditional C# viclient will not work.

Now head over to your vCenter web ui using your browser of choice and
let's get started.

First thing you must do is head over to your HA settings for your
cluster and disable HA. You will not be able to enable vSAN when HA is
enabled and will get an error stating so.

![14-45-51](../../assets/14-45-51-300x176.png)

Next you will need to create a new VMKernel port for vSAN
communications. After creating this port all HA communications within
your cluster will function over this same VMKernel port once HA is
enabled again.

I am using VDS (vSphere Distributed Switch) in my setup so these are
what screenshots will represent this setup.

Create a new VDS Portgroup in my case it will be called vSAN Network.

![15-14-05](../../assets/15-14-05-300x144.png)

Now create new VMKernel ports for each of your hosts in your cluster
using this new Portgroup.

![15-20-37](../../assets/15-20-37-300x93.png)

![15-21-05](../../assets/15-21-05-300x175.png)

Select an existing distributed port group and choose the vSAN Network.

![15-21-36](../../assets/15-21-36-300x225.png)

![15-21-47](../../assets/15-21-47-300x175.png)

Make sure to select Virtual SAN Traffic.

![15-22-16](../../assets/15-22-16-300x175.png)

![15-22-26](../../assets/15-22-26-300x175.png)

![15-22-37](../../assets/15-22-37-300x174.png)

You will now see your new VMKernel port which is VMK4 in my case.

![15-22-51](../../assets/15-22-51-300x155.png)

Now after doing this for each of your hosts in your cluster we will now
enable vSAN. Make sure you have your license key handy as well for the
next step.

Browse to your cluster settings and add your license key for vSAN under
Virtual SAN Licensing.

![15-08-04](../../assets/15-08-04-300x151.png)

![15-08-14](../../assets/15-08-14-300x159.png)

![15-08-43](../../assets/15-08-43-300x157.png)

![15-09-18](../../assets/15-09-18-300x152.png)

Now let's enable vSAN.

![14-46-19](../../assets/14-46-19-300x151.png)

![14-46-33](../../assets/14-46-33-300x172.png)

Select Auto or Manual for adding disks per host. I selected Automatic
but went ahead and manually assigned at the end of this so you can see
that as well.

![14-46-51](../../assets/14-46-51-300x172.png)

![14-47-34](../../assets/14-47-34-300x173.png)

vSAN is now enabled for your cluster.

Now go to Disk Management and we will now claim the disks to be used in
our vSAN setup.

![14-52-18](../../assets/14-52-18-300x161.png)

![14-52-36](../../assets/14-52-36-300x250.png)

![14-52-46](../../assets/14-52-46-300x250.png)

![15-10-44](../../assets/15-10-44-300x169.png)

Now if you browse to your Datastores view you will now see a new
Datastore called vsanDatastore.

![15-12-06](../../assets/15-12-06-300x90.png)

![15-12-17](../../assets/15-12-17-300x134.png)

So now the last thing to do before using your new vSAN setup is to
enable HA for your cluster seeing as we disabled it at the very
beginning.

![15-23-41](../../assets/15-23-41-300x115.png)

![15-23-57](../../assets/15-23-57-300x175.png)

![15-24-05](../../assets/15-24-05-300x174.png)

And there you have it. vSAN is up and running and has presented us with
a new Datastore that you can start using just as any other Datastore.
You can call also get into different VM Storage Policies as well once
you start getting familiar with vSAN in order to set different policies
for each scenario. If you need info on the new VM Storage Policies in
vSphere 5.5 head over [here](https://everythingshouldbevirtual.com/vsphere-5-5-storage-profiles-now-storage-policies "http\://everythingshouldbevirtual.com/vsphere-5-5-storage-profiles-now-storage-policies").

Enjoy!
