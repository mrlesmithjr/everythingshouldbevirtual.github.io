---
  title: Create and Migrate Standard vSwitches to Distributed Switches in vSphere 5.1
  date: 2012-11-13 16:54:16
---

In this scenario we will be migrating existing standard vswitches to
distributed switches. This scenario also includes iSCSI vmkernel ports
which we will be moving as well. There is no audio in this video so it
is just video only. I will be including audio in future videos. Make
sure to change the video quality to auto or hd.

Quick rundown of what is happening is we are going to remove 1 pnic from
each vswitch initially to create our new distributed switch in order to
migrate to. This will include the console, vm network and iSCSI ports.
We will be unbinding the iSCSI ports from the iSCSI software adapter,
which will cause us to lose connection to our iSCSI datastores so make
sure you do this on your hosts one at a time in maintenance mode if you
have workloads running. Or you will cause major issues.

<https://youtu.be/wsFuMXHSa2s>
