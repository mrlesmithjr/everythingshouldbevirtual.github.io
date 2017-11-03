---
  title: How I like to build 'Em - Vsphere5 - What ya think?
  date: 2012-08-23 18:12:51
---

Just wanted to throw this together as my current environment goes a
little like this. HP C7000, Flex-10 configured with two SUS Groups
(Shared Uplink Sets) to carve bandwidth the way I need it into the 8
Flex-10 LOMS on each Blade, 10GB Fiber uplinks to Cisco Nexus 5k (x2)
which uplinks to a Cisco Nexus 7K Core (x2), 4/8GB Virtual Connect FC
connecting to Cisco MDS9509 (x2), BL490G7's (2CPU,6-core/each), 200GB
Ram, Hypervisor boots from internal USB 8GB Stick, VCEM (Virtual Connect
Enterprise Manager) managed Virtual Connect domain (Move those profiles
around), and all of this connecting to a brand new IBM XIV Gen-3
(FAST!!!), Storage Cluster built in vCenter with SDRS and SIOC enabled,
IBM VASA Storage Provider and IBM vSMC. Vcenter5 virtualized on non-VDS
VLAN (I Know....I have done both...I keep going back to this! :) ).
All vm networks configured on VDS with "Route Based on Physical NIC
Load" (LBT \*\*\*Only available with Enterprise+ license) with NIOC
enabled. This setup is rocking for us....

How can it get better? That will be decided after next week at VMworld.
:) Definitely looking forward to it....One thing I know is BL490's are
going away with Gen8 so I will be using BL460 Gen8's. These blades kick
it big-time.

Any feedback would be welcomed for sure. "It can always get better!"
So I definitely look forward to others opinions.
