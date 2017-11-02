---
  title: vCenter 5.5U2 Upgrade on VCSA
  date: 2014-09-11
---

So I decided to upgrade my vCenter 5.5U1 VCSA to the latest vCenter
5.5U2 release yesterday and it has been a complete disaster at this
point. vCenter continually crashes every few minutes now. I was just
poking through logs and found where the service is crashing. I am
including the context of the log below.

```bash
mem> 2014-09-11T19:06:53.383Z [7F02DC3D0700 warning 'Default' opID=HB-host-8637@82541-3778684f] [VdbStatement] SQL execution failed: UPDATE VPX_HOST SET ROOT_RES_CONFIG = ? WHERE ID = ?
mem> 2014-09-11T19:06:53.383Z [7F02DC3D0700 warning 'Default' opID=HB-host-8637@82541-3778684f] [VdbStatement] Execution elapsed time: 2 ms
mem> 2014-09-11T19:06:53.383Z [7F02DC3D0700 warning 'Default' opID=HB-host-8637@82541-3778684f] [VdbStatement] Bind parameters:
mem> 2014-09-11T19:06:53.383Z [7F02DC3D0700 warning 'Default' opID=HB-host-8637@82541-3778684f] [VdbStatement] datatype: 11, size: 1234, arraySize: 0
mem> 2014-09-11T19:06:53.383Z [7F02DC3D0700 warning 'Default' opID=HB-host-8637@82541-3778684f] [VdbStatement] value = "ha-r..."
mem> 2014-09-11T19:06:53.383Z [7F02DC3D0700 warning 'Default' opID=HB-host-8637@82541-3778684f] [VdbStatement] datatype: 7, size: 4, arraySize: 0
mem> 2014-09-11T19:06:53.383Z [7F02DC3D0700 warning 'Default' opID=HB-host-8637@82541-3778684f] [VdbStatement] value = "8637"
mem> 2014-09-11T19:06:53.383Z [7F02DC3D0700 error 'Default' opID=HB-host-8637@82541-3778684f] [VdbStatement] SQLError was thrown: "ODBC error: (00000) - " is returned when executing SQL statement "UPDATE VPX_HOST SET ROOT_RES_CONFIG = ? WHERE ID = ?"
mem> 2014-09-11T19:06:53.417Z [7F02DC3D0700 verbose 'VpxProfiler' opID=HB-host-8637@82541-3778684f] [1-] [VpxdVdb] SaveCLOB: vim.ResourceConfigSpec (took 36 ms)
mem> 2014-09-11T19:06:53.418Z [7F02DC3D0700 error 'commonvpxCommon' opID=HB-host-8637@82541-3778684f] [Vpxd_HandleVmRootError] Received unrecoverable VmRootError. Generating minidump ...
mem> 2014-09-11T19:06:53.418Z [7F02DC3D0700 error 'Default' opID=HB-host-8637@82541-3778684f] An unrecoverable problem has occurred, stopping the VMware VirtualCenter service. Error: Error[VdbODBCError] (-1) "ODBC error: (00000) - " is returned when executing SQL statement "UPDATE VPX_HOST SET ROOT_RES_CONFIG = ? WHERE ID = ?"
mem> 2014-09-11T19:06:53.418Z [7F02DC3D0700 verbose 'commonvpxCommon' opID=HB-host-8637@82541-3778684f] Backtrace:
mem> -->
mem> 2014-09-11T19:06:53.487Z [7F02DC3D0700 panic 'Default' opID=HB-host-8637@82541-3778684f] (Log recursion level 2) Unrecoverable VmRootError. Panic!

------ In-memory logs end   --------
2014-09-11T19:06:53.487Z [7F02DC3D0700 panic 'Default' opID=HB-host-8637@82541-3778684f] Section for VMware VirtualCenter, pid=10869, version=5.5.0, build=2001466, option=Release
```

If anyone has any thoughts let me know.
Enjoy!
