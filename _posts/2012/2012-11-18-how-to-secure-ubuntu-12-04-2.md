---
  title: How to Secure Ubuntu 12.04
  date: 2012-11-18 14:54:55
---

This is just a list of a few tweaks and apps you can use to secure your
Ubuntu 12.04 LTS system (These also apply to other versions of Ubuntu).
These are definitely worth implementing on any system that may be
accessible from the internet.

## Secured Shared Memory

> NOTE: By default /dev/shm is mounted as read/write and the default permissions allow
> execute on programs, and many times httpd is attacked this way. So let's
> secure this by making the following changes

```bash
sudo nano /etc/fstab
tmpfs /dev/shm tmpfs defaults,noexec,nosuid 0 0
```

> NOTE: This will mount /dev/shm as read/write, but no execute and no permission
> to change the UID of a running program.

## Harden SSH

> NOTE: The best way to secure SSH is to disable root login and change the
> standard port tcp/22 to another port number.

We can do this by the following.

```bash
sudo nano /etc/ssh/sshd_config
Port # <change to another port other than 22>
PermitRootLogin no
```

Restart sshd

```bash
sudo /etc/init.d/ssh restart
```

## Prevent IP Spoofing

```bash
sudo nano /etc/host.conf
nospoof on #add this line to the end of the file
```

## Log scanner and banning suspicious hosts

Install DenyHosts and Fail2Ban

```bash
sudo apt-get install denyhosts fail2ban
sudo nano /etc/denyhosts.conf
```

modify the mail settings as needed

```bash
sudo nano /etc/fail2ban/jail.conf
```

Enable or disable the services you want to use by changing `enabled = true` or
`enabled=false`
Also change the SSH port if you changed from the default port of 22 from the
above section on hardening SSH

```bash
sudo /etc/init.d/fail2ban restart
```

## IDS (Intrusion Detection System)

We will use [PSAD](http://www.cipherdyne.org/psad/index.html) for Intrusion
Detection

```bash
sudo apt-get install psad
```

Create IPTables rules so PSAD will scan the logs

```bash
sudo iptables -A INPUT -j LOG
sudo iptables -A FORWARD -j LOG_
sudo nano /etc/psad/psad.conf
```

> NOTE: Reference [this](http://www.cipherdyne.org/psad/docs/config.html "http\://www.cipherdyne.org/psad/docs/config.html") link for more settings that
> can be changed within the psad.conf file\_)

change the following line `_IPT_SYSLOG_FILE`          

```bash
/var/log/messages;_ to _IPT_SYSLOG_FILE            
/var/log/syslog;_
```

Reload psad

```bash
sudo psad -R && sudo psad --sig-update && sudo psad -H_
```

## Rootkit checking tools

We will use [chkrootkit](http://www.chkrootkit.org/) and [rkhunter](http://rkhunter.sourceforge.net/). Both of these tools can be used together.

```bash
sudo apt-get install chkrootkit rkhunter
sudo chkrootkit
sudo rkhunter --update
sudo rkhunter --propupd
sudo rkhunter --check
```

## Log analysis

We are going to use logwatch for this

```bash
sudo apt-get install logwatch libdate-manip-perl
```

Follow the steps [here](https://help.ubuntu.com/community/Logwatch "https\://help.ubuntu.com/community/Logwatch") to finish the installation of
logwatch

## System Audit Security

We will be using [tiger](http://www.nongnu.org/tiger/) to do this.

```bash
sudo apt-get install tiger
```
