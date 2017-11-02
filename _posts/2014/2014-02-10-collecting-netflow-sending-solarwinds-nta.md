---
  title: Collecting Netflow and Sending to Solarwinds NTA
  date: 2014-02-10 07:00:45
---

If you are interested in collecting, viewing and inspecting
[Netflow](http://en.wikipedia.org/wiki/NetFlow "http\://en.wikipedia.org/wiki/NetFlow")
data like I am, then you will be interested in this. Netflow gives you
deep level inspection into your network traffic such as source and
destination of traffic, protocols and types of service, plus much more.
Why is this valuable information? Maybe you are interested in the
different types of traffic that are floating around on your network or
possibly you are getting reports of slow network connectivity by users.
So how do we get this Netflow data? First off your network devices must
support exporting Netflow data to begin with, sort of anyways! This goes
for any network switches, firewalls and even vSphere. The second part of
gathering Netflow data is that you must have a Netflow collector
installed on your network, [NTOP](http://ntop.org "http\://ntop.org") is
a good open-source Netflow collector but it loses all of it's data when
rebooted; however, lately I have been doing some deep level testing with
several of Solarwinds products
([SAM](http://www.solarwinds.com/server-application-monitor-b.aspx "http\://www.solarwinds.com/server-application-monitor-b.aspx"),
[NPM](http://www.solarwinds.com/network-performance-monitor.aspx "http\://www.solarwinds.com/network-performance-monitor.aspx"),
[Virtualization
Manager](http://www.solarwinds.com/virtualization-manager.aspx "http\://www.solarwinds.com/virtualization-manager.aspx"),
[NTA](http://www.solarwinds.com/netflow-traffic-analyzer.aspx "http\://www.solarwinds.com/netflow-traffic-analyzer.aspx")
and [Storage Manager](http://www.solarwinds.com/storage-manager.aspx "http\://www.solarwinds.com/storage-manager.aspx"));
in particular, Solarwinds NTA (Netflow Traffic Analyzer). This product
provides an unbelievable level of detail and is visually pleasing as
well. One other thing about this product is the integration between
their other products which provides a seamless view into your
environment. Instead of jumping between different products that you may
use and not having any sense of correlation between different elements
in your environment.

So assuming that you are running a physical and virtual environment like
I am, setting all of this up can be somewhat of a challenge. In my lab I
am running a Cisco 3750G 48TS switch stack, physical PFSense firewall
and three vSphere 5.5 hosts running between 30-40 VMs at any given time.
So with my setup the first challenge is that my Cisco switches do not
support exporting Netflow, obviously your environment may be different.
So in order for me to collect netflow data from my switches you can take
a look
[here](http://everythingshouldbevirtual.com/vmware-vds-rspan-port-mirroring "http\://everythingshouldbevirtual.com/vmware-vds-rspan-port-mirroring")
on how to create a RSPAN port mirror and send that data to a vDS
(vSphere Distributed Switch) port. The difference in this article
compared to the link above will be that we will not be installing or
using NTOP but instead on our VM we will be using
[nProbe](http://www.ntop.org/products/nprobe/ "http\://www.ntop.org/products/nprobe/")
(created by the same people who create NTOP). nProbe will be acting as a
Netflow proxy in our setup for each device that we will be collecting
Netflow data from and then forwarding onto our Solarwinds NTA. nProbe is
not a free product but so far it is well worth it. There are several
linux open-source
([fprobe](http://sourceforge.net/projects/fprobe/ "http\://sourceforge.net/projects/fprobe/"),
[ipt-netflow](http://sourceforge.net/projects/ipt-netflow/ "http\://sourceforge.net/projects/ipt-netflow/")
and
[pflow](http://www.openbsd.org/cgi-bin/man.cgi?query=pflow&sektion=4&manpath=OpenBSD+Current "http\://www.openbsd.org/cgi-bin/man.cgi?query=pflow&sektion=4&manpath=OpenBSD+Current"))
Netflow forwarders but I have not had good success yet using with
Solarwinds but I will be doing some additional testing on those as well
in the near future.

Installing nProbe is fairly straight forward but configuring it is a
little bit tricky. I chose to install nbox as well to have a good web
interface to configure nProbe but you can also run it from command line
if you would like to as well. Seeing as I set mine up on Ubunutu 12.04
x64 I followed the process
[here](http://www.nmon.net/apt/ "http\://www.nmon.net/apt/") to add the
apt repository to simplify the installation. However I did not install
everything as the link above specified. Below are the commands that I
ran.

```bash
sudo bash
/bin/echo -e "deb http://www.nmon.net/apt x64/ndeb http://www.nmon.net/apt all/" > /etc/apt/sources.list.d/ntop.list
apt-get update
apt-get install pfring nbox nprobe
```

Now you should have a working web ui for nbox and you can connect by
using your browser of choice and connect to <https://ip.or.dns.name>.

![15-06-26](../../assets/15-06-26-300x213.png)

Click Log In (The default username/password is nbox/nbox.)

![15-08-37](../../assets/15-08-37-300x245.png)

You will now be presented with the following UI.

![15-10-22](../../assets/15-10-22-300x238.png)

We will now enable [PF_Ring](http://www.ntop.org/products/pf_ring/ "http\://www.ntop.org/products/pf_ring/").

![15-11-56](../../assets/15-11-56-300x202.png)

Set to enabled (We will reboot after we configure nProbe).

![15-13-17](../../assets/15-13-17-300x147.png)

Now on to configuring nProbe.

![15-16-52](../../assets/15-16-52-300x153.png)

Do not turn the Proxy on at this point.

![15-18-07](../../assets/15-18-07-300x233.png)

We will be configuring as a Proxy so click on Proxy.

![15-19-26](../../assets/15-19-26-300x230.png)

Click Enable to enable automatic startup.
Enter 2055 for the Listening Port. (This is what will be capturing from
our other devices on the network)
Enter your Solarwinds NTA address and port (default is 2055) for the
collector(s) IP: x.x.x.x:2055

Now under Flow Export Format choose v5.

![15-27-20](../../assets/15-27-20-300x46.png)

Now click on Flow Export Policy and scroll down and change the Input
SNMP Interface ID from "Auto" to "lo" and also change the Output
SNMP Interface ID from "Auto" to "eth0". (These are very important!)

![19-23-16](../../assets/19-23-16-300x161.png)

![19-23-33](../../assets/19-23-33-300x168.png)

Now scroll to the bottom and click "Save Changes".

![19-26-06](../../assets/19-26-06-300x110.png)

We are now done configuring nProbe and need to enable it and then
reboot. So click on Status and click "On".

![19-29-22](../../assets/19-29-22-300x237.png)

Now select the Admin dropdown and click "Reboot".

![19-30-52](../../assets/19-30-52-300x206.png)

Now you can head over to your Solarwinds NTA and setup your netflow
node.

![11-18-44](../../assets/11-18-44-300x130.png)

![11-19-15](../../assets/11-19-15-300x185.png)

![11-19-28](../../assets/11-19-28-300x174.png)

And if you followed the link above about setting up RSPAN Port Mirroring
you will likely start seeing some data flowing in.

![11-19-51](../../assets/11-19-51-192x300.png)

Now we will head over to our vCenter WebUI and configure netflow on our
vDS port groups.

Go to the networking view, select your vDS dvSwitch and right-click and
click Manage Distributed Port Groups..

![11-39-41](../../assets/11-39-41-259x300.png)

Select Monitoring and then click Next..

![11-40-24](../../assets/11-40-24-300x175.png)

Now select the port groups that you want to collect netflow on and then
click Next..

![11-41-13](../../assets/11-41-13-300x176.png)

Select the drop-down and choose enabled..

![11-41-24](../../assets/11-41-24-300x176.png)

Click Next..

![11-41-33](../../assets/11-41-33-300x176.png)

Click Finish..

![11-41-44](../../assets/11-41-44-300x176.png)

Now click on the manage tab within your vDS dvSwitch and select Netflow..

![11-42-49](../../assets/11-42-49-300x80.png)

Click edit..

![11-42-59](../../assets/11-42-59-300x75.png)

Enter the IP of your nBox server that we just setup above and leave the
default port 2055. Enter an unused IP on your network preferably on your
management subnet for the Switch IP address (This is the source your
nBox server will see).

![11-43-45](../../assets/11-43-45-300x293.png)

Complete..

![11-43-58](../../assets/11-43-58-300x106.png)

Now your configuration should be complete for gathering netflow from
your Cisco switches and your vSphere vDS setup. Now nProbe will be
forwarding all netflow data over to your Solarwinds NTA and after a bit
of time you should now start seeing some nice graphs and data of traffic
on your network.

![12-02-35](../../assets/12-02-35-300x218.png)

![12-02-56](../../assets/12-02-56-300x171.png)

Now you can also add other devices on your network that export Netflow,
just point them to your nProbe server and let nProbe do it's job!

That's it...Happy Netflow'n...

Enjoy!
