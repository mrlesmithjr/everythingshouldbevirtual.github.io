---
  title: Slow Transfer Through Cisco ASA
  date: 2013-03-25 14:33:29
---

I recently ran into this issue in which transferring files through a
Cisco ASA Firewall was extremely slow only in one direction. The
transfer would run between 10-15 MB/s one way and between 100-500 KB/s
the other way. The servers were both Windows 2008 R2 in which this issue
would occur. I could transfer files from the server inside the DMZ to a
Windows 2003 server without experiencing the same issue. I was able to
duplicate this scenario for any other given Windows 2008 R2 to Windows
2008 R2 server doing the same testing. So obviously the issue was
Windows 2008 R2 specific in this case. So after some investigation and
testing I was able to come up with the following TCP setting changes
that corrected the issue and the transfers were the same either way
after the reboot which was 10-15 MB/s either way. In my scenario I only
needed to make the change on the server inside the DMZ. I also did this
same test using a PFSense firewall with the exact same firewall rules in
place and this issue was not present.

These are the issues you may experience.

-   Poor performance
-   Packet loss
-   Network latency
-   Slow data transfer

Changes made to server inside DMZ.

```powershell
netsh int tcp set global RSS=Disable
netsh int tcp set global chimney=Disabled
netsh int tcp set global autotuninglevel=Disabled
netsh int tcp set global congestionprovider=None
netsh int tcp set global ecncapability=Disabled
netsh int ip set global taskoffload=disabled
netsh int tcp set global timestamps=Disabled
```

To revert changes back to default.

```powershell
netsh int tcp set global RSS=Enabled
netsh int tcp set global chimney=Enabled
netsh int tcp set global autotuninglevel=normal
netsh int tcp set global congestionprovider=ctcp
netsh int tcp set global ecncapability=Enabled
netsh int ip set global taskoffload=Enabled
netsh int tcp set global timestamps=Enabled
```
