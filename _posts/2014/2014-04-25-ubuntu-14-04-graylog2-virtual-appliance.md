---
  title: Ubuntu 14.04 Graylog2 Virtual Appliance
  date: 2014-04-25
---

I have put together a Graylog2 prebuilt virtual appliance to share with
the community. This appliance is built on a fresh Ubuntu 14.04 LTS x64
Server and running the latest Graylog2 v0.20.1. I hope you enjoy this
and find it useful and please feel free to leave comments. You can grab
the .ova torrent from
[here](magnet:?xt=urn:btih:4KKNW3GBKJYS5MIXEXYHGMNWKWB5HNM5&dn=Ubuntu%2014.04%20x64%20Graylog2%20Appliance.ova&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a80&tr=udp%3a%2f%2ftracker.publicbt.com%3a80&tr=udp%3a%2f%2ftracker.istole.it%3a80&tr=udp%3a%2f%2ftracker.ccc.de%3a80 "Graylog2 Virtual Appliance Torrent")
or download from [here](https://t.co/yr4qZChh3S "https\://t.co/yr4qZChh3S")(Thanks [@\_lennart](https://twitter.com/_lennart "@\_lennart")).

Below are the details of the appliance. Thanks to all who helped in
seeding this for me, much appreciated.

FYI....SSH is not running on the appliance so if you want to run the
SSH server you will need to login to the console and run the following
to do so.

```bash
sudo apt-get -y install openssh-server
```

If you would rather install Graylog2 on your own; you can still use the
auto-install script
[here](http://everythingshouldbevirtual.com/ubuntu-12-04-graylog2-installation "Ubuntu 12.04 Graylog2 Installation")
and as well watch the video [here](http://everythingshouldbevirtual.com/ubuntu-graylog2-auto-install-script-video "Ubuntu Graylog2 Auto Install Script – Video").

Enjoy!

**Details**

This is a fully functional Graylog2 syslog server ready for use. This
appliance is built on a fresh Ubuntu 14.04 LTS x64 server. The version
of Graylog2 is v0.20.1 which is the latest at this time.

To login to the console of this appliance use the following details.
login: administrator
password: graylog2

To login to the web ui for Graylog2 use the following details. (Will
also be presented on your console screen when the appliance boots up)

```bash
http://applianceIP:9000
user: admin
password: password123
```

Provided by @mrlesmithjr
<http://everythingshouldbevirtual.com>
mrlesmithjr@everythingshouldbevirtual.com

> **UPDATE!!!!!! ------  As of today 09-17-2015....This post may
> or may not work to upgrade as below.**

**\*\***UPGRADING APPLIANCE**\*\***

If you are interested in upgrading the Graylog2 version running on the
appliance you can do the following to upgrade to the latest version.

From a console session logged in as administrator run the following
commands.

```bash
cd ~
cd graylog2
git pull https://github.com/mrlesmithjr/graylog2
chmod +x Upgrade_Scripts/Graylog2_Appliance_Upgrade.sh
cd ~
sudo ./graylog2/Upgrade_Scripts/Graylog2_Appliance_Upgrade.sh
```

After this completes you should be up and running with the latest
Graylog2 version.
