---
title: "Nested ESXi Templates"
date: "2019-03-18 21:34:00"
categories:
  - Virtualization
tags:
  - VMware
---

Over the past week I have been working on a streamlined process to deploy
nested ESXi 6.7 via automation tooling. While going through this I came across
an issue with all nested ESXi hosts obtaining the same IP address. I had already
followed the normal processes that are well documented in regards to prepping
the image before bundling up for use as a template.

The following were the commands that I found as a typical scenario:

```bash
esxcli system settings advanced set -o /Net/FollowHardwareMac -i 1
sed -i '/\\/system\\/uuid/d' /etc/vmware/esx.conf
/sbin/auto-backup.sh
```

These seemed to be working as desired until I decided to spin up quite a few of
them, and boom, there it was, all nested ESXi hosts had the same IP address.
Now for clarity, I am using Packer to build the base nested ESXi image (more on
this in a later post). But that was not the cause of the issue.

The following is what I was finally able to get to work as desired (in addition
to the above commands):

```bash
sed -i '/\\/net\\/pnic\\/child\\[0000\\]\\/mac/d' /etc/vmware/esx.conf
sed -i '/\\/net\\/vmkernelnic\\/child\\[0000\\]\\/mac/d' /etc/vmware/esx.conf
```

So the whole command set in one would look like:

```bash
esxcli system settings advanced set -o /Net/FollowHardwareMac -i 1
sed -i '/\\/system\\/uuid/d' /etc/vmware/esx.conf
sed -i '/\\/net\\/pnic\\/child\\[0000\\]\\/mac/d' /etc/vmware/esx.conf
sed -i '/\\/net\\/vmkernelnic\\/child\\[0000\\]\\/mac/d' /etc/vmware/esx.conf
/sbin/auto-backup.sh
```

The issue was that all of the hosts were still coming up with the same MAC
address. So, not sure what was going on here in my scenario, but possibly this
may save someone else the headache I went through on this.

Enjoy!
