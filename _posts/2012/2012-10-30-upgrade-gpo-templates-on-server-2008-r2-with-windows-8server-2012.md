---
  title: Upgrade GPO Templates on Server 2008 R2 with Windows 8/Server 2012
  date: 2012-10-30 22:18:48
---

Here is a quick howto on upgrading the GPO templates on a Server 2008 R2
domain controller with the newest ones from a Windows 8 machine. There
are a few additional ones that you can get from a Server 2012 machine as
well that are not included with Windows 8.

\*\*UPDATE\*\*

You can now download the new GPO Templates from
[here](http://www.microsoft.com/en-us/download/details.aspx?id=36991 "http\://www.microsoft.com/en-us/download/details.aspx?id=36991").

\*\*OLD WAY\*\* Use process above.

So here goes.

Open an elevated cmd prompt and run the following commands.

```bash
cd /d %windir%\winsxs
mkdir c:\temp
mkdir c:\temp\admx\PolicyDefinitions
mkdir c:\temp\admx\PolicyDefinitions\en-US
FOR /F %i IN (c:\temp\admx\admx.txt) DO copy %i c:\temp\admx\PolicyDefinitions\
FOR /F %i IN (c:\temp\admx\adml_en-us.txt) DO copy %i c:\temp\admx\PolicyDefinitions\en-US\
```

This will copy all of the new templates from your Windows 8 machine to
the temp location. Now logon to your Domain Controller and browse to
c:\\windows\\ and rename PolicyDefinitions to PolicyDefinition.orig. Now
you need to copy the PolicyDefinitions folder you just created in
c:\\temp\\admx to your domain controller in c:\\windows\\. Open up group
policy editor and enjoy your new upgraded templates.
