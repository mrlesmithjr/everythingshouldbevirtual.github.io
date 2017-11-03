---
  title: Cacti Monitoring for Windows Servers
  date: 2012-09-28 20:13:25
---

A little over four years ago I spent some time learning cacti and snmp
to come up with a good solution to monitor Windows Servers. I had used
Cacti several times previous to then, but never found any graphs that I
really liked or any that would give me the level of detail that I was
looking for. So I created my own and shared back to the Cacti community.
To this day I still use these daily at home and at work, and they
continue to work well. Today I am going to go through a quick
installation of these templates into a new cacti install and then add a
windows server 2008 to monitor.

I will be using a new Ubuntu Server 12.04 install for the cacti server.
Once the server is built install cacti using the following and follow
the screenshots.

```bash
sudo apt-get install cacti
```

![](../../assets/2012-09-28_17-00-05-300x166.png "2012-09-28_17-00-05")

![](../../assets/2012-09-28_17-02-32-300x166.png "2012-09-28_17-02-32")

![](../../assets/2012-09-28_17-03-16-300x167.png "2012-09-28_17-03-16")

![](../../assets/2012-09-28_17-04-02-300x167.png "2012-09-28_17-04-02")

![](../../assets/2012-09-28_17-05-47-300x166.png "2012-09-28_17-05-47")

![](../../assets/2012-09-28_17-08-15-300x168.png "2012-09-28_17-08-15")

![](../../assets/2012-09-28_17-08-54-300x167.png "2012-09-28_17-08-54")

![](../../assets/2012-09-28_17-12-12-300x167.png "2012-09-28_17-12-12")

![](../../assets/2012-09-28_17-12-44-300x166.png "2012-09-28_17-12-44")

Now connect to the web ui of cacti and configure

> NOTE: Change IP to reflect your cacti server

<http://192.168.211.128/cacti>

![](../../assets/2012-09-28_17-14-44-300x167.png "2012-09-28_17-14-44")

![](../../assets/2012-09-28_17-16-23-300x151.png "2012-09-28_17-16-23")

![](../../assets/2012-09-28_17-16-54-200x300.png "2012-09-28_17-16-54")

All done. Now you will be redirected back to the login screen.

![](../../assets/2012-09-28_17-19-17-300x185.png "2012-09-28_17-19-17")

Login using admin/admin and then change password on next screen

![](../../assets/2012-09-28_17-20-18-300x208.png "2012-09-28_17-20-18")

![](../../assets/2012-09-28_17-21-08-300x127.png "2012-09-28_17-21-08")

Cacti is now installed.

Download the latest version of cacti from
[here](https://everythingshouldbevirtual.com/cacti-templates-for-windows/ "http\://everythingshouldbevirtual.com/cacti-templates-for-windows/").
Version 13 is the latest at the time of writing this. Download it and
extract the zip file. There is a readme.txt that explains where to place
the files.

\*\*\*UPDATE\*\*\*

All new templates will be updated and current from GitHub.

<https://github.com/mrlesmithjr/cacti.git>

```bash
git clone https://github.com/mrlesmithjr/cacti/
```

Copy files from the resource\\snmp_queries\\ folder extracted to the
cacti server using WinSCP to the /tmp folder.

![](../../assets/2012-09-20_11-12-17-300x141.png "2012-09-20_11-12-17")

![](../../assets/2012-09-20_11-15-57-300x174.png "2012-09-20_11-15-57")

![](../../assets/2012-09-20_11-16-13-300x171.png "2012-09-20_11-16-13")

![](../../assets/2012-09-20_11-16-21-300x177.png "2012-09-20_11-16-21")

Now on the cacti server cd /tmp

![](../../assets/2012-09-20_11-16-41-300x71.png "2012-09-20_11-16-41")

```bash
sudo cp snmp_informant_standard_*.xml /usr/share/cacti/site/resource/snmp_queries/
```

![](../../assets/2012-09-20_11-18-02-300x100.png "2012-09-20_11-18-02")

![](../../assets/2012-09-20_11-18-10-300x86.png "2012-09-20_11-18-10")

All done.

Now we have to import the xml template files from the \\template folder
into the cacti web ui.

![](../../assets/2012-09-28_19-23-35-300x102.png "2012-09-28_19-23-35")

In the cacti web ui go to Import/Export/Import Templates

![](../../assets/2012-09-28_19-25-12-300x119.png "2012-09-28_19-25-12")

Now browse to the folder where you extracted the zip file to and select
the first
cacti_host_template_windows_host\_-\_snmp_informant.xml and click
import.

![](../../assets/2012-09-28_19-45-58-300x195.png "2012-09-28_19-45-58")

The templates are ready to be used now.

Now install SNMP informant Standard on your Windows machine you want to
monitor. Get SNMP Informant from [here](http://www.snmp-informant.com/downloads.htm#SNMP_Informant_-_Freeware_Products "http\://www.snmp-informant.com/downloads.htm#SNMP_Informant\_-\_Freeware_Products").
Download the 1.6 version. You need to have the Windows SNMP agent enabled and
configured on your windows machine.

![](../../assets/2012-09-28_16-00-09-300x97.png "2012-09-28_16-00-09")

![](../../assets/2012-09-28_19-53-52-300x236.png "2012-09-28_19-53-52")

![](../../assets/2012-09-28_19-54-41-300x222.png "2012-09-28_19-54-41")

![](../../assets/2012-09-28_19-55-04-300x229.png "2012-09-28_19-55-04")

![](../../assets/2012-09-28_19-55-27-300x225.png "2012-09-28_19-55-27")

![](../../assets/2012-09-28_19-55-46-300x229.png "2012-09-28_19-55-46")

![](../../assets/2012-09-28_19-56-07-300x227.png "2012-09-28_19-56-07")

Now configure the SNMP Service

![](../../assets/2012-09-28_19-57-25-300x220.png "2012-09-28_19-57-25")

We are now ready to add our first Windows Server.

Under devices/add

![](../../assets/2012-09-28_19-59-14-300x142.png "2012-09-28_19-59-14")

![](../../assets/2012-09-28_20-01-48-300x140.png "2012-09-28_20-01-48")

Now select create graphs for this host and select the statistics you
want to gather

![](../../assets/2012-09-28_20-03-02-300x160.png "2012-09-28_20-03-02")

Click create and do not change anything on the next screen. Select
create again.

![](../../assets/2012-09-28_20-04-46-300x71.png "2012-09-28_20-04-46")

There are additional drop downs for Disk, memory and network stats. Make
sure to go through select those as well.

Now you have to add the new device to the graph tree.

![](../../assets/2012-09-28_20-07-30-300x98.png "2012-09-28_20-07-30")

![](../../assets/2012-09-28_20-08-48-300x63.png "2012-09-28_20-08-48")

That's it. Now just wait for a few minutes and you should start getting
some nice looking graphs for your device.

Enjoy...

You can follow the thread over on the Cacti forum [here](http://forums.cacti.net/viewtopic.php?f=12&t=29832 "http\://forums.cacti.net/viewtopic.php?f=12&t=29832").

You can download previous versions of the templates from [here](https://everythingshouldbevirtual.com/cacti-templates-for-windows "http\://everythingshouldbevirtual.com/cacti-templates-for-windows")if
you are getting XML parse errors importing.
