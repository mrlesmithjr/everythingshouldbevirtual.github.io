---
  title: vSphere 5.1 - Migrating standard switches (VSS) to distributed switches (VDS) using new web ui
  date: 2012-09-13 21:15:47
---

Here is a quick walk-through on the process of migrating standard
vSwitches (VSS) to distributed switches (VDS) using the new web ui in
vSphere 5.1. In this post I will be migrating the management ports,
vMotion ports and guest network ports only. I chose to do
them separately doing guest networks first and then moving the managment
and vMotion ports second. You can do this all at one time if you want to
and it should work fine. However I did mess up once and the new rollback
feature kicked in for my console ports and I was back in business. Click
through the gallery below and go through each picture one by one and
there are captions in those areas of detail. Enjoy!
