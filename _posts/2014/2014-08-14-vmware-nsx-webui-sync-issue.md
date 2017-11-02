---
  title: VMware NSX WebUI Sync Issue
  date: 2014-08-14
---

So I recently had an issue when trying to create some new tenant
networks and had a very strange issue. When creating the Logical
Switches within NSX I was receiving a timeout error stating that the
they had not been created. However when looking at the thick client I
could clearly see the new Logical Switches in the inventory. What is up
with that? Anyways I opened a support ticket and we started going
through everything trying to figure out what the issue was. Not really
coming up with much as we went through anything. So the engineer was
collecting tech support logs and screenshots I happened to mention to
him that I had upgraded vCenter Server to 5.5 U1c last week and that I
was also using vCenter Heartbeat. He thought about it a bit and we dove
into NSX manager and stumbled upon the view which shows the last
successful inventory sync between NSX Manager and vCenter. Sure enough
there it was....It has not successfully synced up in almost a
week...Right before I upgraded vCenter Server to the latest version. So
we then rebooted the NSX Manager to see if it would sync up and sure
enough it did and all was working again. So if you find yourself in this
scenario make sure to check within NSX Manager to validate that
synchronizations are occurring successfully.

Enjoy!
