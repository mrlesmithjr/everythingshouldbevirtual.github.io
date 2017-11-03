---
  title: My Home Lab NexentaStor CE NAS
  date: 2012-08-29 03:12:53
---

Just wanted to post some of these pics real quick. I took them over a
year ago I think. Just wanted to show it. It has 12TB of usable storage.
One ZPool with 7 mirrored vDev's and 1 hot spare. Dual-Core AMD, 16GB
of memory, 2 Supermicro SAS controller's with sata fanouts for a total
of 8 drives per controller. Each mirror pair spreads across each
controller. 2 Intel 1GBe NICS configured bundled and configured for
LACP. 3 5-bay SuperMicro Hot Swap drive cages. I use it for NFS and
iSCSI testing. You can read more about it and some of the configuration
setup [here](http://everythingshouldbevirtual.com/?p=41). I highly
recommend checking it out. You can run it as a VM too for a VSA, but the
last I checked vmxnet-3 drivers are not supported.
