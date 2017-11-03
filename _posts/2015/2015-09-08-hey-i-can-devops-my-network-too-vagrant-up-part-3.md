---
  title: Hey, I can DevOPS my Network too! -- Vagrant Up (Part 3)
  categories:
    - Automation
  tags:
    - Ansible
---

In the last
[post](https://everythingshouldbevirtual.com/hey-i-can-devops-my-network-too-define-nodes-part-2)
we defined our nodes to spin up with Vagrant which will be used from
here on out during our series. In this post we will actually be spinning
up the environment and begin getting a little familiar with Ansible and
the underlying `Vagrantfile`. So what is all in this `Vagrantfile` you
may be asking? Let's touch on a few things here. Remember, these are
only some of the settings that I find work very well for different
environments that I tend to spin up. Feel free to leave feedback on
other ideas or even submit a PR on GitHub and get your ideas/methods
implemented for others to consume as well.

> Dissecting the `Vagrantfile` in use in this series

One of the first things I would like to point out is the following bit
of code.

```ruby
    # Ensure yaml module is loaded
    require 'yaml'

    # Read yaml node definitions to create **Update nodes.yml to reflect any changes
    nodes = YAML.load_file('nodes.yml')
```

What is being done here is pretty self explanatory from the comments but
just to be clear on what is happening I will elaborate. The first bit of
code is telling Vagrant to ensure that the yaml module is loaded on
Vagrant up in order to read the node configurations from the `nodes.yml`
file. And you guessed it, the next part defines the variable `nodes` and
sets it to our `nodes.yml` file being loaded.

Now onto the rest of the `Vagrantfile`.

```ruby
nodes.each do |nodes|
  config.vm.define nodes["name"] do |node|
    node.vm.hostname = nodes["name"]
    node.vm.box = nodes["box"]
```

The above is setting up a loop to go through and provision each node
defined in our `nodes.yml` file. Within this loop we are defining the
hostname and Vagrant box to use in our provisioning. And those are being
gathered from the following (**bold**) values in our `nodes.yml` file.

```yaml
- name: r5
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.105  #HostOnly interface
```

Now we are defining or not defining an interface (eth1) to add to our
node(s) which will be used for ansible provisioning and management
between nodes. The variable ansible_ssh_host_ip is only used to keep
it clear and understandable. This interface will allow us to run Ansible
playbooks from within our HostOS as well as from any one of our
provisioned nodes (more on this later).

```ruby
if nodes["ansible_ssh_host_ip"] != "None"
  node.vm.network "private_network", ip: nodes["ansible_ssh_host_ip"]
end
```

The above is configured by the following (**bold**) variable.

```yaml
- name: r5
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
  ansible_ssh_host_ip: 192.168.250.105  #HostOnly interface
```

Next we are defining the variable **ints**

```ruby
ints = nodes["interfaces"]
```

And setting it to an array of defined `interfaces` (**bold**) from
`nodes.yml` which allows us to define a different number of interfaces
per node. And we reference these interfaces as `nodes['interfaces']`. Remember
from above we define `nodes` as being the yaml file we are loading. And we are
actually defining each additional interface as a variable of `ip` which is not
only handling the additional interface but also assigning the defined IP address
for the interface. So in the example below the node will actually have a
total of five interfaces. One NAT interface defined by default by
Vagrant (eth0), one Host-Only interface (ansible_ssh_host_ip) (eth1)
and three internal-only interfaces (eth2, eth3 and eth4).

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

Now the next section is where we will be looping through the defined
interfaces from above and doing some checks to decide on the type of
interface to create and if the IP address should be configured by
Vagrant or manually by another method (ie. inside
/etc/network/interfaces). The options here are limitless of course and
this is only a starting point which works well for the given scenarios
we will be using throughout this series. Feel free to explore additional
options and leave feedback. You can view the Vagrant info on
private_networks which is what we are using in this series
[here](https://docs.vagrantup.com/v2/networking/private_network.html).

```ruby
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
```

This bit of code below will setup a synced folder between our HostOS and
our Vagrant node(s) as a mountpoint within our guest(s) as /vagrant.
This basically will mount our root folder (where our Vagrantfile exists)
as /vagrant within our guest(s). We will be exploring this more in depth
as we go on in the series. You will definitely see the power of this and
also how we will rely on running Ansible playbooks within our nodes and
why defining the ansible_ssh_host_ip interface from above comes into
play.

```ruby
node.vm.synced_folder ".", "/vagrant"
```

Now for defining the number of vCPU(s) and memory to assign to our
nodes. As you can see again, we can easily define different values to
each of our nodes based on needs or roles for our environment. We will
however be setting each of these to be the same for all of our nodes in
this series.

```ruby
node.vm.provider "virtualbox" do |v|
  v.memory = nodes["mem"]
  v.cpus = nodes["cpus"]
end
```

And we will be gathering these from our nodes.yml file as seen below
(**bold**). And again feel free to adjust these as you see fit for this
series but these values will function just fine for the focus on this
series. If you do however adjust them, make sure to commit your changes
to GitHub to ensure that you have your code changes saved.

```yaml
- name: r5
  box: ubuntu/trusty64
  mem: 512
  cpus: 1
```

Now this is where our initial Ansible provisioning comes into play.
Vagrant natively has
[provisioning](https://docs.vagrantup.com/v2/provisioning/index.html)
capabilities built-in and Ansible is one of those. So during our spin up
of our environment Vagrant will actually be running a bootstrap playbook
(`bootstrap.yml`) which will be preparing our nodes to have the ability
to run Ansible from within any single one of our nodes as well as pull
down some roles from [Ansible Galaxy](https://galaxy.ansible.com/list#/users/16236)
that I have created and shared out. The next bit of code within our Vagrantfile
is what will run this Ansible provisioning.

```ruby
config.vm.provision :ansible do |ansible|
  ansible.playbook = "bootstrap.yml"
end
```

And the included bootstrap.yml playbook consists of the following. As
you can see there is a lot going on here. But ultimately what is going
on is we are installing Ansible and installing the defined galaxy_roles
(vars: galaxay_roles) within each of our nodes. You may also wish to
install these galaxy_roles within your HostOS assuming that you are
using OSX or Linux. These roles will be required to be installed within
your HostOS if you decide to run the Ansible playbooks against your
nodes from the HostOS. This is why we install Ansible  and also install
the required galaxy_roles within our nodes being deployed. This allows
us to run our Ansible playbooks and such without having to worry about
installing inside our HostOS. The remaining parts of the bootstrap
playbook is defining our host_vars/nodename files to define the
ansible_ssh_host_ip (eth1), ssh port number and the location of our
ssh_key to use allowing us to perform ssh password-less connections
between nodes. Pretty cool right? Well at least it is what I have found
works the best in these types of scenarios. And again the options are
limitless. So feel free to explore but remember to commit your code if
you change anything. Running `git status` within the folder containing
your code for this series will tell you if you have anything that needs
to be committed.
{% raw %}

```yaml
---
- hosts: all
  remote_user: vagrant
  sudo: yes
  vars:
    - galaxy_roles:
      - mrlesmithjr.bootstrap
      - mrlesmithjr.base
      - mrlesmithjr.quagga
    - install_galaxy_roles: true
    - ssh_key_path: '.vagrant/machines/{{ inventory_hostname }}/virtualbox/private_key'
    - update_host_vars: true
  roles:
  tasks:
    - name: updating apt cache
      apt: update_cache=yes cache_valid_time=3600
      when: ansible_os_family == "Debian"

    - name: installing ansible pre-reqs
      apt: name={{ item }} state=present
      with_items:
        - python-pip
        - python-dev
      when: ansible_os_family == "Debian"

    - name: adding ansible ppa
      apt_repository: repo='ppa:ansible/ansible'
      when: ansible_os_family == "Debian"

    - name: installing ansible
      apt: name=ansible state=latest
      when: ansible_os_family == "Debian"

    - name: installing ansible-galaxy roles
      shell: ansible-galaxy install {{ item }} --force
      with_items: galaxy_roles
      when: install_galaxy_roles is defined and install_galaxy_roles

    - name: ensuring host file exists in host_vars
      stat: path=./host_vars/{{ inventory_hostname }}
      delegate_to: localhost
      register: host_var
      sudo: false
      when: update_host_vars is defined and update_host_vars

    - name: creating missing host_vars
      file: path=./host_vars/{{ inventory_hostname }} state=touch
      delegate_to: localhost
      sudo: false
      when: not host_var.stat.exists

    - name: updating ansible_ssh_port
      lineinfile: dest=./host_vars/{{ inventory_hostname }} regexp="^ansible_ssh_port{{ ':' }}" line="ansible_ssh_port{{ ':' }} 22"
      delegate_to: localhost
      sudo: false
      when: update_host_vars is defined and update_host_vars

    - name: updating ansible_ssh_host
      lineinfile: dest=./host_vars/{{ inventory_hostname }} regexp="^ansible_ssh_host{{ ':' }}" line="ansible_ssh_host{{ ':' }} {{ ansible_eth1.ipv4.address }}"
      delegate_to: localhost
      sudo: false
      when: update_host_vars is defined and update_host_vars

    - name: updating ansible_ssh_key
      lineinfile: dest=./host_vars/{{ inventory_hostname }} regexp="^ansible_ssh_private_key_file{{ ':' }}" line="ansible_ssh_private_key_file{{ ':' }} {{ ssh_key_path }}"
      delegate_to: localhost
      sudo: false
      when: update_host_vars is defined and update_host_vars

    - name: ensuring host_vars is yaml formatted
      lineinfile: dest=./host_vars/{{ inventory_hostname }} regexp="---" line="---" insertbefore=BOF
      delegate_to: localhost
      sudo: false
      when: update_host_vars is defined and update_host_vars
```

{% endraw %}
And with all of this out of the way you may be asking "What is this
Vagrant Up thing you are speaking of?". Well we are now ready to see
what this is all about.

Make sure you are at a command prompt within your folder in which you
cloned your code from GitHub to.

```bash
cd vagrant-ansible-routing-template
```

List the directory to ensure that the Vagrantfile exists.

```bash
ls
...
README.md       ansible.cfg     bootstrap_ansible.sh    host_vars       hosts.example       playbook.yml
Vagrantfile        bootstrap.yml       group_vars      hosts           nodes.yml       site.yml
```

And now we are ready to Vagrant Up. Vagrant up will with one line bring
up all of our nodes defined in nodes.yml and do the initial Ansible
bootstrapping.
So with that being said let's do it.

```bash
vagrant up
```

And you should now see the following.

```raw
Bringing machine 'r1' up with 'virtualbox' provider...
Bringing machine 'r2' up with 'virtualbox' provider...
Bringing machine 'r3' up with 'virtualbox' provider...
Bringing machine 'r4' up with 'virtualbox' provider...
Bringing machine 'r5' up with 'virtualbox' provider...
....
==> r1: Importing base box 'ubuntu/trusty64'...
==> r1: Matching MAC address for NAT networking...
==> r1: Checking if box 'ubuntu/trusty64' is up to date...
==> r1: Setting the name of the VM: vagrant-ansible-routing-template_r1_1441723280031_88646
==> r1: Clearing any previously set forwarded ports...
==> r1: Clearing any previously set network interfaces...
==> r1: Preparing network interfaces based on configuration...
    r1: Adapter 1: nat
    r1: Adapter 2: hostonly
    r1: Adapter 3: intnet
    r1: Adapter 4: intnet
    r1: Adapter 5: intnet
    r1: Adapter 6: intnet
    r1: Adapter 7: intnet
==> r1: Forwarding ports...
    r1: 22 => 2222 (adapter 1)
==> r1: Running 'pre-boot' VM customizations...
==> r1: Booting VM...
==> r1: Waiting for machine to boot. This may take a few minutes...
    r1: SSH address: 127.0.0.1:2222
    r1: SSH username: vagrant
    r1: SSH auth method: private key
....
   r1: Warning: Connection timeout. Retrying...
    r1:
    r1: Vagrant insecure key detected. Vagrant will automatically replace
    r1: this with a newly generated keypair for better security.
    r1:
    r1: Inserting generated public key within guest...
    r1: Removing insecure key from the guest if it's present...
    r1: Key inserted! Disconnecting and reconnecting using new SSH key...
==> r1: Machine booted and ready!
==> r1: Checking for guest additions in VM...
    r1: The guest additions on this VM do not match the installed version of
    r1: VirtualBox! In most cases this is fine, but in rare cases it can
    r1: prevent things such as shared folders from working properly. If you see
    r1: shared folder errors, please make sure the guest additions within the
    r1: virtual machine match the version of VirtualBox you have installed on
    r1: your host and reload your VM.
    r1:
    r1: Guest Additions Version: 4.3.10
    r1: VirtualBox Version: 5.0
==> r1: Setting hostname...
==> r1: Configuring and enabling network interfaces...
==> r1: Mounting shared folders...
    r1: /vagrant => /Users/larrysmith/projects/vagrant-ansible-routing-template
==> r1: Running provisioner: ansible...

PLAY [all] ********************************************************************

GATHERING FACTS ***************************************************************
ok: [r1]

TASK: [updating apt cache] ****************************************************
ok: [r1]

TASK: [installing ansible pre-reqs] *******************************************
changed: [r1] => (item=python-pip,python-dev)

TASK: [adding ansible ppa] ****************************************************
changed: [r1]

TASK: [installing ansible] ****************************************************
changed: [r1]

TASK: [installing ansible-galaxy roles] ***************************************
changed: [r1] => (item=mrlesmithjr.bootstrap)
changed: [r1] => (item=mrlesmithjr.base)
changed: [r1] => (item=mrlesmithjr.quagga)

TASK: [ensuring host file exists in host_vars] ********************************
ok: [r1 -> localhost]

TASK: [creating missing host_vars] ********************************************
skipping: [r1]

TASK: [updating ansible_ssh_port] *********************************************
ok: [r1 -> localhost]

TASK: [updating ansible_ssh_host] *********************************************
ok: [r1 -> localhost]

TASK: [updating ansible_ssh_key] **********************************************
ok: [r1 -> localhost]

TASK: [ensuring host_vars is yaml formatted] **********************************
ok: [r1 -> localhost]

PLAY RECAP ********************************************************************
r1                         : ok=11   changed=4    unreachable=0    failed=0
....
```

You will see the above scroll by as each of our five nodes are
provisioned. So sit back and watch as they are spun up.

After a few minutes you should see our final router5 (r5) get spun up
and provisioned. After this completes all of our nodes are ready for our
next part(s) of the series. But before we end here I want to point out
some things first.

First to validate that all of our nodes are up and running.

```bash
vagrant status
....
Current machine states:

r1                        running (virtualbox)
r2                        running (virtualbox)
r3                        running (virtualbox)
r4                        running (virtualbox)
r5                        running (virtualbox)

    This environment represents multiple VMs. The VMs are all listed
    above with their current state. For more information about a specific
    VM, run `vagrant status NAME`.
```

As you can see all of our nodes are up and running. However for instance if for
some reason during the vagrant up an error occurred and stalled the provisioning.
We could continue our deployment by doing the following again.

```bash
vagrant up
```

Vagrant would then validate that each node is up and if any are found to
not be it would then provision the missing node(s).
Also if for some reason an error was to occur during `vagrant up` and
you spun up the remaining node(s) after running `vagrant up` again you
would want to ensure that Vagrant has performed the Ansible
provisioning. We can validate that Ansible provisioning has taken place
by running the following.

```raw
vagrant provision
....

PLAY [all] ********************************************************************

GATHERING FACTS ***************************************************************
ok: [r1]

TASK: [updating apt cache] ****************************************************
ok: [r1]

TASK: [installing ansible pre-reqs] *******************************************
ok: [r1] => (item=python-pip,python-dev)

TASK: [adding ansible ppa] ****************************************************
ok: [r1]

TASK: [installing ansible] ****************************************************
ok: [r1]

TASK: [installing ansible-galaxy roles] ***************************************
changed: [r1] => (item=mrlesmithjr.bootstrap)
changed: [r1] => (item=mrlesmithjr.base)
changed: [r1] => (item=mrlesmithjr.quagga)

TASK: [ensuring host file exists in host_vars] ********************************
ok: [r1 -> localhost]

TASK: [creating missing host_vars] ********************************************
skipping: [r1]

TASK: [updating ansible_ssh_port] *********************************************
ok: [r1 -> localhost]

TASK: [updating ansible_ssh_host] *********************************************
ok: [r1 -> localhost]

TASK: [updating ansible_ssh_key] **********************************************
ok: [r1 -> localhost]

TASK: [ensuring host_vars is yaml formatted] **********************************
ok: [r1 -> localhost]

PLAY RECAP ********************************************************************
r1                         : ok=11   changed=1    unreachable=0    failed=0
....
```

Now that we have validated that all nodes are up and each one has been
bootstrapped using Ansible we can look at a few more things.
Remember up above where I mentioned as part of the Ansible bootstrap
process our host_vars/nodename would be populated with some
information? Well to show this we will show the contents of one of the
files to see what is within this file.

```bash
cat host_vars/r1
....
```

```yaml
---
ansible_ssh_port: 22
ansible_ssh_host: 192.168.250.101
ansible_ssh_private_key_file: .vagrant/machines/r1/virtualbox/private_key
```

If you were to list the host_vars folder you will see all five nodes
have a file associated.

```bash
ls host_vars/
....
r1  r2  r3  r4  r5
```

Also while we are here let's check if our code has changed.

```bash
git status
....
On branch dev
Your branch is up-to-date with 'origin/dev'.
Untracked files:
  (use "git add ..." to include in what will be committed)

    .vagrant/
    host_vars/r5

nothing added to commit but untracked files present (use "git add" to track)
```

As you can see there is our new router5 (r5) host_vars file and our
hidden .vagrant folder has been created which are not being tracked
because they have not been added to our GIT repo. So we will now add
these, commit our changes and push the new code to GitHub. Also remember
we are working within our dev branch.

```bash
git add .vagrant/
git add host_vars/r5
....
git status
....
On branch dev
Your branch is up-to-date with 'origin/dev'.
Changes to be committed:
  (use "git reset HEAD ..." to unstage)

    new file:   .vagrant/machines/r1/virtualbox/action_provision
    new file:   .vagrant/machines/r1/virtualbox/action_set_name
    new file:   .vagrant/machines/r1/virtualbox/creator_uid
    new file:   .vagrant/machines/r1/virtualbox/id
    new file:   .vagrant/machines/r1/virtualbox/index_uuid
    new file:   .vagrant/machines/r1/virtualbox/private_key
    new file:   .vagrant/machines/r1/virtualbox/synced_folders
    new file:   .vagrant/machines/r2/virtualbox/action_provision
    new file:   .vagrant/machines/r2/virtualbox/action_set_name
    new file:   .vagrant/machines/r2/virtualbox/creator_uid
    new file:   .vagrant/machines/r2/virtualbox/id
    new file:   .vagrant/machines/r2/virtualbox/index_uuid
    new file:   .vagrant/machines/r2/virtualbox/private_key
    new file:   .vagrant/machines/r2/virtualbox/synced_folders
    new file:   .vagrant/machines/r3/virtualbox/action_provision
    new file:   .vagrant/machines/r3/virtualbox/action_set_name
    new file:   .vagrant/machines/r3/virtualbox/creator_uid
    new file:   .vagrant/machines/r3/virtualbox/id
    new file:   .vagrant/machines/r3/virtualbox/index_uuid
    new file:   .vagrant/machines/r3/virtualbox/private_key
    new file:   .vagrant/machines/r3/virtualbox/synced_folders
    new file:   .vagrant/machines/r4/virtualbox/action_provision
    new file:   .vagrant/machines/r4/virtualbox/action_set_name
    new file:   .vagrant/machines/r4/virtualbox/creator_uid
    new file:   .vagrant/machines/r4/virtualbox/id
    new file:   .vagrant/machines/r4/virtualbox/index_uuid
    new file:   .vagrant/machines/r4/virtualbox/private_key
    new file:   .vagrant/machines/r4/virtualbox/synced_folders
    new file:   .vagrant/machines/r5/virtualbox/action_provision
    new file:   .vagrant/machines/r5/virtualbox/action_set_name
    new file:   .vagrant/machines/r5/virtualbox/creator_uid
    new file:   .vagrant/machines/r5/virtualbox/id
    new file:   .vagrant/machines/r5/virtualbox/index_uuid
    new file:   .vagrant/machines/r5/virtualbox/private_key
    new file:   .vagrant/machines/r5/virtualbox/synced_folders
    new file:   .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
    new file:   host_vars/r5
```

As you can see now we now show several new files which need to be
committed. I will show you a shortcut (Not everyone agrees with this
method) but I tend to use this method quite often.

```bash
git commit -am "Added newly untracked files for project"
....
[dev 2ca4703] Added newly untracked files for project
 37 files changed, 176 insertions(+)
 create mode 100644 .vagrant/machines/r1/virtualbox/action_provision
 create mode 100644 .vagrant/machines/r1/virtualbox/action_set_name
 create mode 100644 .vagrant/machines/r1/virtualbox/creator_uid
 create mode 100644 .vagrant/machines/r1/virtualbox/id
 create mode 100644 .vagrant/machines/r1/virtualbox/index_uuid
 create mode 100644 .vagrant/machines/r1/virtualbox/private_key
 create mode 100644 .vagrant/machines/r1/virtualbox/synced_folders
 create mode 100644 .vagrant/machines/r2/virtualbox/action_provision
 create mode 100644 .vagrant/machines/r2/virtualbox/action_set_name
 create mode 100644 .vagrant/machines/r2/virtualbox/creator_uid
 create mode 100644 .vagrant/machines/r2/virtualbox/id
 create mode 100644 .vagrant/machines/r2/virtualbox/index_uuid
 create mode 100644 .vagrant/machines/r2/virtualbox/private_key
 create mode 100644 .vagrant/machines/r2/virtualbox/synced_folders
 create mode 100644 .vagrant/machines/r3/virtualbox/action_provision
 create mode 100644 .vagrant/machines/r3/virtualbox/action_set_name
 create mode 100644 .vagrant/machines/r3/virtualbox/creator_uid
 create mode 100644 .vagrant/machines/r3/virtualbox/id
 create mode 100644 .vagrant/machines/r3/virtualbox/index_uuid
 create mode 100644 .vagrant/machines/r3/virtualbox/private_key
 create mode 100644 .vagrant/machines/r3/virtualbox/synced_folders
 create mode 100644 .vagrant/machines/r4/virtualbox/action_provision
 create mode 100644 .vagrant/machines/r4/virtualbox/action_set_name
 create mode 100644 .vagrant/machines/r4/virtualbox/creator_uid
 create mode 100644 .vagrant/machines/r4/virtualbox/id
 create mode 100644 .vagrant/machines/r4/virtualbox/index_uuid
 create mode 100644 .vagrant/machines/r4/virtualbox/private_key
 create mode 100644 .vagrant/machines/r4/virtualbox/synced_folders
 create mode 100644 .vagrant/machines/r5/virtualbox/action_provision
 create mode 100644 .vagrant/machines/r5/virtualbox/action_set_name
 create mode 100644 .vagrant/machines/r5/virtualbox/creator_uid
 create mode 100644 .vagrant/machines/r5/virtualbox/id
 create mode 100644 .vagrant/machines/r5/virtualbox/index_uuid
 create mode 100644 .vagrant/machines/r5/virtualbox/private_key
 create mode 100644 .vagrant/machines/r5/virtualbox/synced_folders
 create mode 100644 .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
 create mode 100644 host_vars/r5
```

Now let's push our changes up to our remote GitHub repo.

```bash
git push
....
Counting objects: 47, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (19/19), done.
Writing objects: 100% (47/47), 9.69 KiB | 0 bytes/s, done.
Total 47 (delta 2), reused 0 (delta 0)
To https://github.com/everythingshouldbevirtual/vagrant-ansible-routing-template.git
   a06b879..2ca4703  dev -> dev
```

Now that all of our code changes have been committed to our dev branch
we are slowly learning why version controlling is important (I Hope). We
now have an easy way to roll back changes if something was to go wrong
for whatever reason. Again we have not touched our Master branch yet
which is our Golden code (Production). So one last quick bit around Vagrant
which we will be covering more as we proceed in this series is how to ssh to
each node independently and be able to run Ansible playbooks.
Remember we ran vagrant status and it showed us our list of nodes
(r1...r5)? Well if this were only a single node environment we could
run the following to ssh to our node.

```bash
vagrant ssh
```

But because we are in a multi-node environment we will get the following
error.

```bash
This command requires a specific VM name to target in a multi-VM environment.
```

What this is telling us is that we must specify a specific node to ssh
to, ie. r1, r2, ..., r5. So let's connect to router1 (r1).

```bash
vagrant ssh r1
....
Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.13.0-63-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Sep  8 14:55:16 UTC 2015

  System load:         0.0               IP address for eth1: 192.168.250.101
  Usage of /:          3.3% of 39.34GB   IP address for eth2: 192.168.12.11
  Memory usage:        29%               IP address for eth3: 192.168.14.11
  Swap usage:          0%                IP address for eth4: 192.168.15.11
  Processes:           73                IP address for eth5: 192.168.31.11
  Users logged in:     0                 IP address for eth6: 1.1.1.10
  IP address for eth0: 10.0.2.15

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud


Last login: Tue Sep  8 14:55:23 2015 from 192.168.250.1
vagrant@r1:~$
```

We are now connected to router1 (r1) and you can see all of our
interfaces which we defined in our nodes.yml file are configured. We
will now change to our synced /vagrant mountpoint and view what is
there.

```bash
cd /vagrant
....
ls
....
ansible.cfg  bootstrap_ansible.sh  bootstrap.yml  group_vars  hosts  hosts.example  host_vars  nodes.yml  playbook.yml  README.md  site.yml  Vagrantfile
```

See there...We show the same files/folders as within our HostOS. We can
now manipulate these files/folders either within our node(s) or within
our HostOS and they will be synced across all nodes. Also remember that
our host_vars/nodename files contain the information which tells
Ansible on how to connect to each of our other nodes without prompting
for username/password which in turn allows us to run an Ansible playbook
from within any one of our nodes against the others. So for fun let's
run our bootstrap.yml playbook from within router1 (r1). One additional
thing to note is if you view our hosts file you will notice that every
node is defined as a loopback address and non standard ssh ports because
of port forwarding. This is why we generated our host_vars/nodename
files because these files will take precedence on the variables defined
within our hosts file which in turn allows us to run Ansible within our
nodes. Without doing this we would be trying to port forward within our
node(s) to connect to the others. Pretty cool right?

```bash
cat hosts
....
# Generated by Vagrant

r1 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r1/virtualbox/private_key
r2 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r2/virtualbox/private_key
r3 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2201 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r3/virtualbox/private_key
r4 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2202 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r4/virtualbox/private_key
r5 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2203 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r5/virtualbox/private_key
```

So let's now run our Ansible bootstrap playbook from within router1
(r1)

```raw
ansible-playbook -i hosts bootstrap.yml
....
PLAY [all] ********************************************************************

GATHERING FACTS ***************************************************************
ok: [r2]
ok: [r3]
ok: [r5]
ok: [r4]
ok: [r1]

TASK: [updating apt cache] ****************************************************
ok: [r2]
ok: [r1]
ok: [r5]
ok: [r4]
ok: [r3]

TASK: [installing ansible pre-reqs] *******************************************
ok: [r1] => (item=python-pip,python-dev)
ok: [r4] => (item=python-pip,python-dev)
ok: [r3] => (item=python-pip,python-dev)
ok: [r5] => (item=python-pip,python-dev)
ok: [r2] => (item=python-pip,python-dev)

TASK: [adding ansible ppa] ****************************************************
ok: [r1]
ok: [r3]
ok: [r2]
ok: [r5]
ok: [r4]

TASK: [installing ansible] ****************************************************
ok: [r2]
ok: [r3]
ok: [r1]
ok: [r5]
ok: [r4]

TASK: [installing ansible-galaxy roles] ***************************************
changed: [r2] => (item=mrlesmithjr.bootstrap)
changed: [r4] => (item=mrlesmithjr.bootstrap)
changed: [r3] => (item=mrlesmithjr.bootstrap)
changed: [r5] => (item=mrlesmithjr.bootstrap)
changed: [r1] => (item=mrlesmithjr.bootstrap)
changed: [r4] => (item=mrlesmithjr.base)
changed: [r3] => (item=mrlesmithjr.base)
changed: [r1] => (item=mrlesmithjr.base)
changed: [r2] => (item=mrlesmithjr.base)
changed: [r5] => (item=mrlesmithjr.base)
changed: [r4] => (item=mrlesmithjr.quagga)
changed: [r3] => (item=mrlesmithjr.quagga)
changed: [r1] => (item=mrlesmithjr.quagga)
changed: [r2] => (item=mrlesmithjr.quagga)
changed: [r5] => (item=mrlesmithjr.quagga)

TASK: [ensuring host file exists in host_vars] ********************************
ok: [r2 -> localhost]
ok: [r3 -> localhost]
ok: [r1 -> localhost]
ok: [r4 -> localhost]
ok: [r5 -> localhost]

TASK: [creating missing host_vars] ********************************************
skipping: [r2]
skipping: [r1]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [updating ansible_ssh_port] *********************************************
ok: [r1 -> localhost]
ok: [r3 -> localhost]
ok: [r2 -> localhost]
ok: [r5 -> localhost]
ok: [r4 -> localhost]

TASK: [updating ansible_ssh_host] *********************************************
ok: [r3 -> localhost]
ok: [r2 -> localhost]
ok: [r4 -> localhost]
ok: [r1 -> localhost]
ok: [r5 -> localhost]

TASK: [updating ansible_ssh_key] **********************************************
ok: [r4 -> localhost]
ok: [r1 -> localhost]
ok: [r2 -> localhost]
ok: [r3 -> localhost]
ok: [r5 -> localhost]

TASK: [ensuring host_vars is yaml formatted] **********************************
ok: [r1 -> localhost]
ok: [r2 -> localhost]
ok: [r5 -> localhost]
ok: [r4 -> localhost]
ok: [r3 -> localhost]

PLAY RECAP ********************************************************************
r1                         : ok=11   changed=1    unreachable=0    failed=0
r2                         : ok=11   changed=1    unreachable=0    failed=0
r3                         : ok=11   changed=1    unreachable=0    failed=0
r4                         : ok=11   changed=1    unreachable=0    failed=0
r5                         : ok=11   changed=1    unreachable=0    failed=0
```

And there you have it, we have now ran our playbook in parallel against
all of our nodes. And that concludes this section of the series on
Vagrant Up.

Next up...[Auto-configured OSPF](https://everythingshouldbevirtual.com/hey-i-can-devops-my-network-too-auto-configured-ospf-part-4)
