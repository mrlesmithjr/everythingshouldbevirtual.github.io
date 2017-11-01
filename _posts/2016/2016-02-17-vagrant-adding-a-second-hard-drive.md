---
  title: Vagrant - Adding a second hard drive
---

I was just working on some Vagrant lab stuff and had a need to spin up a
Vagrant node with more than one hard drive. So I turned to google and
did some searching and finally came up with the pieces of the
Vagrantfile that needed to be added to accomplish this. So more as a
reference for myself and maybe someone else may have a need I am showing
an example Vagrantfile which does just this.

In the example below I have made the relevant configurations required
**bold** text.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
disk = './secondDisk.vdi'
Vagrant.configure(2) do |config|
  config.vm.define "iscsitarget" do |iscsitarget|
    iscsitarget.vm.box = "ubuntu/trusty64"
    iscsitarget.vm.hostname = "iscsitarget"

    iscsitarget.vm.network :private_network, ip: "192.168.202.201"

    iscsitarget.vm.provider "virtualbox" do |vb|
      unless File.exist?(disk)
        vb.customize ['createhd', '--filename', disk, '--variant', 'Fixed', '--size', 20 * 1024]
      end
      vb.memory = "1024"
      vb.customize ['storageattach', :id,  '--storagectl', 'SATAController', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk]
    end
  end
  config.vm.provision :shell, path: "provision.sh", keep_color: "true"
end
```

So what we are doing in the above example is:
Defining the variable disk= to the desired filename for the second hard
drive. Defining the new hard drive size to be created if it does not already
exist. `20 * 1024 = 20G`
And finally defining the controller and port to attach this new hard drive to.
The SATAController is what is used for my Virtualbox instance running on OSX.
This may be different depending on your setup. So it might be just SATA or
something else in your case.

And if I spin my node up and do a quick check it shows the new hard
drive as being attached and available for use.

```raw
vagrant@iscsitarget:~$ sudo fdisk -l

Disk /dev/sda: 42.9 GB, 42949672960 bytes
4 heads, 32 sectors/track, 655360 cylinders, total 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000789a2

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048    83886079    41942016   83  Linux

Disk /dev/sdb: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders, total 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/sdb doesn't contain a valid partition table
```

So there you have it and hope this may just help someone else too.

And if you want to add more than two hard-drives to your Vagrant build.
See example below:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  config.vm.define "zfs" do |zfs|
#    zfs.vm.box = "mrlesmithjr/jessie64"
    zfs.vm.box = "mrlesmithjr/trusty64"
    zfs.vm.hostname = "zfs"

    zfs.vm.network :private_network, ip: "192.168.202.201"

    zfs.vm.provider "virtualbox" do |vb|
      unless File.exist?('./secondDisk.vdi')
        vb.customize ['createhd', '--filename', './secondDisk.vdi', '--variant', 'Fixed', '--size', 10 * 1024]
      end
      unless File.exist?('./thirdDisk.vdi')
        vb.customize ['createhd', '--filename', './thirdDisk.vdi', '--variant', 'Fixed', '--size', 10 * 1024]
      end
      vb.memory = "1024"
      vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', './secondDisk.vdi']
      vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', './thirdDisk.vdi']
    end
    zfs.vm.provision :shell, path: "provision.sh", keep_color: "true"
  end
  config.vm.define "client" do |client|
#    client.vm.box = "mrlesmithjr/jessie64"
    client.vm.box = "mrlesmithjr/trusty64"
    client.vm.hostname = "client"

    client.vm.network :private_network, ip: "192.168.202.202"
    client.vm.provision :shell, inline: "sudo apt-get update && sudo apt-get -y install open-iscsi"
  end
end
```

Enjoy!
