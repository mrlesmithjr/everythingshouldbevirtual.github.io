---
  title: Nexentastor/ESXi5/3750G/LACP/VDS/NFS/iSCSI - Part 2
  date: 2012-12-05 21:30:00
---

This article is a continuation of Part 1 that we did a while back.  I
would highly suggest checking out that article as well prior to going
through this article.  You can read Part 1 [here](https://everythingshouldbevirtual.com/nexentastoresxi53750glacpvdsnfsiscsi-part-1/).

# MPIO Round Robin Policy

The default round robin policy for each iSCSI LUN presented is set to
1000 IOPS, which means that for every 1000 IOPS the round robin policy
will switch NIC paths.  We are going to change the IOPS to 1.  Then it
will switch paths every 1 IOP.  Therefore utilizing our paths more
effectively.

The first thing we need to do is get the list of devices that we need to
make this change to by executing the command below. On your ESXi console
via SSH run the following command.

```bash
esxcli storage nmp device list | grep naa | grep NEXENTA
   Device Display Name: NEXENTA iSCSI Disk (naa.600144f03eb7cd0000004ffd9d940001)
   Device Display Name: NEXENTA iSCSI Disk (naa.600144f03eb7cd000000500d85a40003)
   Device Display Name: NEXENTA iSCSI Disk (naa.600144f03eb7cd000000501319fa0001)
   Device Display Name: NEXENTA iSCSI Disk (naa.600144f03eb7cd000000500b500a0002)
   Device Display Name: NEXENTA iSCSI Disk (naa.600144f03eb7cd0000005058b14b0001)
```

As you can see we have 5 Nexenta devices that we need to modify the
policy for here.  The device names that we need to change are within
parenthesis starting with **_naa_**.

Run the following command and you will notice that the IOOperation Limit
is 1000 and the Limit Type is Default

```bash
esxcli storage nmp psp roundrobin deviceconfig get -d naa.600144f03eb7cd000000500d85a40003    Byte Limit: 10485760     Device: naa.600144f03eb7cd000000500d85a40003
   IOOperation Limit: 1000
   Limit Type: Default
   Use Active Unoptimized Paths: false
```

Now to change the IOPS policy execute the following for each device
name.  Also make sure that for each datastore or LUN using the vSphere
client that Round-Robin is selected for the Load Balancing policy.

```bash
esxcli storage nmp psp roundrobin deviceconfig set -d naa.600144f03eb7cd0000004ffd9d940001 --iops 1 --type iops
```

Now do this same thing for each additional device you listed from above
by replacing the naa.\_\_\_\_ device name.  Also remember to do this on
each host that uses these same iSCSI devices.

Once you are done we need to make sure that all of the devices have been
changed.  Run the following command to verify quickly.

```bash
# esxcli storage nmp device list | grep policy
   Path Selection Policy Device Config: {policy=iops,iops=1,bytes=10485760,useANO=0;lastPathIndex=0: NumIOsPending=0,numBytesPending=0}
   Path Selection Policy Device Config: {policy=rr,iops=1000,bytes=10485760,useANO=0;lastPathIndex=0: NumIOsPending=0,numBytesPending=0}
   Path Selection Policy Device Config: {policy=iops,iops=1,bytes=10485760,useANO=0;lastPathIndex=0: NumIOsPending=0,numBytesPending=0}
   Path Selection Policy Device Config: {policy=iops,iops=1,bytes=10485760,useANO=0;lastPathIndex=0: NumIOsPending=0,numBytesPending=0}
   Path Selection Policy Device Config: {policy=iops,iops=1,bytes=10485760,useANO=0;lastPathIndex=0: NumIOsPending=0,numBytesPending=0}
```

As you can see from above highlighted policy and iops values we missed
one of our devices.  So you will need to go back and make sure to change
that device to the new values using the commands from above.  Also if
you run the same command to list our policies that are set you will
notice that the lastPathIndex= will change which means that our paths
are definitely changing.

Remember your device names will be different than the examples above.

When configuring a folder share to export for NFS as a VMware datastore
configure the following settings for the folder as in the screenshot.

[15-52-13](../../assets/15-52-13-296x300.png)

You can reference [this](http://info.nexenta.com/rs/nexenta/images/5000-nxs-v0.0-000002-A_nxstor_vmware_best_practices.pdf) doc for more information on Nexenta's recommended best practices.

You can also reference the VMware best practices guide for NFS
datastores [here](http://www.vmware.com/files/pdf/techpaper/VMware-NFS-Best-Practices-WP-EN-New.pdf "http\://www.vmware.com/files/pdf/techpaper/VMware-NFS-Best-Practices-WP-EN-New.pdf").
