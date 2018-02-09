---
title: "Ansible - Generating Inventory From DNSMasq Leases"
date: "2018-02-09 16:17"
---

As I am working on a [little project](https://github.com/mrlesmithjr/ansible-rpi-k8s-cluster)
I wanted to have a way to update my `Ansible` inventory from DHCP leases
handed out by DNSMasq. Turns out it is very easy, therefore I wanted to share
this.

> NOTE: Assumption is that you have a functional DNSMasq server which is configured
> to allocate IP addresses correctly.

First thing we need to do is create a `Jinja2` template for our inventory:

> NOTE: I am using the template based on the little project that I mentioned
> above.

`hosts.inv.j2`:
{% raw %}

```yaml
[rpi_k8s:children]
rpi_k8s_master
rpi_k8s_slaves

[rpi_k8s_master]
rpi-k8s-1 ansible_host={{ jumphost_ip }}

[rpi_k8s_slaves]
{% for _host in _dnsmasq_dhcp_leases['stdout_lines']|sort %}
{{ _host.split(' ')[0] }} ansible_host={{ _host.split(' ')[1] }}
{% endfor %}

[missing]
{% for _host in range(rpi_nodes) %}
{%   if ("rpi-k8s-", loop.index)|join('') not in _dnsmasq_dhcp_leases['stdout'] %}
{%     if ("rpi-k8s-", loop.index)|join('') != "rpi-k8s-1" %}
rpi-k8s-{{ loop.index }}
{%     endif %}
{%   endif %}
{% endfor %}
```

{% endraw %}

Next we need to create our `Ansible` playbook:

`capture_dhcp_leases.yml`:
{% raw %}

```yaml
---
- hosts: rpi_k8s_master
  vars:
    jumphost_ip: 172.24.16.186
    rpi_nodes: 5
    rpi_update_inventory: true
  tasks:
    - name: Capturing DNSMasq DHCP Leases
      shell: cat /var/lib/misc/dnsmasq.leases | awk '{ print $4,$3,$2 }'
      register: _dnsmasq_dhcp_leases
      changed_when: false

    - name: Updating {{ inventory_dir }}
      template:
        src: hosts.inv.j2
        dest: "{{ inventory_dir }}/hosts.inv"
      delegate_to: localhost
      become: false
      when: rpi_update_inventory
```

{% endraw %}

Now we are ready to run this playbook:

```bash
ansible-playbook -i inventory/ capture_dhcp_leases.yml
```

And when we are done it will either update or not change anything in my inventory.

Example inventory is below:

```bash
[rpi_k8s:children]
rpi_k8s_master
rpi_k8s_slaves

[rpi_k8s_master]
rpi-k8s-1 ansible_host=172.16.24.186

[rpi_k8s_slaves]
rpi-k8s-2 ansible_host=192.168.100.128
rpi-k8s-3 ansible_host=192.168.100.129
rpi-k8s-4 ansible_host=192.168.100.130
rpi-k8s-5 ansible_host=192.168.100.131
```

Now based on the above examples the assumption is that we know a little something
about the results that we may get based on the project example. So maybe we need
to just reach out to the DNSMasq server and capture all of the leases and create
our inventory without knowing anything other than our DNSMasq server.

So let's modify our example above and do just that.

Let's first create an inventory with our DNSMasq server in it:
`hosts`:

```bash
dnsmasq ansible_host=172.16.24.186
```

Now let's adjust our `Ansible` template but let's also capture the mac addresses:

`hosts.inv.j2`:
{% raw %}

```yaml
[discovered_dhcp_leases]
{% for _host in _dnsmasq_dhcp_leases['stdout_lines']|sort %}
{{ _host.split(' ')[0] }} ansible_host={{ _host.split(' ')[1] }} mac_address={{ _host.split(' ')[2] }}
{% endfor %}
```

And also adjust our `Ansible` playbook:

`capture_dhcp_leases.yml`:
{% raw %}

```yaml
---
- hosts: dnsmasq
  vars:
  tasks:
    - name: Capturing DNSMasq DHCP Leases
      shell: cat /var/lib/misc/dnsmasq.leases | awk '{ print $4,$3,$2 }'
      register: _dnsmasq_dhcp_leases
      changed_when: false

    - name: Updating {{ inventory_dir }}
      template:
        src: hosts.inv.j2
        dest: "{{ inventory_dir }}/hosts.inv"
      delegate_to: localhost
      become: false
```

{% endraw %}

And now we are ready to run our `Ansible` playbook with these new changes:

```bash
ansible-playbook -i inventory/hosts capture_dhcp_leases.yml
```

And our `Ansible` inventory might look something like:

```bash
[discovered_dhcp_leases]
rpi-k8s-2 ansible_host=192.168.100.128 mac_address=b8:27:eb:e2:e9:cf
rpi-k8s-3 ansible_host=192.168.100.129 mac_address=b8:27:eb:b3:14:c1
rpi-k8s-4 ansible_host=192.168.100.130 mac_address=b8:27:eb:eb:cf:ba
rpi-k8s-5 ansible_host=192.168.100.131 mac_address=b8:27:eb:59:2c:47
```

So there you have it. A quick and easy way to generate your `Ansible` inventory
by capturing DHCP leases from a DNSMasq server.

Enjoy!
