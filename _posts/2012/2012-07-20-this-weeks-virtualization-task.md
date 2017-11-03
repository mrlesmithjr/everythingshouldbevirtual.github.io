---
  title: This weeks virtualization task
  date: 2012-07-20 15:23:32
---

Currently working on designing a stretched cluster that will include
vMSC (vSphere Metro Storage Cluster). We are currently designing around
OTV between 2 sites with Cisco Nexus 7k's at each site. We will be
deploying several HP Blade servers in a C7000 chassis using Flex-10 with
10GB uplinks into Cisco Nexus 5k switches locally at each site. Our WAN
link initial sizing would be a 2GB WAN link. Storage is still an open
item. I would like to lean on using HP Lefthand P4000 storage at each
site. Build a network Raid-10 storage cluster with half of the storage
at each site using iSCSI and connecting to the VIP of the storage
cluster, also 10GB connections local at each site. More to come later
