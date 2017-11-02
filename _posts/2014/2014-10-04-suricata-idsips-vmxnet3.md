---
  title: Suricata IDS/IPS VMXNET3
  date: 2014-10-04
---

As part of a bigger post coming soon I have been using [Suricata IDS](http://suricata-ids.org/ "http\://suricata-ids.org/")
and my Logstash server has been getting hammered and unable to keep up (running
a single node setup) but finally figured out why this was happening so I
am sharing this with others in case you decide to send Suricata IDS logs
to Logstash or any other Syslog collector you will more than likely run
into this as well.

What I was noticing in /var/log/suricata/fast.log was the following
entries being spit out (note the timestamps to see how fast these were
being generated)

```bash
10/03/2014-20:45:32.046859  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.046909  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.046938  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.046966  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.046995  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.055373  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.055449  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.094880  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.094951  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.171191  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.171253  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.211188  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.211348  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.215418  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.215471  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
10/03/2014-20:45:32.216030  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49202 -> y.y.y.y:443
```

So basically this has been doing this non-stop for about 2-3 days now
and being that I am sending the Suricata logs through a Redis cache
broker it was overload the queues and chewing up all of the available
memory and crashing Redis and also in turn crashing Elasticsearch.
(Sounds like fun right?)
Anyways here is what the issue with this is. These invalid checksum
alerts are being generated because by default Suricata has
checksum-validation set to yes in /etc/suricata/suricata.yaml and by
default checksum offloading is turned on for VMXNET3 (Other adapters as
well). So now the decision is to set checksum-validation to no in
Suricata or turn off checksum offloading on the adapter. At least in my
case I still want Suricata to detect **TRUE** invalid checksums
therefore I chose to disable checksum offloading on the adapter. So
being that Suricata is running on a Linux VM you need to use the ethtool
to turn this off.
If on a Ubuntu system you can install this using apt.

```bash
sudo apt-get install ethtool
```

Now to validate if checksum offloading is turned on for the adapter that
you have Suricata setup to listen on do the following. (Change eth0 to
whichever adapter is in your setup)

```bash
sudo ethtool -k eth0
```

And you should see output similar to below.

```bash
Offload parameters for eth0:
rx-checksumming: on
tx-checksumming: on
scatter-gather: on
tcp-segmentation-offload: on
udp-fragmentation-offload: off
generic-segmentation-offload: on
generic-receive-offload: on
large-receive-offload: off
rx-vlan-offload: on
tx-vlan-offload: on
ntuple-filters: off
receive-hashing: off
```

All of the settings in bold above represent what settings are on when
checksum offloading is turned on. So all you need to do to turn off
checksum offloading is run the following.

```bash
sudo ethtool --offload eth0 rx off tx off
```

Now if you validate your settings you should the following

```bash
Offload parameters for eth0:
rx-checksumming: off
tx-checksumming: off
scatter-gather: off
tcp-segmentation-offload: off
udp-fragmentation-offload: off
generic-segmentation-offload: off
generic-receive-offload: on
large-receive-offload: off
rx-vlan-offload: on
tx-vlan-offload: on
ntuple-filters: off
receive-hashing: off
```

And now if you take a look at your /var/log/suricata/fast.log it should
be much cleaner. And should only be seeing invalid checksums for valid
packets instead of the majority.

```bash
10/03/2014-21:57:32.101095  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:53742 -> y.y.y.y:80
10/03/2014-21:57:51.013553  [**] [1:2200074:1] SURICATA TCPv4 invalid checksum [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:49593 -> y.y.y.y:80
10/03/2014-21:57:54.392025  [**] [1:2210044:1] SURICATA STREAM Packet with invalid timestamp [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:65264 -> y.y.y.y:80
10/03/2014-21:57:57.231561  [**] [1:2210044:1] SURICATA STREAM Packet with invalid timestamp [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:29270 -> x.x.x.x:80
10/03/2014-21:57:57.233436  [**] [1:2210033:1] SURICATA STREAM FIN1 invalid ack [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:80 -> x.x.x.x:29270
10/03/2014-21:57:57.233436  [**] [1:2210045:1] SURICATA STREAM Packet with invalid ack [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:80 -> y.y.y.y:29270
10/03/2014-21:57:57.237873  [**] [1:2210033:1] SURICATA STREAM FIN1 invalid ack [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:80 -> y.y.y.y:29270
10/03/2014-21:57:57.237873  [**] [1:2210045:1] SURICATA STREAM Packet with invalid ack [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:80 -> y.y.y.y:29270
10/03/2014-21:57:57.237894  [**] [1:2210032:1] SURICATA STREAM FIN1 FIN with wrong seq [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:80 -> y.y.y.y:29270
10/03/2014-21:57:57.409695  [**] [1:2210044:1] SURICATA STREAM Packet with invalid timestamp [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:29270 -> y.y.y.y:80
10/03/2014-21:57:57.409808  [**] [1:2210044:1] SURICATA STREAM Packet with invalid timestamp [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:29270 -> y.y.y.y:80
10/03/2014-21:57:57.409843  [**] [1:2210044:1] SURICATA STREAM Packet with invalid timestamp [**] [Classification: (null)] [Priority: 3] {TCP} x.x.x.x:29270 -> y.y.y.y:80
```

Now if you would like these settings to apply after a reboot you can add
the following to _/etc/network/interfaces_.

```bash
sudo nano /etc/network/interfaces
```

Now add/change the following appropriate for your corresponding
interface.

```bash
# Connected to TAP or SPAN port for traffic monitoring
auto eth1
iface eth1 inet manual
  up ifconfig $IFACE -arp up
  up ip link set $IFACE promisc on
  down ip link set $IFACE promisc off
  down ifconfig $IFACE down
  post-up for i in rx tx sg tso ufo gso gro lro; do ethtool -K $IFACE $i off; done
  post-up echo 1 > /proc/sys/net/ipv6/conf/$IFACE/disable_ipv6
```

And if you want to modify your ring buffer change the following.\\

From:

```bash
post-up for i in rx tx sg tso ufo gso gro lro; do ethtool -K $IFACE $i off; done
```

To:

```bash
post-up ethtool -G $IFACE rx 4096; for i in rx tx sg tso ufo gso gro lro; do ethtool -K $IFACE $i off; done
```

Hope this helps someone else out as well.

Enjoy!
