---
title: KVM - VM Templates
---

It has been a minute since I had done any KVM based VMs so I wanted to
share some little tidbits on creating your KVM templates. I will be
focusing on Debian/Ubuntu here for now so YMMV.

**Update: 01/07/2017 - CentOS added**

First, let's install a few pre-req packages:

```bash
sudo apt-get install -y virtinst libosinfo-bin libguestfs-tools virt-top
```

Below is a shell script that I have created which will allow me to
provision my baseline template easily. You will need to adjust based on
your needs. I am using **_br0_** for my **_NETWORK_BRIDGE_** which
allows me to connect directly to my external network rather than a
default NAT connection. So you may want to use the built-in
default **_virbr0_** instead.

{% raw %}

```bash
#!/usr/bin/env bash

# Larry Smith Jr.
# @mrlesmithjr
# http://everythingshouldbevirtual.com

DISK_FORMAT="qcow2" # Define VM disk image format.. qcow2|img
DISK_PATH="/var/lib/libvirt/images" # Define path where VM disk images are stored
DISK_SIZE="36" # Define disk size in GB
EXTRA_ARGS="console=ttyS0,115200n8 serial"
LOCATION="http://archive.ubuntu.com/ubuntu/dists/xenial/main/installer-amd64/"
NETWORK_BRIDGE="br0" # virbr0
OS_TYPE="linux" # Define OS type to install... linux|windows
OS_VARIANT="ubuntu16.04" # ubuntu14.04|debian8|centos7.0
PRESEED_FILE="$DISK_PATH/preseeds/ubuntu1604/preseed.cfg" # Define preseed file for Debian based installs if desired
PRESEED_INSTALL="true" # Define if preseed install is desired
RAM="512" # Define memory to allocate to VM in MB... 512|1024|2048
VCPUS="1" # Define number of vCPUs to allocate to VM
VMName="ubuntu1604" # Define name of VM to create

# Provision VM with preseed.cfg
if [ "$PRESEED_INSTALL" = false ]; then
virt-install \
--name $VMName \
--ram $RAM \
--disk path=$DISK_PATH/$VMName.$DISK_FORMAT,size=$DISK_SIZE \
--vcpus $VCPUS \
--os-type $OS_TYPE \
--os-variant $OS_VARIANT \
--network bridge=$NETWORK_BRIDGE \
--graphics none \
--console pty,target_type=serial \
--location $LOCATION \
--extra-args "$EXTRA_ARGS"
fi

# Provision VM with preseed.cfg
if [ "$PRESEED_INSTALL" = true ]; then
virt-install \
--name $VMName \
--ram $RAM \
--disk path=$DISK_PATH/$VMName.$DISK_FORMAT,size=$DISK_SIZE \
--vcpus $VCPUS \
--os-type $OS_TYPE \
--os-variant $OS_VARIANT \
--network bridge=$NETWORK_BRIDGE \
--graphics none \
--console pty,target_type=serial \
--location $LOCATION \
--initrd-inject=$PRESEED_FILE \
--noreboot \
--extra-args "$EXTRA_ARGS"
fi
```

{% endraw %}

Below are _preseed_ files which work for Ubuntu 16.04 and Debian Jessie
as they are based on systemd and on reboot using virsh console the login
prompt is enabled to work correctly. That bit of magic works in the
**_late_command_** section at the end. You will also want to change the
default user and password as they are defined as **_ubuntu/ubuntu_** and
**_debian/debian_**.

{% raw %}

```raw
# Ubuntu 16.04 preseed config
# Larry Smith Jr.
# @mrlesmithjr
# http://everythingshouldbevirtual.com

### Localization
d-i debian-installer/locale string en_US
d-i console-setup/ask_detect boolean false
d-i keyboard-configuration/layoutcode string us

### Network configuration
d-i netcfg/choose_interface select auto
d-i netcfg/get_hostname string unassigned-hostname
d-i netcfg/get_domain string domain.com
d-i netcfg/wireless_wep string

### Mirror settings
d-i mirror/http/countries select US
d-i mirror/country string US
d-i mirror/http/hostname string us.archive.ubuntu.com
d-i mirror/http/directory string /images/Ubuntu/16.04
d-i mirror/http/mirror select us.archive.ubuntu.com
d-i mirror/http/proxy string

### Clock and time zone setup
d-i clock-setup/utc boolean true
d-i time/zone string US/Eastern
d-i clock-setup/ntp boolean true

### Partitioning
d-i partman-auto/disk string /dev/vda
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-lvm/confirm boolean false
d-i partman-lvm/confirm_nooverwrite boolean true
d-i partman-auto-lvm/guided_size string max
# - atomic: all files in one partition
# - home:   separate /home partition
# - multi:  separate /home, /usr, /var, and /tmp partitions
d-i partman-auto/choose_recipe select atomic
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
d-i partman-md/confirm boolean true
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm_nooverwrite boolean true

### Account setup
d-i passwd/root-login boolean false
d-i passwd/user-fullname string ubuntu
d-i passwd/username string ubuntu
d-i passwd/user-password password ubuntu
d-i passwd/user-password-again password ubuntu
# Use mkpasswd -m sha-512 to create an MD5 hash password
#d-i passwd/user-password-crypted password [MD5 hash]
d-i user-setup/allow-password-weak boolean true
d-i user-setup/encrypt-home boolean false

### Apt setup

### Package selection
tasksel tasksel/first multiselect
# Individual additional packages to install
d-i pkgsel/include string git openssh-server python-simplejson sudo
d-i pkgsel/update-policy select none
d-i pkgsel/upgrade select none
popularity-contest popularity-contest/participate boolean false

### Boot loader installation
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true

### Finishing up the installation
d-i finish-install/reboot_in_progress note
d-i debian-installer/exit/poweroff boolean true

### Preseeding other packages

#### Advanced options
d-i preseed/late_command string \
echo "ubuntu    ALL=(ALL) NOPASSWD:ALL" >> /target/etc/sudoers; \
in-target ln -s /lib/systemd/system/serial-getty@.service /etc/systemd/system/getty.target.wants/serial-getty@ttyS0.service; \
rm -f /target/etc/ssh/ssh_host_*; \
in-target sed -i -e 's|exit 0||' /etc/rc.local; \
in-target sed -i -e 's|.*test -f /etc/ssh/ssh_host_dsa_key.*||' /etc/rc.local; \
in-target bash -c 'echo "test -f /etc/ssh/ssh_host_dsa_key || dpkg-reconfigure openssh-server" >> /etc/rc.local'; \
in-target bash -c 'echo "exit 0" >> /etc/rc.local'; \
cat /dev/null > /target/etc/hostname
```

{% endraw %}

{% raw %}

```raw
# Debian 8 preseed config
# Larry Smith Jr.
# @mrlesmithjr
# http://everythingshouldbevirtual.com

### Localization
d-i debian-installer/locale string en_US
d-i debian-installer/language string en
d-i debian-installer/country string US
d-i debian-installer/locale string en_US.UTF-8
d-i localechooser/supported-locales multiselect en_US.UTF-8

# Keyboard selection.
d-i console-tools/archs select at
d-i console-keymaps-at/keymap select us
d-i keyboard-configuration/xkb-keymap select us

### Network configuration
d-i netcfg/choose_interface select auto
d-i netcfg/get_hostname string unassigned-hostname
d-i netcfg/get_domain string unassigned-domain
d-i netcfg/wireless_wep string
d-i hw-detect/load_firmware boolean true

### Mirror settings
d-i mirror/country string manual
d-i mirror/http/hostname string http.debian.net
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string
d-i mirror/suite string jessie

### Clock and time zone setup
d-i clock-setup/utc boolean true
d-i time/zone string US/Eastern
d-i clock-setup/ntp boolean true

### Partitioning
d-i partman-auto/disk string /dev/vda
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-lvm/confirm boolean false
d-i partman-lvm/confirm_nooverwrite boolean true
d-i partman-auto-lvm/guided_size string max
# - atomic: all files in one partition
# - home:   separate /home partition
# - multi:  separate /home, /usr, /var, and /tmp partitions
d-i partman-auto/choose_recipe select atomic
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
d-i partman-md/confirm boolean true
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm_nooverwrite boolean true

### Account setup
d-i passwd/root-login boolean false
d-i passwd/user-fullname string Debian User
d-i passwd/username string debian
d-i passwd/user-password password debian
d-i passwd/user-password-again password debian
# Use mkpasswd -m sha-512 to create an MD5 hash password
#d-i passwd/user-password-crypted password [MD5 hash]
d-i user-setup/allow-password-weak boolean true
d-i user-setup/encrypt-home boolean false

### Apt setup
d-i apt-setup/non-free boolean true
d-i apt-setup/contrib boolean true

### Package selection
tasksel tasksel/first multiselect
# Individual additional packages to install
d-i pkgsel/include string git openssh-server python-simplejson sudo
d-i pkgsel/update-policy select none
d-i pkgsel/upgrade select none
popularity-contest popularity-contest/participate boolean false

### Boot loader installation
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true
d-i grub-installer/bootdev string /dev/vda
#d-i grub-installer/bootdev string default

### Finishing up the installation
d-i finish-install/reboot_in_progress note
d-i debian-installer/exit/poweroff boolean true

#### Advanced options
d-i preseed/late_command string \
echo "debian    ALL=(ALL) NOPASSWD:ALL" >> /target/etc/sudoers; \
in-target ln -s /lib/systemd/system/serial-getty@.service /etc/systemd/system/getty.target.wants/serial-getty@ttyS0.service; \
rm -f /target/etc/ssh/ssh_host_*; \
in-target sed -i -e 's|exit 0||' /etc/rc.local; \
in-target sed -i -e 's|.*test -f /etc/ssh/ssh_host_dsa_key.*||' /etc/rc.local; \
in-target bash -c 'echo "test -f /etc/ssh/ssh_host_dsa_key || dpkg-reconfigure openssh-server" >> /etc/rc.local'; \
in-target bash -c 'echo "exit 0" >> /etc/rc.local'; \
cat /dev/null > /target/etc/hostname
```

{% endraw %}

And this little script can be executed within your VM in order to prep
it to be used as a template by cleaning up some important things. Or
better yet, if you look closely at the above _preseed_ files I have
included those important cleanup steps in the **_late_command_** steps,
so you do not have to run this script at all if you use the _preseed_
files.

{% raw %}

```bash
#!/bin/bash

#grab Ubuntu Codename
codename="$(lsb_release -c | awk {'print $2}')"

#update apt-cache
apt-get update

#Stop services for cleanup
service rsyslog stop

#clear audit logs
if [ -f /var/log/audit/audit.log ]; then
    cat /dev/null > /var/log/audit/audit.log
fi
if [ -f /var/log/wtmp ]; then
    cat /dev/null > /var/log/wtmp
fi
if [ -f /var/log/lastlog ]; then
    cat /dev/null > /var/log/lastlog
fi

#cleanup persistent udev rules
if [ -f /etc/udev/rules.d/70-persistent-net.rules ]; then
    rm /etc/udev/rules.d/70-persistent-net.rules
fi

#cleanup /tmp directories
rm -rf /tmp/*
rm -rf /var/tmp/*

#cleanup current ssh keys
rm -f /etc/ssh/ssh_host_*

#add check for ssh keys on reboot...regenerate if neccessary
sed -i -e 's|exit 0||' /etc/rc.local
sed -i -e 's|.*test -f /etc/ssh/ssh_host_dsa_key.*||' /etc/rc.local
bash -c 'echo "test -f /etc/ssh/ssh_host_dsa_key || dpkg-reconfigure openssh-server" >> /etc/rc.local'
bash -c 'echo "exit 0" >> /etc/rc.local'

#reset hostname
cat /dev/null > /etc/hostname

#cleanup apt
apt-get clean

#cleanup shell history
history -w
history -c
```

{% endraw %}

So how do we wrap all of this up into an easy format to consume? You can
quite simply copy/paste the script and appropriate _preseed_ file into
your folder structure that you use. Make sure to name the preseed file
as **_preseed.cfg_** otherwise it will not work with **_virt-install_**.
Also, make sure to modify the variables
for **\*DISK_PATH**,\* **_PRESEED_FILE_**, and **_NETWORK_BRIDGE _**as
appropriate. And once you have completed that you are ready to run the
script and watch the VM build and get ready for template usage. Also
note, that when using the _preseed_ file with our script, the VM will
actually shutdown at the end of the installation. This is the desired
state as we want everything including our non-generated SSH keys to be
clear for template usage.

Build template:

```bash
chmod +x libvirt_install_vm.sh
sudo ./libvirt_install_vm.sh
```

Clone template (Note: Replace OURNEWVM with the name you desire for the
cloned VM):

```bash
sudo virt-clone -o ubuntu1604 -n OURNEWVM -f /var/lib/libvirt/images/OURNEWVM.qcow2
sudo virsh start OURNEWVM --console
```

And there you have it. A new shiny KVM based Ubuntu template ready for
cloning. Stay tuned as I will be taking this process further with some
Ansible automation to fully deploy and etc.

**Update: CentOS Info**

Below you will find the script and ks.cfg files to use with CentOS.

{% raw %}

```bash
#!/usr/bin/env bash

# Larry Smith Jr.
# @mrlesmithjr
# http://everythingshouldbevirtual.com

DISK_FORMAT="qcow2" # Define VM disk image format.. qcow2|img
DISK_PATH="/var/lib/libvirt/images" # Define path where VM disk images are stored
DISK_SIZE="36" # Define disk size in GB
EXTRA_ARGS="ks=file:/ks.cfg network --bootproto=dhcp --device=eth0 console=ttyS0,115200n8 serial"
LOCATION="http://mirror.centos.org/centos/7/os/x86_64/"
NETWORK_BRIDGE="br0" # virbr0
OS_TYPE="linux" # Define OS type to install... linux|windows
OS_VARIANT="centos7.0" # ubuntu14.04|debian8|centos7.0
KICKSTART_FILE="$DISK_PATH/kickstart/centos7/ks.cfg" # Define kickstart file for CentOS based installs if desired
KICKSTART_INSTALL="true" # Define if kickstart install is desired
RAM="2048" # Define memory to allocate to VM in MB... 512|1024|2048
VCPUS="1" # Define number of vCPUs to allocate to VM
VMName="centos7" # Define name of VM to create

# Provision VM without ks.cfg
if [ "$KICKSTART_INSTALL" = false ]; then
virt-install \
--name $VMName \
--ram $RAM \
--disk path=$DISK_PATH/$VMName.$DISK_FORMAT,size=$DISK_SIZE \
--vcpus $VCPUS \
--os-type $OS_TYPE \
--os-variant $OS_VARIANT \
--network bridge=$NETWORK_BRIDGE \
--graphics none \
--console pty,target_type=serial \
--location $LOCATION \
--extra-args "$EXTRA_ARGS"
fi

# Provision VM with ks.cfg
if [ "$KICKSTART_INSTALL" = true ]; then
virt-install \
--name $VMName \
--ram $RAM \
--disk path=$DISK_PATH/$VMName.$DISK_FORMAT,size=$DISK_SIZE \
--vcpus $VCPUS \
--os-type $OS_TYPE \
--os-variant $OS_VARIANT \
--network bridge=$NETWORK_BRIDGE \
--graphics none \
--console pty,target_type=serial \
--location $LOCATION \
--initrd-inject=$KICKSTART_FILE \
--noreboot \
--extra-args "$EXTRA_ARGS"
fi
```

{% endraw %}

{% raw %}

```bash
# CentOS 7 kickstart config
# Larry Smith Jr.
# @mrlesmithjr
# http://everythingshouldbevirtual.com

lang en_US
keyboard us
timezone America/New_York --isUtc
rootpw r00tme
#platform x86, AMD64, or Intel EM64T
text
url --url="http://mirror.centos.org/centos/7/os/x86_64/"
bootloader --location=mbr
zerombr
clearpart --all --initlabel --drives=vda
autopart  --type=lvm
auth --passalgo=sha512 --useshadow
selinux --enforcing
firewall --enabled
network  --bootproto=dhcp --device=eth0 --ipv6=auto --activate
skipx
firstboot --disable
ignoredisk --only-use=vda
%packages
@base
%end
%post
rm -f /etc/ssh/*key*
%end
```

{% endraw %}

Enjoy!
