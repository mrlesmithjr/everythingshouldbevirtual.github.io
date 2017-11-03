---
  title: New Nexenta Server Test - iSCSI - SSD
  date: 2013-03-13 12:23:05
---

I just did a quick test using the VMware IO Analyzer using the same
tests that I did [here](https://everythingshouldbevirtual.com/nexenta-performance-testing-no-ssdssd "http\://everythingshouldbevirtual.com/nexenta-performance-testing-no-ssdssd").
This test was only using iSCSI with SSD for ZIL/SLOG and L2ARC. But the
results show a considerable jump in performance which means that
apparently the money has been well spent. I also changed the vmdk file
size for the IO Analyzer to 50GB this time seeing as the new server has
32GB of RAM. This way it is not caching the data in memory giving us
false results.

So here are the results of this test.

**OLTP 4K (4K 70% Read 100% Random) -- iSCSI**

Original Test Results

![OLTP_4K-Old](../../assets/OLTP_4K-Old-300x223.png)

New Test Results

![OLTP_4K-New](../../assets/OLTP_4K-New-300x216.png)

**SQL -- 64k (64k 66% Read 100% Random) -- iSCSI**

Old test results

![SQL_64K-Old](../../assets/SQL_64K-Old-300x228.png)

New test results

![SQL_64K-New](../../assets/SQL_64K-New-300x222.png)

**Exchange 2007 (8k 55% Read 80% Random) -- iSCSI**

Old test results

![Exchange_2007-Old](../../assets/Exchange_2007-Old-300x223.png)

New test results

![Exchange_2007-New](../../assets/Exchange_2007-New-300x222.png)

**Webserver (8k 95% Read 75% Random) -- iSCSI**

Old test results

![WebServer_8K-Old](../../assets/WebServer_8K-Old-300x223.png)

New test results

![WebServer_8K-New](../../assets/WebServer_8K-New-300x223.png)

As you can see the new server is definitely performing at a much higher
rate than the previous server was. I will be doing some additional
testing here soon to show the comparison even further to my prior build.
So stay tuned for that.

Enjoy!
