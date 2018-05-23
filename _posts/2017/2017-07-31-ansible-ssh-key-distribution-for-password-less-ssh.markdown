---
title: "Ansible - SSH Key Distribution For Password-less SSH"
date: "2017-07-31 22:00"
---

## Ansible - SSH Key Distribution For Password-less SSH

When setting up massive scale environments you will likely run into this
scenario. How can I distribute a specific user account's SSH keys for all of my
hosts to allow password-less SSH logins between them? I have done this
previously by using the [following](https://github.com/mrlesmithjr/ansible-manage-ssh-keys)
[Ansible](https://www.ansible.com) role which means I have to `fetch` and keep
SSH keys in a GIT repo. But what if I want to do this more simplistic
and more dynamic and flexible? That is what I will be showing in this post. The
process is actually quite simple and works very well.

First I need to identify the account(s) in which I would like to distribute. So
for this example let's say you have `user1` and `user2` which have been added on
every host in your environment.

We will first define these as variables.

`group_vars/all/ssh_key_distribution.yml`:

```yaml
users_ssh_key_distribution:
  - 'user1'
  - 'user2'
```

Next we need to create a `jinja2` template to use as a `vars_files` definition
in our playbook which will contain our SSH Public Keys.

`ssh_keys_distribution.yml.j2`:

```raw
---
_ssh_keys_distribution:
{% for host in play_hosts %}
  - host: '{{ host }}'
    keys:
{%   for _keys in hostvars[host]['_ssh_pub_key']['results'] %}
      - user: '{{ _keys['item'] }}'
        key: '{{ _keys['stdout'] }}'
{%   endfor %}
{% endfor %}
```

[ssh_keys_distribution.yml.j2](https://gist.github.com/mrlesmithjr/afefbc00688e7038dac2bb9a131d1242)

> NOTE: If you were to look at the template generated file from the above `jinja2` template it would look similar to below:

```yaml
---
_ssh_keys_distribution:
  - host: 'node0'
    keys:
      - user: 'user1'
        key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDANbXQsS+VOhJa4p4vipJneaPXNmfcjU7Hwpgo1lj01Vmv+xKViVf3HLb15JRMS6ozuK0dmHQ9CLFVP8zC3tZe4b5CqIKR4l6HVI1n5MC++NnXQ2pUqaaJeS4Tr2ixsfYwVIYtL+eTrpH4L8Sihn6W7XrHmVelVHncMeI+hnvlWSyFhKybZ2csFG3mgEi+13+OWhgvN2cxVhg0PYMtsa6F+5ByRLarpJMF6Z5vZOlggu4G1cHTRdHAJm7Fx5G0oo2jqxGxuXlbYv8irp0IxZHJ3EhxOCzwKq9IpEot6zzl0szDe2HEXW6QRokyKydzMoCAL0LPq+FctJ+Ar0jhSrzP ansible-generated on node0'
      - user: 'user2'
        key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDXC+vM8Ezsuh742Y8PThAW8eX3+nzd9GW937vNrIjTIb+xPIlz/9Nc6LsZIH7H77BgMuUQSm48s1rfNKytgaj4dX0RLtBsya/92/OdhiGoPOE2vHfC23dF1W8n8ytTWmv/IcDIk9JA411PpmHQ0wZi/pcQelEl+TEYFqBQXtMWb2ambWB8DA6DXVHqnsSpbIIk1s47rCEzQ6VTWAjqqWt0iJ1Uip0kyVh/OCVOdkmklxVQiPsq1fcGbZ/4m3ajDZetCF+4wNt2CHXhJhXjKOkeBaFNWdJB7gjueF31aWhskwmZzSKb4jJ5wXxkjUMcHhYQo5bzM431X8/eyZl8WT5h ansible-generated on node0'
  - host: 'node1'
    keys:
      - user: 'user1'
        key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDmFdOfbJMMB2gsZcfTNjKGk+qEYRf67Z+9d8ncIaGe1kh1fJjkGj1dn/D80F46qDlMyAho6pamn6a5jOE4E2MpaC08TBIuZAmP3n3/9PGs7PpqHFiKuuJRRzIrmd7KKWBeZQxu/ikMvSoFAmoL+YXHX7emmgrkZaMUOHpd+F/NiHvVNOGmivAxksJ5EwL5xDbtopOHtWi4D8rKkYZLAF/whwjUXN25GmsdUXgRtOhmbfb/C7A2G59NAk3wfeWTBqI8dAh5IxKULNERmhc5SIVODRX82pfIl+xTm7jYUKevb0JPwzJkNPYFZv522D9dEJd2qel5srJB65XqAjGTutKX ansible-generated on node1'
      - user: 'user2'
        key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDzIrGP4L5LjSurrQb6zK14XQJ8pVEj/FYfVzH0mn8LyzWDCRr8Tfrd5Z32dZIcnhlnmDWbvUDM0UGa0gNtQ8Xjf4HaxoR3q3lxiv0R8IJxFeTKSyk3CzffsfOv4emFDpOzu1RCDftwd4Iv7dElCE5E3/eEI7IEbGkcveqjiCR8QsmXT0qytznqqxaDq//M3qtUuxjq4fSvdsB1EXDO1qhrZ5fR42XoiZLA4X3cj8A0JhR2s0KqXKhqnEqmHZERC87Ci+9g4P0nlZLjdv7ygJ+uppgDBvzHk7U6ljq2DWHQtCmJtlQMda4xJ5W3nZ9ofh9lvGyPg0k1xiDt6VAhl+9t ansible-generated on node1'
  - host: 'node2'
    keys:
      - user: 'user1'
        key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDC4gpaSykcdEKhUMAp5hXe0KBT5TJns1qlU7iYP5KpJtZJkOL0Z3SG+tMD7wfSHEyg+bub6YTPhj4VEN8F+HwXm27gWjtX7bir7cFDm+5e2TTYVx1koewylpH1GLVBVftOegyrAO0Y+kpGDWbvr5uM7hi8T59xE5Sr6FFcLP9tXhuoJjowQ00ipCHReyMp4psxQcd34eIAt8/XY1mBFG3cVdXb0iB8zGs0u0/Nw68MN9mNocmpmMpDN2OJWWSccKhZHzNlI9R8M5rYac+ZMje+GCjeGk/KO9aJZD0iKsza3pbi/U2+Ti/+HKPCBLLQ5YK1kTdvQqjLd6SaeCGnE9aR ansible-generated on node2'
      - user: 'user2'
        key: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQqKV0GLCh2vB6qKe4pewyTwenso81xilIpnd1nR0eG/w4XqY4P5xh2KH6nfGcwXrGLdjw5XRW6WE+df3VDDYNONxIavW64IO+P9Fn4OH856felM6Efv/4xI9uIXuSfcq1P2qqEytwpobmijf6zdifbj0k1t1f5g+EU/3nvqcr+yRukGKEZC7YLkyaf6j1Xcd1+FUkgAwqG1y4wuGxdXs9E3ZtPGwQJJxuPIa7cJ0XYMadkwoyEWiJOWxzPpODpX/fVZVysvbf/jW7LFxXi1AObyBOhJEKnoFolfjMSCD5aycO0AOtVcV/sQ09Ic0zqCPuT5YrKYs4iqyFhhdnLSHP ansible-generated on node2'
```

Now we need to create another `jinja2` template which we will use to distribute
all of our SSH host keys by hostname and IP address to our remote hosts `/etc/ssh/ssh_known_hosts`. This will ensure that we do not get prompted to
accept the SSH host key when logging in.

`ssh-hosts.j2`:

```raw
{% for item in host_keys_hostname['results'] %}
{%   for key in item['stdout_lines'] %}
{{ key }}
{%   endfor %}
{% endfor %}
{% for item in host_keys_ip['results'] %}
{%   for key in item['stdout_lines'] %}
{{ key }}
{%   endfor %}
{% endfor %}
```

[ssh-hosts.j2](https://gist.github.com/mrlesmithjr/10a0ad5ef831ca83e28f9b100e0f8ac6)

And finally we will create the playbook which will handle this distribution for us.

`ssh_key_distribution.yml`:

```raw
---
- hosts: all
  tasks:
    - name: Scan And Register SSH Host Keys (hostname)
      command: "ssh-keyscan {{ item }}"
      register: "host_keys_hostname"
      changed_when: false
      with_items: "{{ groups['all'] }}"
      delegate_to: localhost
      become: false

    - name: Scan And Register SSH Host Keys (IP)
      command: "ssh-keyscan {{ hostvars[item]['ansible_ssh_host'] }}"
      register: "host_keys_ip"
      changed_when: false
      with_items: "{{ groups['all'] }}"
      delegate_to: localhost
      become: false

    - name: Write SSH Host Keys
      template:
        src: "ssh-hosts.j2"
        dest: "/etc/ssh/ssh_known_hosts"
      become: true

    - name: Capturing SSH Keys
      command: "cat /home/{{ item }}/.ssh/id_rsa.pub"
      register: "_ssh_pub_key"
      become: true
      changed_when: false
      with_items: '{{ users_ssh_key_distribution }}'

    - name: Generating SSH Keys
      template:
        src: "ssh_keys_distribution.yml.j2"
        dest: "ssh_keys_distribution.yml"
      become: false
      run_once: true
      delegate_to: localhost

- hosts: all
  vars_files:
    - ./ssh_keys_distribution.yml
  tasks:
    - name: Adding SSH Keys
      authorized_key:
        user: "{{ item[1]['user'] }}"
        key: "{{ item[1]['key'] }}"
        state: "present"
      become: true
      with_subelements:
        - "{{ _ssh_keys_distribution }}"
        - keys
      when: inventory_hostname != item[0]['host']
```

Now that we have everything in place we are ready to run this playbook against
all of ours hosts in our inventory. And enjoy the benefits of not having to
run `ssh-copy-id` manually on each and every host in our environment.

```bash
ansible-playbook -i hosts ssh_keys_distribution.yml
```

And once the playbook completes you should be good to go.

And you can validate everything works as desired by connecting to one of your
hosts and test logging into other hosts using one of the defined user accounts.

```bash
vagrant@node0:~$ sudo su - user1
user1@node0:~$ ssh node2
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-66-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Development Environment
Last login: Wed Aug  2 03:19:50 2017 from 192.168.250.11
$ hostname
node2
$ whoami
user1
$ ssh 192.168.250.11
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-66-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Development Environment
Last login: Wed Aug  2 03:19:36 2017 from 192.168.250.10
$ hostname
node1
$
```

And there you have it, a nice and easy way to manage your SSH password-less
logins across your enivornment.

Enjoy!
