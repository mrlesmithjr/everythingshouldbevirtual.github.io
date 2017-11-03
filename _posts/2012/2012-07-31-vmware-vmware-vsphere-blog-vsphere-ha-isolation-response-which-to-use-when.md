---
  title: VMware: VMware vSphere Blog: vSphere HA isolation response... which to use when?
  date: 2012-07-31 13:15:14
---

Good info on the different isolation modes and recommendations on which
one to use based on your infrastructure. vSphere4 the default policy in
an HA cluster was to power off. The default in vSphere5 is to leave
powered on. I have mixed opinions on this, but do agree just because
your host lost management communications does not mean you would want to
power off all of the vms running on that host. However if they do get
powered off the negotiations of a downtime window do not have to be
worked around. :) But wonder if one of those vm's was your exchange
server? You be the judge.

[VMware: VMware vSphere Blog: vSphere HA isolation response... which to use when?](http://blogs.vmware.com/vsphere/2012/07/vsphere-ha-isolation-response-which-to-use-when.html).
