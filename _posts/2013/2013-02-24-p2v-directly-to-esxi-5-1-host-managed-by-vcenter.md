---
  title: P2V Directly to ESXi 5.1 Host Managed by vCenter
  date: 2013-02-24 20:03:31
---

Recently I had a need to perform a P2V directly to an ESXi 5.1 host and
found that I could not do this because it would fail with the error
"The access to this host resource is restricted to the server
'x.x.x.x' that is managing the host". The failure was caused because
using the latest VMware standalone converter 5.0.1 and trying to connect
directly to an ESXi5.1 host managed by vCenter requires you to select
the vCenter server as your target. Well you may be asking well why
didn't you just do that. :) The issue was that the subnet that location
I was at did not have network access back to where the vCenter server
was, but I had access directly to the host which was in a remote site. I
was shocked because I had done this many times in the past but not with
an ESXi 5.x host. But the good thing is that it is fairly simple to get
around however there is a small catch which is that if you only have a 2
node cluster for your target host then you will be in effect disabling
HA functionality.

I decided to do this video real quick to show what the scenario would
look like and how to get around it.

Enjoy!

<https://youtu.be/Op-JZqViGnM>
