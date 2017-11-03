---
  title: Installing Nagios on Ubuntu 12.04
  date: 2013-07-09 19:29:32
---

This is just a quick Nagios setup guide for installing on Ubuntu. This
will get you up and running quickly. May create a github script at some
point to auto install.

```bash
sudo apt-get install wget build-essential apache2 php5-gd libgd2-xpm libgd2-xpm-dev libapache2-mod-php5 libssl-dev sendmail mrtg

cd /tmp
wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-3.5.0.tar.gz
wget http://prdownloads.sourceforge.net/sourceforge/nagiosplug/nagios-plugins-1.4.16.tar.gz

useradd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios

cd nagios
./configure --with-nagios-group=nagios --with-command-group=nagcmd
./configure --with-mail=/usr/bin/sendmail

make all
make install
make install-init
make install-config
make install-commandmode
make install-webconf

cp -R contrib/eventhandlers/ /usr/local/nagios/libexec/
chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

/etc/init.d/nagios start
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
cd /tmp/nagios-plugins-1.4.15
./configure --with-nagios-user=nagios --with-nagios-group=nagios
make
make install

ln -s /etc/init.d/nagios /etc/rcS.d/S99nagios
```

Now open your browser and login to your new Nagios Web UI.

<http://nagios.server.ip/nagios>
