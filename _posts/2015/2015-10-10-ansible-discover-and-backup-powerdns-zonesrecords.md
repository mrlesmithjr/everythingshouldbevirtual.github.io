---
  title: Ansible - Discover and Backup PowerDNS
  categories:
    - Automation
  tags:
    - Ansible
    - PowerDNS
---

While working on a solution that requires
[PowerDNS](https://www.powerdns.com/), I have come to a point in which I
would like to include backup and recovery options for this solution. So
I figured I would share these options and possibly help out others. I
will keep this post short as there really is not anything to it. Another
post will be forthcoming which will cover recovering from a disaster.
Again, we will be leveraging Ansible to provide all of this

There are actually two Ansible playbooks that are required to be ran but
I will wrap this up in a shell script for simplicity. I have created
these playbooks to run locally from where they are executed and leverage
the [PowerDNS API](https://doc.powerdns.com/md/httpapi/README/).

The first playbook connects to the PowerDNS API and discovers all zones
created on the server.
{% raw %}

```yaml
---
- hosts: localhost
  connection: local
  sudo: false
  vars:
    - pdns_api_key: changeme
    - pdns_api_web_url: 127.0.0.1
    - pdns_webserver_port: 8081
    - zones_dir: pdns_zone_backups
  tasks:
    - name: checking for zones_dir
      stat: path={{ zones_dir }}
      register: zones_dir_check

    - name: creating zones_dir if not exist
      file: path={{ zones_dir }} state=directory
      when: not zones_dir_check.stat.exists

    - name: gathering zones
      shell: "curl -H 'X-API-Key: {{ pdns_api_key }}' http://{{ pdns_api_web_url }}:{{ pdns_webserver_port }}/servers/localhost/zones | jq . > {{ zones_dir }}/pdns_zones_query.yml"

    - name: parsing zone names
      shell: "cat {{ zones_dir }}/pdns_zones_query.yml | grep name > {{ zones_dir }}/pdns_zones_query_names.yml"

    - name: cleaning up zone names
      replace: dest={{ zones_dir }}/pdns_zones_query_names.yml regexp="^    \"name\"{{ ':' }} \"" replace="  - "

    - name: cleaning up zones names
      replace: dest={{ zones_dir }}/pdns_zones_query_names.yml regexp="," replace=""

    - name: cleaning up zones names
      replace: dest={{ zones_dir }}/pdns_zones_query_names.yml regexp="\"" replace=""

    - name: making file yaml with vars
      lineinfile: dest={{ zones_dir }}/pdns_zones_query_names.yml line={{ item }} insertafter=BOF
      with_items:
        - "pdns_zones_query_names:"
        - "---"
```

{% endraw %}
The above will create a folder pdns_zone_backups with two files
initially (pdns_zones_query.yml and pdns_zones_query_names.yml).
Examples are below.

`pdns_zones_query.yml`

```json
[
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "=5Fmsdcs.vagrant.local.",
    "url": "/servers/localhost/zones/=5Fmsdcs.vagrant.local.",
    "name": "_msdcs.vagrant.local",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "=5Fsites.vagrant.local.",
    "url": "/servers/localhost/zones/=5Fsites.vagrant.local.",
    "name": "_sites.vagrant.local",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "=5Ftcp.vagrant.local.",
    "url": "/servers/localhost/zones/=5Ftcp.vagrant.local.",
    "name": "_tcp.vagrant.local",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "=5Fudp.vagrant.local.",
    "url": "/servers/localhost/zones/=5Fudp.vagrant.local.",
    "name": "_udp.vagrant.local",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "vagrant.local.",
    "url": "/servers/localhost/zones/vagrant.local.",
    "name": "vagrant.local",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "0.0.10.in-addr.arpa.",
    "url": "/servers/localhost/zones/0.0.10.in-addr.arpa.",
    "name": "0.0.10.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "2.0.10.in-addr.arpa.",
    "url": "/servers/localhost/zones/2.0.10.in-addr.arpa.",
    "name": "2.0.10.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "101.0.10.in-addr.arpa.",
    "url": "/servers/localhost/zones/101.0.10.in-addr.arpa.",
    "name": "101.0.10.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "106.0.10.in-addr.arpa.",
    "url": "/servers/localhost/zones/106.0.10.in-addr.arpa.",
    "name": "106.0.10.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "107.0.10.in-addr.arpa.",
    "url": "/servers/localhost/zones/107.0.10.in-addr.arpa.",
    "name": "107.0.10.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "110.0.10.in-addr.arpa.",
    "url": "/servers/localhost/zones/110.0.10.in-addr.arpa.",
    "name": "110.0.10.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "125.0.10.in-addr.arpa.",
    "url": "/servers/localhost/zones/125.0.10.in-addr.arpa.",
    "name": "125.0.10.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "10.10.10.in-addr.arpa.",
    "url": "/servers/localhost/zones/10.10.10.in-addr.arpa.",
    "name": "10.10.10.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "24.16.172.in-addr.arpa.",
    "url": "/servers/localhost/zones/24.16.172.in-addr.arpa.",
    "name": "24.16.172.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "1.168.192.in-addr.arpa.",
    "url": "/servers/localhost/zones/1.168.192.in-addr.arpa.",
    "name": "1.168.192.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "70.168.192.in-addr.arpa.",
    "url": "/servers/localhost/zones/70.168.192.in-addr.arpa.",
    "name": "70.168.192.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  },
  {
    "last_check": 0,
    "notified_serial": 0,
    "id": "200.168.192.in-addr.arpa.",
    "url": "/servers/localhost/zones/200.168.192.in-addr.arpa.",
    "name": "200.168.192.in-addr.arpa",
    "kind": "Native",
    "dnssec": false,
    "account": "",
    "masters": [],
    "serial": 2015101001
  }
]
```

From the above above information after running through Ansible we end up
with a usable list to use as a variable for our next Ansible playbook.
Example below.

`pdns_query_zones_names.yml`

```yaml
---
pdns_zones_query_names:
  - _msdcs.vagrant.local
  - _sites.vagrant.local
  - _tcp.vagrant.local
  - _udp.vagrant.local
  - vagrant.local
  - 0.0.10.in-addr.arpa
  - 2.0.10.in-addr.arpa
  - 101.0.10.in-addr.arpa
  - 106.0.10.in-addr.arpa
  - 107.0.10.in-addr.arpa
  - 110.0.10.in-addr.arpa
  - 125.0.10.in-addr.arpa
  - 10.10.10.in-addr.arpa
  - 24.16.172.in-addr.arpa
  - 1.168.192.in-addr.arpa
  - 70.168.192.in-addr.arpa
  - 200.168.192.in-addr.arpa
```

Now we are ready to use the above variable list for our next Ansible
playbook which will connect back to the PowerDNS API, query all records
for each zone and back them up. Our next playbook looks like below.
{% raw %}

```yaml
---
- hosts: localhost
  connection: local
  sudo: false
  vars:
    - pdns_api_key: changeme
    - pdns_api_web_url: 127.0.0.1
    - pdns_webserver_port: 8081
    - zones_dir: pdns_zone_backups
  vars_files:
    - "{{ zones_dir }}/pdns_zones_query_names.yml"
  tasks:
    - name: creating zone folders
      file: path={{ zones_dir }}/{{ item }} state=directory
      with_items: pdns_zones_query_names

    - name: pulling records for zones
      shell: "curl -H 'X-API-Key: {{ pdns_api_key }}' http://{{ pdns_api_web_url }}:{{ pdns_webserver_port }}/servers/localhost/zones/{{ item }} | jq . > {{ zones_dir }}/{{ item }}/{{ item }}.yml"
      with_items: pdns_zones_query_names
```

{% endraw %}
And once the above runs we will now have a folder structure that looks
like below, which includes all of our zones created as folders and then
all of our records created as a file for each zone.

```bash
|-- pdns_zone_backups
|   |-- 0.0.10.in-addr.arpa
|   |   `-- 0.0.10.in-addr.arpa.yml
|   |-- 10.10.10.in-addr.arpa
|   |   `-- 10.10.10.in-addr.arpa.yml
|   |-- 101.0.10.in-addr.arpa
|   |   `-- 101.0.10.in-addr.arpa.yml
|   |-- 106.0.10.in-addr.arpa
|   |   `-- 106.0.10.in-addr.arpa.yml
|   |-- 107.0.10.in-addr.arpa
|   |   `-- 107.0.10.in-addr.arpa.yml
|   |-- 110.0.10.in-addr.arpa
|   |   `-- 110.0.10.in-addr.arpa.yml
|   |-- 1.168.192.in-addr.arpa
|   |   `-- 1.168.192.in-addr.arpa.yml
|   |-- 125.0.10.in-addr.arpa
|   |   `-- 125.0.10.in-addr.arpa.yml
|   |-- 200.168.192.in-addr.arpa
|   |   `-- 200.168.192.in-addr.arpa.yml
|   |-- 2.0.10.in-addr.arpa
|   |   `-- 2.0.10.in-addr.arpa.yml
|   |-- 24.16.172.in-addr.arpa
|   |   `-- 24.16.172.in-addr.arpa.yml
|   |-- 70.168.192.in-addr.arpa
|   |   `-- 70.168.192.in-addr.arpa.yml
|   |-- _msdcs.vagrant.local
|   |   `-- _msdcs.vagrant.local.yml
|   |-- pdns_zones_query_names.yml
|   |-- pdns_zones_query.yml
|   |-- _sites.vagrant.local
|   |   `-- _sites.vagrant.local.yml
|   |-- _tcp.vagrant.local
|   |   `-- _tcp.vagrant.local.yml
|   |-- _udp.vagrant.local
|   |   `-- _udp.vagrant.local.yml
|   `-- vagrant.local
|       `-- vagrant.local.yml
```

And if we take a look at our `vagrant.local/vagrant.local.yml` file for
example.

```json
{
  "comments": [],
  "records": [
    {
      "content": "192.168.70.241",
      "disabled": false,
      "ttl": 3600,
      "type": "A",
      "name": "dns.vagrant.local"
    },
    {
      "content": "10.0.101.60",
      "disabled": false,
      "ttl": 3600,
      "type": "A",
      "name": "logstash.vagrant.local"
    },
    {
      "content": "dns.vagrant.local",
      "disabled": false,
      "ttl": 3600,
      "type": "CNAME",
      "name": "ntp1.vagrant.local"
    },
    {
      "content": "ns1.vagrant.local",
      "disabled": false,
      "ttl": 3600,
      "type": "NS",
      "name": "vagrant.local"
    },
    {
      "content": "ns2.vagrant.local",
      "disabled": false,
      "ttl": 3600,
      "type": "NS",
      "name": "vagrant.local"
    },
    {
      "content": "node-1.vagrant.local hostmaster.vagrant.local 2015101001 10800 3600 604800 3600",
      "disabled": false,
      "ttl": 3600,
      "type": "SOA",
      "name": "vagrant.local"
    },
    {
      "content": "10.0.101.40",
      "disabled": false,
      "ttl": 3600,
      "type": "A",
      "name": "vcsa.vagrant.local"
    }
  ],
  "soa_edit": "",
  "soa_edit_api": "",
  "last_check": 0,
  "notified_serial": 0,
  "id": "vagrant.local.",
  "url": "/servers/localhost/zones/vagrant.local.",
  "name": "vagrant.local",
  "kind": "Native",
  "dnssec": false,
  "account": "",
  "masters": [],
  "serial": 2015101001
}
```

As you can see from everything above it is actually quite simple to
accomplish our backups of zones/records from our PowerDNS server(s). Now
these tasks could be scheduled to run as a cron job, Jenkins job or any
other method. We can also incorporate these backups into our version
control utilizing Git. Again, all of these tasks we would want to
automate (more on this in a later post).

Looking for an Ansible playbook to install PowerDNS? Go
[here](https://galaxy.ansible.com/list#/roles/4687).

Up next will be how we can restore our zones/records from these backups.

Enjoy!
