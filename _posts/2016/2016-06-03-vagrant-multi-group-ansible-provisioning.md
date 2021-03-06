---
  title: Vagrant - Multi-Group Ansible Provisioning
  categories:
    - Automation
    - Virtualization
  tags:
    - Ansible
    - Vagrant
  redirect_from:
    - /vagrant-multi-group-ansible-provisioning
---

As I continue to develop different scenario testing I continually try to
leverage a common Vagrantfile across scenarios while only having to
adjust variables if required for these scenarios. With one of those
being defining Ansible groups to provision with Vagrant. So I wanted to
share with others what I have found to be a good working solution for
just that. Check out this [post](https://everythingshouldbevirtual.com/vagrant-complex-vagrantfile-configurations)
if you are interested in what I have been using as a common Vagrantfile.

In this scenario I will be spinning up 5 Vagrant boxes and would like
each box to reside in an Ansible group or groups. So if you were to use
the link above as a reference you could adjust the following settings
and vagrant up.

Change the following settings:

From:

```ruby
    N = 1
```

To:

```ruby
    N = 5
```

From:

```ruby
    ansible_groups = {
      "test-nodes" => [
        "node[0:#{N-1}]"
      ]
    }
```

To:

```ruby
    ansible_groups = {
      "db-nodes" => [
        "node0"
      ],
      "test-nodes" => [
        "node3"
      ],
      "random-nodes" => [
        "node[4:#{N-1}]"
      ],
      "mixed-nodes" => [
        "node1",
        "node4"
      ],
      "renamed-nodes" => [
        "node1"
      ]
    }
```

And after we do a vagrant up our new Vagrant Ansible inventory file
should look similar to..

```raw
# Generated by Vagrant

node0 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/larrysmithjr/projects/Vagrant/ansible/.vagrant/machines/node0/virtualbox/private_key'
node1 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/larrysmithjr/projects/Vagrant/ansible/.vagrant/machines/node1/virtualbox/private_key'
node2 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2201 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/larrysmithjr/projects/Vagrant/ansible/.vagrant/machines/node2/virtualbox/private_key'
node3 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2202 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/larrysmithjr/projects/Vagrant/ansible/.vagrant/machines/node3/virtualbox/private_key'
node4 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2203 ansible_ssh_user='vagrant' ansible_ssh_private_key_file='/Users/larrysmithjr/projects/Vagrant/ansible/.vagrant/machines/node4/virtualbox/private_key'

[db-nodes]
node0

[test-nodes]
node3

[random-nodes]
node[4:4]

[mixed-nodes]
node1
node4

[renamed-nodes]
node1
```

We can now provision our nodes successfully with our Ansible playbook
based on groups.

Example playbook:

{% raw %}

```yaml
- hosts: db-nodes
  become: true
  vars:
    - django_db_type: 'mysql'
    - install_django: true
    - mysql_allow_remote_connections: true
    - mysql_root_password: 'root'
    - pri_domain_name: 'test.vagrant.local'
  roles:
    - role: ansible-django
      when: install_django
    - role: ansible-dropbox-nsot
    - role: ansible-mariadb-mysql
  tasks:
    - name: Installing Pre-Reqs
      apt:
        name: "{{ item }}"
        state: "present"
      with_items:
        - curl
        - libapache2-mod-php5
        - libcurl3
        - libsqlite3-dev
        - php5-curl
        - php5-mcrypt
        - php5-sqlite
        - sqlite3
      when: ansible_os_family == "Debian"

    - name: Allowing Root Login to MySQL from Anywhere
      mysql_user:
        name: root
        host: "{{ item }}"
        password: "{{ mysql_root_password }}"
        priv: "*.*:ALL,GRANT"
      with_items:
        - '%'

    - name: Creating Django_DB (If Installed)
      mysql_db:
        name: "django_db"
        state: "present"
      when: install_django

    - name: Installing Django Related Apps
      pip:
        name: "{{ item }}"
        state: "present"
      with_items:
        - 'django-markdown-deux'
        - 'django-tables2'
      when: install_django

- hosts: all
  become: true
  vars:
    - pri_domain_name: 'test.vagrant.local'
  roles:
    - role: ansible-inventory
  #    ignore_errors: true
  tasks:

- hosts: mixed-nodes
  become: true
  vars:
    - pri_domain_name: 'test.vagrant.local'
  roles:
    - role: ansible-nginx
  tasks:

- hosts: all
  become: true
  vars:
    - enable_manage_ssh_keys: true  #defines if remote ssh keys should be managed
    - manage_ssh_keys:
        - remote_user: vagrant  #define username on remote system to add defined keys to
          state: present  #defines if ssh key should be added or removed (absent|present)
          keys:  #define key(s) to add to remote username
            - '/Users/larrysmithjr/.ssh/id_rsa.pub'
  roles:
    - role: ansible-manage-ssh-keys
```

{% endraw %}
So there you have it. A simple way to define your different Ansible
groups all the while keeping a standard Vagrantfile easily customizable
per Vagrant environment.

Enjoy!
