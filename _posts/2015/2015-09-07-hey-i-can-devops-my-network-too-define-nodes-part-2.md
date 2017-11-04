---
  title: Hey, I can DevOPS my Network too! -- Define Nodes (Part 2)
  categories:
    - Automation
  tags:
    - Ansible
  redirect_from:
    - /hey-i-can-devops-my-network-too-define-nodes-part-2
---

In the [previous post](https://everythingshouldbevirtual.com/hey-i-can-devops-my-network-too-prep-work-part-1)
we prepped our environment in order to get everything ready to begin
building out our environment. Which means we are now ready to finalize
our node definitions to spin up our environment with Vagrant. So to
begin with if you look at the drawing below, which was also shown in the
intro post, and compare the drawing to our _nodes.yml_ file which is in
the root of the downloaded repo you will notice that router5 (r5) is
missing.

![ossrouting-bgp-drawing - New Page](../../assets/ossrouting-bgp-drawing-New-Page-300x232.png)

Below is the original _nodes.yml_

```yaml
---
- name: r1
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.101  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.12.11
      auto_config: "True"
      network_name: 01-to-02
      method: static
      type: private_network
    - ip: 192.168.14.11
      auto_config: "True"
      network_name: 01-to-04
      method: static
      type: private_network
    - ip: 192.168.31.11
      auto_config: "True"
      network_name: 03-to-01
      method: static
      type: private_network
    - ip: 1.1.1.10
      auto_config: "True"
      network_name: 1-1-1
      method: static
      type: private_network
- name: r2
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.102  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.23.12
      auto_config: "True"
      network_name: 02-to-03
      method: static
      type: private_network
    - ip: 192.168.12.12
      auto_config: "True"
      network_name: 01-to-02
      method: static
      type: private_network
    - ip: 2.2.2.10
      auto_config: "True"
      network_name: 2-2-2
      method: static
      type: private_network
- name: r3
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.103  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.31.13
      auto_config: "True"
      network_name: 03-to-01
      method: static
      type: private_network
    - ip: 192.168.23.13
      auto_config: "True"
      network_name: 02-to-03
      method: static
      type: private_network
    - ip: 3.3.3.10
      auto_config: "True"
      network_name: 3-3-3
      method: static
      type: private_network
- name: r4
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.104  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.14.14
      auto_config: "True"
      network_name: 01-to-04
      method: static
      type: private_network
    - ip: 192.168.31.14
      auto_config: "True"
      network_name: 03-to-01
      method: static
      type: private_network
    - ip: 192.168.41.14
      auto_config: "True"
      network_name: utopia
      method: static
      type: private_network
    - ip: 4.4.4.10
      auto_config: "True"
      network_name: 4-4-4
      method: static
      type: private_network
```

So let's first add router5 (r5) to our _nodes.yml_ file.

Open up _nodes.yml_ with your editor of choice and add the following to
the end of the file.

```yaml
- name: r5
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.105  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.15.15
      auto_config: "True"
      network_name: 01-to-05
      method: static
      type: private_network
    - ip: 192.168.51.15
      auto_config: "True"
      network_name: utopia
      method: static
      type: private_network
    - ip: 5.5.5.10
      auto_config: "True"
      network_name: 5-5-5
      method: static
      type: private_network
```

If you also notice, router1 (r1) is missing the interface to connect to
the newly added router5 (r5) as well. So we will need to go ahead and
add that interface to router1 (r1) which is shown below in bold.

```yaml
- name: r1
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.101  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.12.11
      auto_config: "True"
      network_name: 01-to-02
      method: static
      type: private_network
    - ip: 192.168.14.11
      auto_config: "True"
      network_name: 01-to-04
      method: static
      type: private_network
    - ip: 192.168.15.11
      auto_config: "True"
      network_name: 01-to-05
      method: static
      type: private_network
    - ip: 192.168.31.11
      auto_config: "True"
      network_name: 03-to-01
      method: static
      type: private_network
    - ip: 1.1.1.10
      auto_config: "True"
      network_name: 1-1-1
      method: static
      type: private_network
```

So now the whole _nodes.yml_ file should look like below.

```yaml
---
- name: r1
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.101  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.12.11
      auto_config: "True"
      network_name: 01-to-02
      method: static
      type: private_network
    - ip: 192.168.14.11
      auto_config: "True"
      network_name: 01-to-04
      method: static
      type: private_network
    - ip: 192.168.15.11
      auto_config: "True"
      network_name: 01-to-05
      method: static
      type: private_network
    - ip: 192.168.31.11
      auto_config: "True"
      network_name: 03-to-01
      method: static
      type: private_network
    - ip: 1.1.1.10
      auto_config: "True"
      network_name: 1-1-1
      method: static
      type: private_network
- name: r2
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.102  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.23.12
      auto_config: "True"
      network_name: 02-to-03
      method: static
      type: private_network
    - ip: 192.168.12.12
      auto_config: "True"
      network_name: 01-to-02
      method: static
      type: private_network
    - ip: 2.2.2.10
      auto_config: "True"
      network_name: 2-2-2
      method: static
      type: private_network
- name: r3
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.103  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.31.13
      auto_config: "True"
      network_name: 03-to-01
      method: static
      type: private_network
    - ip: 192.168.23.13
      auto_config: "True"
      network_name: 02-to-03
      method: static
      type: private_network
    - ip: 3.3.3.10
      auto_config: "True"
      network_name: 3-3-3
      method: static
      type: private_network
- name: r4
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.104  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.14.14
      auto_config: "True"
      network_name: 01-to-04
      method: static
      type: private_network
    - ip: 192.168.31.14
      auto_config: "True"
      network_name: 03-to-01
      method: static
      type: private_network
    - ip: 192.168.41.14
      auto_config: "True"
      network_name: utopia
      method: static
      type: private_network
    - ip: 4.4.4.10
      auto_config: "True"
      network_name: 4-4-4
      method: static
      type: private_network
- name: r5
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.105  #HostOnly interface
  interfaces:  #Internal only interfaces
    - ip: 192.168.15.15
      auto_config: "True"
      network_name: 01-to-05
      method: static
      type: private_network
    - ip: 192.168.51.15
      auto_config: "True"
      network_name: utopia
      method: static
      type: private_network
    - ip: 5.5.5.10
      auto_config: "True"
      network_name: 5-5-5
      method: static
      type: private_network
```

At this point you are probably wondering what all of these settings are
for in _nodes.yml_, correct? So let's go over what they are for.

-   name: defines the name of the node that Vagrant will define as the
    hostname
-   box: defines the Vagrant Box that will be used as the OS for the
    node
-   mem: defines the amount of memory to allocate to the node
-   cpus: defines the number of vCPUS to allocate to the node
-   ansible_ssh_host_ip: defines the interface/IP that will be
    defined as eth1 and used for all Ansible related tasks
-   interfaces:
    -   ip: defines the IP address for each additional interface to
        create
    -   auto_config: defines if Vagrant should automatically configure
        the interface with the defined IP address. If set to False you
        can configure /etc/network/interfaces manually outside of
        Vagrant.
    -   network_name: defines the internal network name that VirtualBox
        will place the interface on. Node interfaces need to reside on
        the same network_name in order to communicate internally to
        VirtualBox
    -   method: defines if the interface should be static or dhcp
    -   type: defines if interface should be NAT, Bridged, Internal
        ("private_network") or Host-Only...more info
        [here](https://www.virtualbox.org/manual/ch06.html). We are
        using private_network which is internal only to the nodes. Not
        accessible from our HostOS.

Now let's go ahead and commit our changed code to GitHub.\
Check the status of our local git repo.

```bash
git status
........
On branch dev
Changes not staged for commit:
  (use "git add ..." to update what will be committed)
  (use "git checkout -- ..." to discard changes in working directory)

    modified:   nodes.yml

no changes added to commit (use "git add" and/or "git commit -a")
```

The above shows us that nodes.yml has been modified and we should commit
the changes.

```bash
git commit -a
```

Now type in a description of whatever you want...I am using "updated
nodes.yml in preparation of Vagrant deployment" and save the file.

```bash
[dev a06b879] updated nodes.yml in preparation of Vagrant deployment
 1 file changed, 26 insertions(+)
```

Now let's push our changes to GitHub into our dev branch.

```bash
git push --set-upstream origin dev
.........
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 413 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
To https://github.com/everythingshouldbevirtual/vagrant-ansible-routing-template.git
   caf5002..a06b879  dev -> dev
Branch dev set up to track remote branch dev from origin.
```

The other important file here at this time is our _Vagrantfile_. This is
the file that Vagrant will use to spin up the nodes that we have defined
in our _nodes.yml_ file. There should not be any changes required at
this time in this file. However if you do decide to make changes, make
sure to commit those changes and push up to your GitHub repo. Below is
what our _Vagrantfile_ looks like.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Ensure yaml module is loaded
require 'yaml'

# Read yaml node definitions to create **Update nodes.yml to reflect any changes
nodes = YAML.load_file('nodes.yml')

Vagrant.configure(2) do |config|
#  config.ssh.insert_key = false
#  config.vm.provision :shell, path: "bootstrap.sh"

  nodes.each do |nodes|
    config.vm.define nodes["name"] do |node|
      node.vm.hostname = nodes["name"]
      node.vm.box = nodes["box"]
#      node.vm.provision :shell, path: "bootstrap_ansible.sh"
      if nodes["ansible_ssh_host_ip"] != "None"
        node.vm.network "private_network", ip: nodes["ansible_ssh_host_ip"]
      end
      ints = nodes["interfaces"]
      ints.each do |int|
        if int["method"] == "static" and int["type"] == "private_network" and int["network_name"] != "None" and int["auto_config"] == "True"
          node.vm.network "private_network", ip: int["ip"], virtualbox__intnet: int["network_name"]
        end
        if int["method"] == "static" and int["type"] == "private_network" and int["network_name"] != "None" and int["auto_config"] == "False"
          node.vm.network "private_network", ip: int["ip"], virtualbox__intnet: int["network_name"], auto_config: false
        end
        if int["method"] == "static" and int["type"] == "private_network" and int["network_name"] == "None" and int["auto_config"] == "True"
          node.vm.network "private_network", ip: int["ip"]
        end
        if int["method"] == "static" and int["type"] == "private_network" and int["network_name"] == "None" and int["auto_config"] == "False"
          node.vm.network "private_network", ip: int["ip"], auto_config: false
        end
        if int["method"] == "dhcp" and int["type"] == "private_network"
          node.vm.network "private_network", type: "dhcp"
        end
      end
      node.vm.synced_folder ".", "/vagrant"
      node.vm.provider "virtualbox" do |v|
        v.memory = nodes["mem"]
        v.cpus = nodes["cpus"]
      end
    end
  end
  config.vm.provision :ansible do |ansible|
    ansible.playbook = "bootstrap.yml"
  end
#  if Vagrant.has_plugin?("vagrant-cachier")
#    config.cache.scope = :box
#  end
end
```

So this concludes our node definitions which we will be using here on
out for this series.

Up next...[Vagrant UP](https://everythingshouldbevirtual.com/hey-i-can-devops-my-network-too-vagrant-up-part-3)...
