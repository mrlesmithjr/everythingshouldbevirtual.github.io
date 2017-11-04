---
  title: Vagrant - Multi-NIC Definitions via YAML
  categories:
    - Virtualization
  tags:
    - Vagrant
  redirect_from:
    - /vagrant-multi-nic-definitions-via-yaml
---

I am currently working on a little routing project to use as a learning
tool for others to easily consume by using Vagrant and Ansible (Inspired
by [this](http://keepingitclassless.net/2015/06/open-source-routing-practical/)
post by [Matt Oswalt](https://twitter.com/Mierdin) with added
functionality) and came across a hard to find solution, so I figured I
would share this. I am using a nodes.yml file to define the nodes to
spin up using Vagrant and had a need to define a different number of
interfaces per node. Now I know this is pretty easy when defining
everything in your Vagrantfile but I want this to be a little more
dynamic by only needing to make changes to a yaml file which is loaded
via Vagrant when spinning up.

`nodes.yml`

```yaml
---
- name: r1
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  priv_ip_1: 192.168.250.101  #HostOnly interface
  priv_ips:  #Internal only interfaces
    - ip: 192.168.12.11
      desc: 01-to-02
    - ip: 192.168.14.11
      desc: 01-to-04
    - ip: 192.168.31.11
      desc: 03-to-01
    - ip: 1.1.1.10
      desc: 'Network to Advertise'
- name: r2
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  priv_ip_1: 192.168.250.102  #HostOnly interface
  priv_ips:  #Internal only interfaces
    - ip: 192.168.23.12
      desc: 02-to-03
    - ip: 192.168.12.12
      desc: 01-to-02
    - ip: 2.2.2.10
      desc: 'Network to Advertise'
- name: r3
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  priv_ip_1: 192.168.250.103  #HostOnly interface
  priv_ips:  #Internal only interfaces
    - ip: 192.168.31.13
      desc: 03-to-01
    - ip: 192.168.23.13
      desc: 02-to-03
    - ip: 3.3.3.10
      desc: 'Network to Advertise'
- name: r4
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  priv_ip_1: 192.168.250.104  #HostOnly interface
  priv_ips:  #Internal only interfaces
    - ip: 192.168.14.14
      desc: 01-to-04
    - ip: 192.168.31.14
      desc: 03-to-01
    - ip: 192.168.41.14
      desc: utopia
    - ip: 4.4.4.10
      desc: 'Network to Advertise'
```

As you can see above the _priv_ip_1_ is the same on each node which
creates a HostOnly interface, however; the _priv_ips_ definition is
where I am defining multiple different internal links and some routers
will have more than others depending on your topology.\
Below is the corresponding Vagrantfile to spin up these nodes.

`Vagrantfile`

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
      node.vm.network "private_network", ip: nodes["priv_ip_1"]
      ints = nodes["priv_ips"]
      ints.each do |int|
        node.vm.network "private_network", ip: int["ip"], virtualbox__intnet: int["desc"]
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

The highlighted section above is the magic behind what I needed to do.
Hopefully this will help others out as well if searching for such.

> UPDATE: After doing a demo for some of my internal team members some additional
> ideas came from all of this. I have changed up the nodes.yml and
> Vagrantfile to get a little more granular and used different variables
> to make things a little clearer. So I am adding to this post to show the
> additional ways of doing this.

`nodes.yml`

```yaml
---
# Keep in mind....Vagrant will always create an initial interface as a NAT interface..Any definitions below are for adding additional interfaces.
# for network_name define a var other than remaining blank to define an internal only network. Otherwise leave blank for a host-only network.
# for DHCP leave ip and network_name vars blank
# for Static define ip var
# type should generally be private_network..Other option(s) are: public_network
- name: node-1
  box: ubuntu/trusty64
  mem: 512
  cpus: 2
  ansible_ssh_host_ip: 192.168.202.33 #always create for Ansible provisioning within nodes
  interfaces:  #Define additional interface settings
    - ip: 192.168.12.11
      auto_config: "True"
      network_name: 01-to-02
      method: static
      type: private_network
    - ip: 192.168.14.11
      auto_config: "True"
      network_name:
      method: static
      type: private_network
    - ip: 192.168.15.11
      auto_config: "False"
      network_name: 01-to-05
      method: static
      type: private_network
    - ip:
      auto_config: "True"
      network_name:
      method: dhcp
      type: private_network
```

`Vagrantfile`

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
#  config.vm.provision :ansible do |ansible|
#    ansible.playbook = "bootstrap.yml"
#  end
#  if Vagrant.has_plugin?("vagrant-cachier")
#    config.cache.scope = :box
#  end
end
```

Enjoy!
