---
  title: Latest Nexenta Testing Results
  date: 2013-04-27 08:27:31
---

So I have been running my new Nexentastor CE storage for a little more
than a month now and wanted to run another IO test real quick as a
sanity check. So I setup the [VMware IO Analyzer](http://labs.vmware.com/flings/io-analyzer "http\://labs.vmware.com/flings/io-analyzer") again, but changed the virtual disk
size to 150GB and ran the test for 600 secs this time. I ran the same iSCSI
tests as I ran [here](https://everythingshouldbevirtual.com/nexenta-performance-testing-no-ssdssd "http\://everythingshouldbevirtual.com/nexenta-performance-testing-no-ssdssd") to
get an idea of the results I was getting from my older setup. Compare to iSCSI - SSD.

Here are the results.

**OLTP 4K (4K 70% Read 100% Random) -- iSCSI**

![08-18-24](../../assets/08-18-24-300x19.png)

![08-18-39](../../assets/08-18-39-300x224.png)

**SQL -- 64k (64k 66% Read 100% Random) -- iSCSI**

![08-20-12](../../assets/08-20-12-300x17.png)

![08-20-24](../../assets/08-20-24-300x228.png)

**Exchange 2007 (8k 55% Read 80% Random) -- iSCSI**

![08-22-17](../../assets/08-22-17-300x17.png)

![08-22-27](../../assets/08-22-27-300x226.png)

**Webserver (8k 95% Read 75% Random) -- iSCSI**

![08-23-56](../../assets/08-23-56-300x17.png)

![08-24-05](../../assets/08-24-05-300x225.png)

\*\*Update 05/15/2013\*\*

Updating with NFS results. The same IOMeter tests were done as setup at
the beginning of this post. This time I tested the NFS performance
instead of iSCSI. Again as the test I performed a while back iSCSI
outperformed NFS again.

So here are the results.

**OLTP 4K (4K 70% Read 100% Random) -- NFS**

![12-53-49](../../assets/12-53-49-300x16.png)

![12-54-00](../../assets/12-54-00-300x224.png)

**SQL -- 64k (64k 66% Read 100% Random) -- NFS**

![12-55-43](../../assets/12-55-43-300x17.png)

![12-55-57](../../assets/12-55-57-300x225.png)

**Exchange 2007 (8k 55% Read 80% Random) -- NFS**

![12-58-07](../../assets/12-58-07-300x16.png)

![12-58-20](../../assets/12-58-20-300x226.png)

**Webserver (8k 95% Read 75% Random) -- NFS**

![12-59-37](../../assets/12-59-37-300x17.png)

![12-59-48](../../assets/12-59-48-300x224.png)

So it looks like I have confirmed that I am definitely getting much
better results on my new build. :) Money well spent!

Enjoy!
