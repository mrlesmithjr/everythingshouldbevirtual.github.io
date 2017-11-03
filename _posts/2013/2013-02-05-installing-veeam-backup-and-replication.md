---
  title: Installing Veeam Backup and Replication
  date: 2013-02-05 16:48:11
---

In this post we will be installing the Backup and Replication server
components. This is a fairly straight forward installation with very
minimal setup steps after installation to get started using it. Of
course there are many advanced setup scenarios that you can build which
would require much more setup time. I will go into those on another post
soon.

First off read the minimum requirements and FAQ
[here](http://www.veeam.com/veeam_backup_6_5_release_notes_rn.pdf).

Or just read below for the hardware requirements which were copied from
the above pdf.

**Veeam Backup & Replication Server**
Hardware
CPU: any x86/x64 processor.
Memory: 4 GB RAM.
Hard Disk Space: 2 GB for product installation. 10 GB per 100 VM for
guest file system catalog folder (persistent data). Sufficient free disk
space for Instant VM Recovery cache folder (non-persistent data, at
least 10 GB recommended).
Network: 1 Gbps LAN for on-site backup and replication, 1 Mbps or faster
WAN for off-site backup and replication. High latency links are
supported, but TCP/IP connection must not drop.

**Backup Proxy Server
**Hardware
CPU: modern x86/x64 processor (minimum 2 cores). Using faster multi-core
processors improves data processing performance, and allows for more
concurrent jobs.
Memory: 2 GB RAM per concurrent job. Using faster memory (DDR3) improves
data processing performance.
Hard Disk Space: 300 MB.
Network: 1 Gbps LAN for on-site backup and replication, 1 Mbps or faster
WAN for off-site backup and replication. High latency links are
supported, but TCP/IP connection must not drop.

**Backup Repository Server
**Hardware
CPU: any x86/x64 processor.
Memory: 2 GB RAM per concurrent job.
Network: 1 Gbps LAN for on-site backup and replication, 1 Mbps or faster
WAN for off-site backup and replication. Unreliable, high latency links
with packet loss are supported, but TCP/IP connection must not drop
completely.

In this scenario we will be using a Windows 2008 R2  x64 server to
install on.

Let's get started.

Launch installation
program![](../../assets/ATAYUdfwOcnRAAAAAElFTkSuQmCC)

![](../../assets/4I3WMEKH8AH8AGdD1iB4YABPoAP4AP4QNs+8P9yeL8dfC8IHgAAAABJRU5ErkJggg==)

![](../../assets/wCw6GpB4NtZvgAAAABJRU5ErkJggg==)

![](../../assets/7rt9YThXnO+AAAAAElFTkSuQmCC)![](../../assets/0Jo3uShUGBAAAAAElFTkSuQmCC)

Download and browse to your trial license file.

![](../../assets/wPDFcUfQTp9+gAAAABJRU5ErkJggg==)

![](../../assets/o14LsAAAAAElFTkSuQmCC)

![](../../assets/x8Ib1ne2vQhVgAAAABJRU5ErkJggg==)

![](../../assets/+bCffChSB4SAAAAAElFTkSuQmCC)

![](../../assets/wUA5gJzBEAAAAAElFTkSuQmCC)![](../../assets/Hx+VP2Cx2ZaoAAAAAElFTkSuQmCC)

![](../../assets/D7ie8uRgu0gAEYgAEYGAoDo6E0hHYwKGEABmAABmAgzADGvzQsEiChEQzAAAzAwFAY+P89ezKHApjscwAAAABJRU5ErkJggg==)

![](../../assets/0HAzDQBgNIHsmTwcMADMAADBTKAJIvtGPbyADZB5UEDMAADPSbASSP5MngYQAGYAAGCmXgJMnzCMjRPAKSuBJXGIABGICBLhiQp0DqJ979fwZubtZE3jr2AAAAAElFTkSuQmCC)

![](../../assets/6UgYoA5QBysAYZaCFPAENKAOUAcoAZYAysLwy8P8B5HesKcbM99sAAAAASUVORK5CYII=)

Now that the installation is complete launch the app.

![](../../assets/w+37YmNoNM3CQAAAABJRU5ErkJggg==)

Now we will configure our first backup job to run.

![](../../assets/wfsRDhlx1TlVAAAAABJRU5ErkJggg==)

Click backup job.

![](../../assets/dHy975eqz0j52spx+taOV81VJbfKIACKIACKNCiArIw8oMCKIACKIACLSvAWtWydWgbCqAACqCAOhlRYlUB71gv5XhdK+fr2L33tJ+TZhiAARiAARiAARiAARiAARiAARiAARiAARiAgT4YUBetjZC1UbL+fph2YD6fL+RfHw2iDsCGARiAARiAARiAARiAARiAARiAARiAARiAARiYIgPqZxWn7KvXJ4tn8gfOVwbDFAcDfYZ7GIABGIABGIABGIABGIABGIABGIABGICBPhnA+fqQK7BPQakLPWEABmAABmAABmAABmAABmAABmAABmAABmAABoQBnK84X3E+wwAMwAAMwAAMwAAMwAAMwAAMwAAMwAAMwAAMDMBA6Hzl281QAAVQAAVQAAVQAAVQAAVQAAVQAAVQAAVQAAVQAAX6U0Byvv4D7dw9oi1+w4wAAAAASUVORK5CYII=)

![](../../assets/B8m6umX8rJIbwAAAABJRU5ErkJggg==)

Click on VMware vSphere

![](../../assets/HwlzDg+QxzDgAAAAAElFTkSuQmCC)

Enter IP or DNS name for host or enter your vCenter info.

![](../../assets/pgEn2Dbdbi8AAAAASUVORK5CYII=)

![](../../assets/rxIUAAAAAElFTkSuQmCC)

![](../../assets/m2P7thBsCMAAAAABJRU5ErkJggg==)

![](../../assets/wOVYFPqluMbigAAAABJRU5ErkJggg==)

![](../../assets/X+K5GT4P91GeQAAAABJRU5ErkJggg==)

Click add to select which virtual machines you would like to backup.

![](../../assets/9bWwS4Q4x3AAAAAElFTkSuQmCC)

When adding objects to backup you can select data center, cluster,
hosts, folders, resource pools, datastores or individual virtual
machines. Make sure that you layout your different objects in vCenter to
whatever makes sense and makes this selection process the easiest and
cleanest as well as something that will capture the virtual machines
even if they move between any of these objects.

![](../../assets/7rVp3DZ8y2LAAAAAElFTkSuQmCC)

![](../../assets/Jq13Q2ubTdYAAAAASUVORK5CYII=)

Select your backup repository you want to backup to. For information on
adding a Linux NFS backup repository read
[this](https://everythingshouldbevirtual.com/veeam-backup-and-replication-to-nexenta-nfs "https\://everythingshouldbevirtual.com/veeam-backup-and-replication-to-nexenta-nfs")article.

![](../../assets/hQhJGz9IAAAAASUVORK5CYII=)

Select Enable application-aware image processing for servers such as
Exchange or MS SQL and any other application that uses VSS technology.

![](../../assets/A4yX0R0pnztiAAAAAElFTkSuQmCC)

![](../../assets/HykGHmF6g1v2AAAAAElFTkSuQmCC)

![](../../assets/DlHQMYOBIDBPWavBJUgqqDemEGjrRIGes1Fyl5l3eCek0GCOqF5UQH9ZoXvQVf3jGAgSMxQFCvyStBJag6qBdm4EiLlLFec5GSd3knqNdkgKBeWE50UK950Vvw5R0DGDgSAwT1mrwSVIKqg3phBo60SBnrNRcpeZd3gnpNBggqOSGoGMAABjCAAQxgYCgGCCoghwJSt+SavynLu7xjAAMYwEDIAEElqAQVAxjAAAYwgAEMDMUAQQXkUED6Ddpv0BjAAAYwgAEMzIL6fwHiSOSQnYkArgAAAABJRU5ErkJggg==)

That's all there is to it. Now you are ready to start your backups.
More posts will follow going over file level restores, full vm restores
and etc. As well as on replication scenarios.

Feel free to leave comments as I look forward to any and all feedback.
