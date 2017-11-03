---
  title: "VMware KB: Disabling VAAI Thin Provisioning Block Space Reclamation UNMAP in ESXi 5.0"
  date: 2012-07-28 23:32:25
---

Here is the VMware KB article on how to disable the VAAI SCSI UNMAP
feature in ESXi5. The latest update patch disables this. You can check
to see if it is enabled or disabled by running the following on your
host command line by using the following command. The returned Int Value
should be 0.

```bash
esxcli system settings advanced list --option /VMFS3/EnableBlockDelete

Path: /VMFS3/EnableBlockDelete

Type: integer

**Int Value: 0**

Default Int Value: 0

Min Value: 0

Max Value: 1

String Value:

Default String Value:

Valid Characters:

Description: Enable VMFS block delete
```

[VMware KB: Disabling VAAI Thin Provisioning Block Space Reclamation UNMAP in ESXi 5.0](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2007427)
