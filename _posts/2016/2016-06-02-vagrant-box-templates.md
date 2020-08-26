---
  title: Vagrant Box Templates
  categories:
    - Virtualization
  tags:
    - Vagrant
  redirect_from:
    - /vagrant-box-templates
---

Over the past few months I have been putting together some Vagrant box
templates based on different Linux distros all using Packer. My goal is
to have a standard methodology for building them as well as consuming
them. In this post I want to share those templates with others as well
as share some little tricks I have learned for myself that have proven
to be extremely useful.

List of current Vagrant box templates

-   Alpine Linux
-   CentOS-6
-   CentOS-7
-   Debian Jessie (server)
-   Debian Jessie (desktop)
-   Debian Wheezy
-   Fedora 22
-   Fedora 23
-   LinuxMint 17 (cinnamon)
-   OpenSuse 42.1
-   Ubuntu Precise
-   Ubuntu Trusty
-   Ubuntu Vivid
-   Ubuntu Wily
-   Ubuntu Xenial

You can check out the actual Vagrant boxes on Hashicorp's Atlas site
[here](https://atlas.hashicorp.com/mrlesmithjr).

I have also been saving these as templates on GitHub which also includes
a consistent and customizable Vagrantfile for each distro ready for
consumption. You can check these out on GitHub
[here](https://github.com/mrlesmithjr/vagrant-box-templates).

And if you are interested in pulling these templates down from GitHub to
start leveraging you can do so by doing the following...

```bash
git clone https://github.com/mrlesmithjr/vagrant-box-templates.git
```

Now you are ready to begin spinning these environments up the way they
are or you can tweak the Vagrantfile to fit your testing scenario(s).

Below is an example of what the Vagrantfile currently looks like for
Ubuntu Xenial.

{% raw %}

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

# ---- Define number of nodes to spin up ----
N = 1

# ---- Define any custom memory/cpu requirement ----
# if custom requirements are desired...ensure to set
# custom_cpu_mem == "yes" otherwise set to "no"
# By default if custom requirements are defined and set below
# any node not defined will be configured as the default...
# which is 1vCPU/512mb...So if setting custom requirements
# only define any node which requires more than the defaults.
nodes = [
  {
    :node => "node0",
    :cpu => 1,
    :mem => 4096
  }
]

# ---- Define variables below ----
additional_disks = "no"  #Define if additional drives defined should be added (yes | no)
additional_disks_controller = "SATA Controller"
additional_disks_num = 1  #Define the number of additional disks to add
additional_disks_size = 10  #Define disk size in GB
additional_nics = "no"  #Define if additional network adapters should be created (yes | no)
additional_nics_num = 1  #Define the number of additional nics to add
box = "mrlesmithjr/xenial64"  #Define Vagrant box to load
custom_cpu_mem = "no"  #Define if custom cpu and memory requirements are needed (yes| no)...defined within nodes variable above
desktop = "no"  #Define if running desktop OS (yes | no)
enable_port_forwards = "no"  #Define if port forwards should be enabled
linked_clones = "no"  #Defines if nodes should be linked from master VM (yes | no)
port_forwards = [
  {
    :node => "node0",
    :guest => 80,
    :host => 8080
  }
]
provision_nodes = "no"  #Define if provisioners should run (yes | no)
server_cpus = 1  #Define number of CPU cores...will be ignored if custom_cpu_mem == "yes"
server_memory = 512  #Define amount of memory to assign to node(s)...will be ignored if custom_cpu_mem == "yes"
subnet = "192.168.202."  #Define subnet for private_network
subnet_ip_start = 200  #Define starting last octet of the subnet range to begin addresses for node(s)

Vagrant.configure(2) do |config|

  #Iterate over nodes
  (1..N).each do |node_id|
    nid = (node_id - 1)

    config.vm.define "node#{nid}" do |node|
      node.vm.box = box
      node.vm.provider "virtualbox" do |vb|
        if linked_clones == "yes"
          vb.linked_clone = true
        end
        if custom_cpu_mem == "no"
          vb.customize ["modifyvm", :id, "--cpus", server_cpus]
          vb.customize ["modifyvm", :id, "--memory", server_memory]
        end
        if custom_cpu_mem == "yes"
          nodes.each do |cust_node|
            if cust_node[:node] == "node#{nid}"
              vb.customize ["modifyvm", :id, "--cpus", cust_node[:cpu]]
              vb.customize ["modifyvm", :id, "--memory", cust_node[:mem]]
            end
          end
        end
        if desktop == "yes"
          vb.gui = true
          vb.customize ["modifyvm", :id, "--graphicscontroller", "vboxvga"]
          vb.customize ["modifyvm", :id, "--accelerate3d", "on"]
          vb.customize ["modifyvm", :id, "--ioapic", "on"]
          vb.customize ["modifyvm", :id, "--vram", "128"]
          vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
        end
        if additional_disks == "yes"
          (1..additional_disks_num).each do |disk_num|
            dnum = (disk_num + 1)
            ddev = ("node#{nid}_Disk#{dnum}.vdi")
            unless File.exist?("#{ddev}")
              vb.customize ['createhd', '--filename', ("#{ddev}"), '--variant', 'Fixed', '--size', additional_disks_size * 1024]
            end
            vb.customize ['storageattach', :id,  '--storagectl', "#{additional_disks_controller}", '--port', dnum, '--device', 0, '--type', 'hdd', '--medium', "node#{nid}_Disk#{dnum}.vdi"]
          end
        end
      end
      node.vm.hostname = "node#{nid}"

      ### Define additional network adapters below
      if additional_nics == "yes"
        (1..additional_nics_num).each do |nic_num|
          nnum = Random.rand(0..50)
          node.vm.network :private_network, ip: subnet+"#{subnet_ip_start + nid + nnum}"
        end
      end

      ### Define port forwards below
      if enable_port_forwards == "yes"
        port_forwards.each do |pf|
          if pf[:node] == "node#{nid}"
            node.vm.network "forwarded_port", guest: pf[:guest], host: pf[:host] + nid
          end
        end
      end
      if provision_nodes == "yes"
        if node_id == N
          node.vm.provision "ansible" do |ansible|  #runs bootstrap Ansible playbook
            ansible.limit = "all"
            ansible.playbook = "bootstrap.yml"
          end
          node.vm.provision "ansible" do |ansible|  #runs Ansible playbook for installing roles/executing tasks
            ansible.limit = "all"
            ansible.playbook = "playbook.yml"
            ansible.groups = {
              "test-nodes" => [
                "node0"
              ]
            }
          end
        end
      end

    end
  end
  if provision_nodes == "yes"
    config.vm.provision :shell, path: "bootstrap.sh", keep_color: "true"  #runs initial shell script
  end
end
```

{% endraw %}
So there you have it...Just find the environment that you would like to
spin up and give it a go.

Enjoy!
