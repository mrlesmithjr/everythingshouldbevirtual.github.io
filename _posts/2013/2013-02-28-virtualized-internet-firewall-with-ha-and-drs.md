---
  title: Virtualized Internet Firewall With HA and DRS
  date: 2013-02-28 11:54:05
---

I wanted to throw this together real quick and explain how to accomplish
running a virtualized firewall (PFSense, Astaro, etc.). The goal to
accomplish here is to use your cable/dsl internet connection with the
ability to vmotion across VMware hosts and not lose your internet
connectivity. :) So the first thing you will need to have on hand is a
switch that you can configure VLANs on. You will need to either create a
VLAN in which your hosts will have dedicated nics connected into or
create port trunks that contain that VLAN as well as any other VLAN you
use in your environment. Then create your vm network on top of either
your vSwitches or dvSwitches with the VLAN ID you created for this
scenario. This vm network connection is what you will use for your WAN
side connection for your virtualized firewall. Then all that is left is
to connect your LAN side port of your cable/dsl modem into a port on
your switch which is also set to the VLAN ID you created. Now your
firewall can vmotion across hosts and it will broadcast out over the
VLAN and establish your internet connection. This setup works great and
I have been doing this for years so now you can too enjoy it. :)

Enjoy!

Leave any comments, suggestions or your other creative ways of doing
this as well.
