---
  title: Setup Storage Cluster with SDRS - vSphere5
  date: 2012-08-19 17:43:15
---

This is just a quick walk-through of setting up a new storage cluster
within vSphere5. When using a storage cluster it makes the management of
vm placement, space management and high disk IO vms easier to manage and
control automatically or manually (default). If this is your first time
using this feature I would recommend to first use the default settings
and let SDRS make recommendations that you may then take action on. This
is the same as standard DRS functionality on an HA cluster which
vmotions vms between hosts. SDRS will do storage vmotions to spread out
the load of your datastores based on IO latency (SIOC) (>15ms) and
space utilization (>80%). The downside is that as of current SRM5 does
not support Storage Clusters. Below are some screenshots which are step
by step of what it would look like setting up a new storage cluster.

Create new storage cluster and give it a name.

![](../../assets/New-Datastore-Cluster_2012-08-19_17-22-01-300x225.png "New Datastore Cluster_2012-08-19_17-22-01")

Select default which is manual mode. Once you get use to how it works
then you can enable fully automated.

![](../../assets/New-Datastore-Cluster_2012-08-19_17-23-40-300x227.png "New Datastore Cluster_2012-08-19_17-23-40")

Keep defaults. Enables SIOC (Storage IO Control), 15ms latency and 80%
storage utilization.

![](../../assets/New-Datastore-Cluster_2012-08-19_17-24-39-300x225.png "New Datastore Cluster_2012-08-19_17-24-39")

Select Host and clusters you want to use for the new storage cluster.

![](../../assets/vSphere-Client_2012-08-19_17-25-01-300x226.png "vSphere Client_2012-08-19_17-25-01")

Select newly created or existing datastores to add to the storage
cluster.

![](../../assets/New-Datastore-Cluster_2012-08-19_17-25-52-300x194.png "New Datastore Cluster_2012-08-19_17-25-52")

View summary and click finish if everything looks good.

![](../../assets/New-Datastore-Cluster_2012-08-19_17-26-42-300x194.png "New Datastore Cluster_2012-08-19_17-26-42")

Summary details of the newly created storage cluster.

![](../../assets/vSphere-Client_2012-08-19_17-18-55-300x121.png "vSphere Client_2012-08-19_17-18-55")

Make sure to enable Round Robin mutlipathing for the load balancing
policy on the new datastores if they are iSCSI, FC or FCOE connected.

![](../../assets/09-50-16-300x132.png "09-50-16")

![](../../assets/09-54-17-300x117.png "09-54-17")

![](../../assets/10-01-02-300x218.png "10-01-02")
