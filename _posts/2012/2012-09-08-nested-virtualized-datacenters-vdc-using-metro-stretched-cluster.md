---
  title: Nested Virtualized Datacenters (VDC) using metro stretched cluster
  date: 2012-09-08 09:56:51
---

I am currently working on building a 2 datacenter scenario all running
under VMware workstation 9. This will include a four node HP P4000 VSA
multi-site cluster, four ESXi5 hosts, vCenter, and two PFsense firewalls
using openvpn to connect both sites. I am sure I will be adding more as
I go through this setup. I plan to build this and have it to take with
me on my laptop for demo purposes. I am hoping for the best. :) So far I
have the PFsense firewalls working with an openVPN tunnel built and all
routing working. The four node HP P4000 VSA multi-site cluster is built
and running. :) Ran out of space on my desktop building this, forgot
that I was running my desktop on small partition running Ubuntu 12.04,
but fixed that issue once I remember I have TB's of storage available
on my NexentaStor NAS, so I just mounted up some NFS storage and moved
all of the VM's I am using for this scenario over to that. I will be
adding some load balancers to the mix too and I am debating on testing
the vCenter virtual appliance during this too just to see how that goes.
Anyways stay tuned as I go through this build-out. And if anyone else
has done this that reads this share your thoughts with me.
