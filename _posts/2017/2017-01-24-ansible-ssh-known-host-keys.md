---
  title: Ansible - SSH Known Host Keys
---

I wanted to throw this together mainly for my own reference but maybe it
will help someone else as well. I had a need to add every host's ssh
keys to every host so that every host knew what every other hosts ssh
keys were. After a bit of attempting many different things below is what
I came up with. And it works.

First create a simple playbook:

{% raw %}

```yaml
---
- hosts: all
  become: true
  tasks:
    - name: scan and register
      command: "ssh-keyscan {{ item }}"
      register: "host_keys"
      changed_when: false
      with_items: '{{ groups.all }}'
      delegate_to: localhost
      become: false

    - name: write keys
      template:
        src: "ssh-hosts.j2"
        dest: "/etc/ssh/ssh_known_hosts"
```

{% endraw %}

Next create this simple template:

{% raw %}

```yaml
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

{% endraw %}

Then run

```bash
ansible-playbook -i yourinventoryfile ssh-keys.yml
```

and it will run through each host and capture their respective ssh key
and then create /etc/ssh/ssh_known_hosts on each host including all
other hosts ssh keys as well. Pretty simple after quite a bit of trial
and error but it does work.

Enjoy!
