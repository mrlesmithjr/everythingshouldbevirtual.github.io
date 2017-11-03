---
  title: Python - Manage PowerDNS Zones/Records
  categories:
    - Code
  tags:
    - PowerDNS
    - Python
---

While working on another PowerDNS project I have started working on a
Python script to manage Zones/Records. Being that there is not a ton of
info out there around this I wanted to begin sharing what I have put
together so far.

As far as provisioning PowerDNS I have spent a good bit of time over the
past year working on an Ansible role which will do a ton of the PowerDNS
deployment. You can also create Zones/Records using this role which is
basically just a template that creates a shell script and uses curl to
leverage the PowerDNS API. You can checkout this role
[here](https://github.com/mrlesmithjr/ansible-powerdns).

So as I mentioned this post is purely about using Python to manage
PowerDNS. And with that being said below is the latest version of the
script. Or you can head over to the GitHub repo
[here](https://github.com/mrlesmithjr/python-powerdns-management).

{% raw %}

```python
#!/usr/bin/env python
"""
pdns.py: Manage PowerDNS Zones/Records
"""

import argparse
import csv
import json
import requests

__author__ = "Larry Smith Jr."
__email___ = "mrlesmithjr@gmail.com"
__maintainer__ = "Larry Smith Jr."
__status__ = "Development"
# https://everythingshouldbevirtual.com
# @mrlesmithjr

EXAMPLES = """
Create a new Master Zone with info below:
-----------------------------------------

Zone: dev.vagrant.local
ZoneType: MASTER
Master: 172.28.128.3
Nameservers:
172.28.128.3
172.28.128.4
172.28.128.5

./pdns.py add_zones --apihost 172.28.128.3 --zone dev.vagrant.local --zoneType MASTER --nameservers 172.28.128.3,172.28.128.4,172.28.128.5


Create the Slave Zones with the info below:
-------------------------------------------

Zone: dev.vagrant.local
ZoneType: SLAVE
Master: 172.28.128.3
Slaves:
172.28.128.4
172.28.128.5

./pdns.py add_zones --apihost 172.28.128.4 --zone dev.vagrant.local --zoneType SLAVE --masters 172.28.128.3


Create Master Zones using a CSV file:
-------------------------------------

Create a master_zones.csv similar to below:

zone,zoneType,masters,nameservers
128.28.172.in-addr.arpa,MASTER,,"172.28.128.3,172.28.128.4,172.28.128.5"
dev.vagrant.local,MASTER,,"172.28.128.3,172.28.128.4,172.28.128.5"
prod.vagrant.local,MASTER,,"172.28.128.3,172.28.128.4,172.28.128.5"
test.vagrant.local,MASTER,,"172.28.128.3,172.28.128.4,172.28.128.5"
vagrant.local,MASTER,,"172.28.128.3,172.28.128.4,172.28.128.5"

The first row is the header...

Now read the csv file using CLI argument:

./pdns.py add_zones --apihost 172.28.128.3 --readcsv master_zones.csv

Create records with info below:
-------------------------------

Zone: dev.vagrant.local
name: test01.dev.vagrant.local
recordType: A
content: 172.28.128.161
name: development.dev.vagrant.local
recordType: CNAME
content: test01.dev.vagrant.local

./pdns.py add_records --zone dev.vagrant.local --name test01 --content 172.28.128.161 --recordType A
./pdns.py add_records --zone dev.vagrant.local --name development --content test01.dev.vagrant.local --recordType CNAME


Create records using a csv file:
---------------------------------------

Create a add_records.csv file similar to below:

name,zone,record_type,content,disabled,ttl,set_ptr,priority
development,test.vagrant.local,A,172.28.128.3,FALSE,3600,TRUE,0
node0,test.vagrant.local,A,172.28.128.4,FALSE,3600,TRUE,0
node1,test.vagrant.local,A,172.28.128.5,FALSE,3600,TRUE,0
node100,test.vagrant.local,A,172.28.128.100,FALSE,3600,TRUE,0
node101,test.vagrant.local,A,172.28.128.101,FALSE,3600,TRUE,0
node102,test.vagrant.local,A,172.28.128.102,FALSE,3600,TRUE,0
node2,dev.vagrant.local,A,172.28.128.201,FALSE,3600,TRUE,0
node201,dev.vagrant.local,A,172.28.128.202,FALSE,3600,TRUE,0
node202,dev.vagrant.local,A,172.28.128.203,FALSE,3600,TRUE,0
node203,dev.vagrant.local,CNAME,node201.dev.vagrant.local,FALSE,3600,TRUE,0
smtp,vagrant.local,A,172.28.128.20,FALSE,3600,TRUE,0
mail,vagrant.local,CNAME,smtp.vagrant.local,FALSE,3600,TRUE,0

The first row is the header...

Now read the csv file using CLI argument:

./pdns.py add_records --apihost 172.28.128.3 --readcsv add_records.csv


Delete records with info below:
-------------------------------

Zone: vagrant.local
name: smtp.vagrant.local
recordType: A

./pdns.py delete_records --apihost 172.28.128.3 --name smtp --zone vagrant.local --recordType A


Delete records reading from a csv file:
---------------------------------------

Create a delete_records.csv similar to below:

name,zone,record_type
node100,test.vagrant.local,A
node101,test.vagrant.local,A
node202,dev.vagrant.local,A
node203,dev.vagrant.local,CNAME

The first row is the header...

Now read the csv file using CLI argument:

./pdns.py delete_records --apihost 172.28.128.3 --readcsv delete_records.csv

Query PDNS config
-----------------

./pdns.py query_config --apihost 172.28.128.3


Query zones
-----------

./pdns.py query_zones --apihost 172.28.128.3

"""

class PDNSControl(object):
    """
    Main execution
    """
    def __init__(self):
        self.read_cli_args()
        self.setup_api_call()
        self.decide_action()

    def add_records(self):
        """
        Add new DNS records

        Create new DNS records of different types
        """
        if self.args.readcsv is None:
            payload = {
                "rrsets": [
                    {
                        "name": self.args.name + '.' + self.args.zone,
                        "type": self.args.recordType,
                        "changetype": "REPLACE",
                        "records": [
                            {
                                "content": self.args.content,
                                "disabled": self.args.disabled,
                                "name": self.args.name + '.' + self.args.zone,
                                "ttl": self.args.ttl,
                                "set-ptr": self.args.setPTR,
                                "type": self.args.recordType,
                                "priority": self.args.priority
                            }
                        ]
                    }
                ]
            }
            zone_check = requests.get(self.uri, headers=self.headers)
            if zone_check.status_code == 200:
                dummy_r = requests.patch(self.uri, data=json.dumps(payload), headers=self.headers)
                print ("DNS Record '%s' Successfully Added/Updated"
                       % (self.args.name + '.' + self.args.zone))
            else:
                print "DNS Zone '%s' Does Not Exist..." % self.args.zone
        elif self.args.readcsv is not None:
            try:
                f = open(self.args.readcsv)
                csv_f = csv.reader(f)
                next(csv_f, None) #skip headers
                for row in csv_f:
                    uri = ("http://%s:%s/servers/localhost/zones/%s"
                           %(self.args.apihost, self.args.apiport, row[1]))
                    if row[4].lower() == "false":
                        disabled = False
                    elif row[4].lower() == "true":
                        disabled = True
                    if row[6].lower() == "false":
                        set_ptr = False
                    if row[6].lower() == "true":
                        set_ptr = True
                    payload = {
                        "rrsets": [
                            {
                                "name": row[0] + '.' + row[1],
                                "type": row[2],
                                "changetype": "REPLACE",
                                "records": [
                                    {
                                        "content": row[3],
                                        "disabled": disabled,
                                        "name": row[0] + '.' + row[1],
                                        "ttl": row[5],
                                        "set-ptr": set_ptr,
                                        "type": row[2],
                                        "priority": row[7]
                                    }
                                ]
                            }
                        ]
                    }
                    zone_check = requests.get(uri, headers=self.headers)
                    if zone_check.status_code == 200:
                        dummy_r = (requests.patch(uri, data=json.dumps(payload),
                                                  headers=self.headers))
                        print ("DNS Record '%s' Successfully Added/Updated"
                               % (row[0] + '.' + row[1]))
                    else:
                        print "DNS Zone '%s' Does Not Exist...Skipping" % row[1]
            finally:
                f.close()

    def add_zones(self):
        """
        Add new DNS zones

        Create Master, Native or Slave zones
        """
        if self.args.readcsv is None:
            masters = []
            nameservers = []
            if self.args.masters:
                for master in self.args.masters.split(','):
                    masters.append(master)
            if self.args.nameservers:
                for nameserver in self.args.nameservers.split(','):
                    nameservers.append(nameserver)
            if self.args.zoneType == "MASTER":
                payload = {
                    "name": self.args.zone,
                    "kind": self.args.zoneType,
                    "masters": [],
                    "soa_edit_api": "INCEPTION-INCREMENT",
                    "nameservers": nameservers
                }
            elif self.args.zoneType == "NATIVE":
                payload = {
                    "name": self.args.zone,
                    "kind": self.args.zoneType,
                    "masters": [],
                    "nameservers": nameservers
                }
            else:
                payload = {
                    "name": self.args.zone,
                    "kind": self.args.zoneType,
                    "masters": masters,
                    "nameservers": []
                }
            zone_check_uri = ("http://%s:%s/servers/localhost/zones/%s"
                              %(self.args.apihost, self.args.apiport, self.args.zone))
            zone_check = requests.get(zone_check_uri, headers=self.headers)
            if zone_check.status_code == 200:
                print "DNS Zone '%s' Already Exists..." % self.args.zone
            else:
                dummy_r = requests.post(self.uri, data=json.dumps(payload), headers=self.headers)
                print "DNS Zone '%s' Successfully Added..." % self.args.zone
        elif self.args.readcsv is not None:
            try:
                f = open(self.args.readcsv)
                csv_f = csv.reader(f)
                next(csv_f, None) #skip headers
                for row in csv_f:
                    masters = []
                    nameservers = []
                    zone_check_uri = ("http://%s:%s/servers/localhost/zones/%s"
                                      %(self.args.apihost, self.args.apiport, row[0]))
                    zone_check = requests.get(zone_check_uri, headers=self.headers)
                    if zone_check.status_code == 200:
                        print "DNS Zone '%s' Already Exists...Skipping" % row[0]
                    else:
                        if row[2]:
                            for master in row[2].split(','):
                                masters.append(master)
                        if row[3]:
                            for nameserver in row[3].split(','):
                                nameservers.append(nameserver)
                        if row[1].upper() == "MASTER":
                            payload = {
                                "name": row[0],
                                "kind": row[1],
                                "masters": [],
                                "soa_edit_api": "INCEPTION-INCREMENT",
                                "nameservers": nameservers
                            }
                        elif row[1].upper() == "NATIVE":
                            payload = {
                                "name": row[0],
                                "kind": row[1],
                                "masters": [],
                                "nameservers": nameservers
                            }
                        else:
                            payload = {
                                "name": row[0],
                                "kind": row[1],
                                "masters": masters,
                                "nameservers": []
                            }
                        dummy_r = (requests.post(self.uri,
                                                 data=json.dumps(payload), headers=self.headers))
                        print "DNS Zone '%s' Successfully Added..." % row[0]
            finally:
                f.close()

    def decide_action(self):
        """
        Determine action

        Based on action passed determine which action to take
        """
        if self.args.action == "add_records":
            self.add_records()
        elif self.args.action == "add_zones":
            self.add_zones()
        elif self.args.action == "delete_records":
            self.delete_records()
        elif self.args.action == "delete_zones":
            self.delete_zones()
        elif self.args.action == "query_config":
            self.query_config()
        elif self.args.action == "query_stats":
            self.query_stats()
        elif self.args.action == "query_zones":
            self.query_zones()

    def delete_records(self):
        """
        Delete DNS records

        Delete DNS records of different types
        """
        if self.args.readcsv is None:
            payload = {
                "rrsets": [
                    {
                        "name": self.args.name + '.' + self.args.zone,
                        "type": self.args.recordType,
                        "changetype": "DELETE",
                    }
                ]
            }
            zone_check = requests.get(self.uri, headers=self.headers)
            if zone_check.status_code == 200:
                dummy_r = requests.patch(self.uri, data=json.dumps(payload), headers=self.headers)
                print ("DNS Record '%s' Successfully Deleted"
                       % (self.args.name + '.' + self.args.zone))
            else:
                print "DNS Zone '%s' Does Not Exist..." % self.args.zone
        elif self.args.readcsv is not None:
            try:
                f = open(self.args.readcsv)
                csv_f = csv.reader(f)
                next(csv_f, None) #skip headers
                for row in csv_f:
                    uri = ("http://%s:%s/servers/localhost/zones/%s"
                           %(self.args.apihost, self.args.apiport, row[1]))
                    payload = {
                        "rrsets": [
                            {
                                "name": row[0] + '.' + row[1],
                                "type": row[2],
                                "changetype": "DELETE",
                            }
                        ]
                    }
                    zone_check = requests.get(uri, headers=self.headers)
                    if zone_check.status_code == 200:
                        dummy_r = (requests.patch(uri, data=json.dumps(payload),
                                                  headers=self.headers))
                        print ("DNS Record '%s' Successfully Deleted"
                               % (row[0] + '.' + row[1]))
                    else:
                        print "DNS Zone '%s' Does Not Exist...Skipping" % row[1]
            finally:
                f.close()

    def delete_zones(self):
        """
        Delete DNS Zones
        """
        payload = {
            "name": self.args.zone
        }
        zone_check = requests.get(self.uri, headers=self.headers)
        if zone_check.status_code == 200:
            dummy_r = requests.delete(self.uri, data=json.dumps(payload), headers=self.headers)
            print "DNS Zone '%s' Successfully Deleted..." % self.args.zone
        else:
            print "DNS Zone '%s' Does Not Exist..." % self.args.zone

    def query_config(self):
        """
        Query PDNS Config
        """
        r = requests.get(self.uri, headers=self.headers)
        python_data = json.loads(r.text)
        print json.dumps(python_data, indent=4)

    def query_stats(self):
        """
        Query DNS Stats
        """
        r = requests.get(self.uri, headers=self.headers)
        python_data = json.loads(r.text)
        print json.dumps(python_data, indent=4)

    def query_zones(self):
        """
        Query DNS Zones

        Query existing DNS Zones
        """
        r = requests.get(self.uri, headers=self.headers)
        if r.status_code == 200:
            python_data = json.loads(r.text)
            print json.dumps(python_data, indent=4)
        else:
            print "DNS Zone '%s' Does Not Exist..." % self.args.zone

    def read_cli_args(self):
        """
        Read variables from CLI

        Read CLI variables passed on CLI
        """
        parser = argparse.ArgumentParser(description='PDNS Controls...')
        parser.add_argument('action', help='Define action to take',
                            choices=['add_records', 'add_zones', 'delete_records',
                                     'delete_zones', 'query_config', 'query_stats', 'query_zones'])
        parser.add_argument('--apikey', help='PDNS API Key', default='changeme')
        parser.add_argument('--apihost', help='PDNS API Host', default='127.0.0.1')
        parser.add_argument('--apiport', help='PDNS API Port', default='8081')
        parser.add_argument('--content', help='DNS Record content')
        parser.add_argument('--disabled', help='Define if Record is disabled',
                            choices=['True', 'False'], default=False)
        parser.add_argument('--masters', help='DNS zone masters')
        parser.add_argument('--name', help='DNS record name')
        parser.add_argument('--nameservers', help='DNS nameservers for zone')
        parser.add_argument('--priority', help='Define priority', default=0)
        parser.add_argument('--readcsv', help='Read input from CSV')
        parser.add_argument('--recordType', help='DNS record type',
                            choices=['A', 'AAAA', 'CNAME', 'MX', 'NS', 'PTR', 'SOA', 'SRV', 'TXT'])
        parser.add_argument('--setPTR', help='Define if PTR record is created',
                            choices=['True', 'False'], default=True)
        parser.add_argument('--ttl', help='Define TTL', default=3600)
        parser.add_argument('--zone', help='DNS zone')
        parser.add_argument('--zoneType', help='DNS Zone Type',
                            choices=['MASTER', 'NATIVE', 'SLAVE'])
        self.args = parser.parse_args()
        if self.args.action == "add_zones" and (self.args.zoneType == "MASTER" and
                                                self.args.nameservers is None):
            parser.error("--nameservers is required to create MASTER zone")
        if self.args.action == "add_zones" and (self.args.zoneType == "SLAVE" and
                                                self.args.masters is None):
            parser.error("--masters is required to create SLAVE zone")

    def setup_api_call(self):
        """
        Setup API Call

        Based on action setup the correct API call to make
        """
        self.headers = {'X-API-Key': self.args.apikey}
        if (self.args.action == "add_zones" or (self.args.action == "query_zones" and
                                                self.args.zone is None)):
            self.uri = ("http://%s:%s/servers/localhost/zones"
                        %(self.args.apihost, self.args.apiport))
        elif ((self.args.action == "add_records" and self.args.readcsv is None)
              or (self.args.action == "delete_records" and self.args.readcsv is None)
              or self.args.action == "delete_zones" or
              (self.args.action == "query_zones" and self.args.zone is not None)):
            self.uri = ("http://%s:%s/servers/localhost/zones/%s"
                        %(self.args.apihost, self.args.apiport, self.args.zone))
        elif self.args.action == "query_config":
            self.uri = ("http://%s:%s/servers/localhost/config"
                        %(self.args.apihost, self.args.apiport))
        elif self.args.action == "query_stats":
            self.uri = ("http://%s:%s/servers/localhost/statistics"
                        %(self.args.apihost, self.args.apiport))

if __name__ == '__main__':
    PDNSControl()
```

{% endraw %}

## Usage Examples

To get an idea of some of the cli parameters that you can use do a quick
show help...

```raw
./pdns.py --help

usage: pdns.py [-h] [--apikey APIKEY] [--apihost APIHOST] [--apiport APIPORT]
               [--content CONTENT] [--disabled {True,False}]
               [--masters MASTERS] [--name NAME] [--nameservers NAMESERVERS]
               [--priority PRIORITY]
               [--recordType {A,AAAA,CNAME,MX,NS,SOA,SRV}]
               [--setPTR {True,False}] [--ttl TTL] [--zone ZONE]
               [--zoneType {MASTER,NATIVE,SLAVE}]
               {add_records,add_zones,delete_zones,query_zones}

PDNS Controls...

positional arguments:
  {add_records,add_zones,delete_zones,query_zones}
                        Define action to take

optional arguments:
  -h, --help            show this help message and exit
  --apikey APIKEY       PDNS API Key
  --apihost APIHOST     PDNS API Host
  --apiport APIPORT     PDNS API Port
  --content CONTENT     DNS Record content
  --disabled {True,False}
                        Define if Record is disabled
  --masters MASTERS     DNS zone masters
  --name NAME           DNS record name
  --nameservers NAMESERVERS
                        DNS nameservers for zone
  --priority PRIORITY   Define priority
  --recordType {A,AAAA,CNAME,MX,NS,SOA,SRV}
                        DNS record type
  --setPTR {True,False}
                        Define if PTR record is created
  --ttl TTL             Define TTL
  --zone ZONE           DNS zone
  --zoneType {MASTER,NATIVE,SLAVE}
                        DNS Zone Type
```

Create a new Master Zone with info below:

```yaml
-   Zone: dev.vagrant.local
-   ZoneType: MASTER
-   Master: 172.28.128.3
-   Nameservers:
    -   172.28.128.3
    -   172.28.128.4
    -   172.28.128.5
```

But before we start let's do a quick query of the existing zones
defined on our master.

```bash
./pdns.py query_zones --apihost 172.28.128.3
```

Results...

```json
[
    {
        "kind": "Master",
        "name": "prod.vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/prod.vagrant.local.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062816,
        "masters": [],
        "serial": 2016062816,
        "id": "prod.vagrant.local."
    },
    {
        "kind": "Master",
        "name": "test.vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/test.vagrant.local.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062879,
        "masters": [],
        "serial": 2016062879,
        "id": "test.vagrant.local."
    },
    {
        "kind": "Master",
        "name": "vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/vagrant.local.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062806,
        "masters": [],
        "serial": 2016062806,
        "id": "vagrant.local."
    },
    {
        "kind": "Master",
        "name": "128.28.172.in-addr.arpa",
        "dnssec": false,
        "url": "/servers/localhost/zones/128.28.172.in-addr.arpa.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062842,
        "masters": [],
        "serial": 2016062842,
        "id": "128.28.172.in-addr.arpa."
    }
]
```

Now let's create our new master zone...

```bash
./pdns.py add_zones --apihost 172.28.128.3 --zone dev.vagrant.local --zoneType MASTER --nameservers 172.28.128.3,172.28.128.4,172.28.128.5
```

Results of the above ...

```json
{
    "kind": "Master",
    "name": "dev.vagrant.local",
    "dnssec": false,
    "url": "/servers/localhost/zones/dev.vagrant.local.",
    "account": "",
    "comments": [],
    "last_check": 0,
    "records": [
        {
            "disabled": false,
            "content": "172.28.128.3",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "172.28.128.4",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "172.28.128.5",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "node0.test.vagrant.local hostmaster.test.vagrant.local 2016062901 10800 3600 604800 3600",
            "type": "SOA",
            "name": "dev.vagrant.local",
            "ttl": 3600
        }
    ],
    "soa_edit": "",
    "notified_serial": 0,
    "masters": [],
    "soa_edit_api": "INCEPTION-INCREMENT",
    "serial": 2016062901,
    "id": "dev.vagrant.local."
}
```

Now let's create the slave zones on our PowerDNS slaves with the info
below:

```yaml
-   Zone: dev.vagrant.local
-   ZoneType: SLAVE
-   Master: 172.28.128.3
-   Slaves:
    -   172.28.128.4
    -   172.28.128.5
```

```bash
./pdns.py add_zones --apihost 172.28.128.4 --zone dev.vagrant.local --zoneType SLAVE --masters 172.28.128.3
```

And the results after running the above ....

```json
{
    "id": "dev.vagrant.local.",
    "url": "/servers/localhost/zones/dev.vagrant.local.",
    "name": "dev.vagrant.local",
    "kind": "Slave",
    "dnssec": false,
    "account": "",
    "masters": ["172.28.128.3"],
    "serial": 0,
    "notified_serial": 0,
    "last_check": 0,
    "soa_edit_api": "",
    "soa_edit": "",
    "records": [],
    "comments": []
}
```

And now repeat the above process on our 172.28.128.5 slave.

```bash
./pdns.py add_zones --apihost 172.28.128.5 --zone dev.vagrant.local --zoneType SLAVE --masters 172.28.128.3
```

And to validate that our new zones exist on our Master...

```bash
./pdns.py query_zones --apihost 172.28.128.3
```

Results show...

```json
[
    {
        "kind": "Master",
        "name": "prod.vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/prod.vagrant.local.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062816,
        "masters": [],
        "serial": 2016062816,
        "id": "prod.vagrant.local."
    },
    {
        "kind": "Master",
        "name": "test.vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/test.vagrant.local.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062879,
        "masters": [],
        "serial": 2016062879,
        "id": "test.vagrant.local."
    },
    {
        "kind": "Master",
        "name": "vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/vagrant.local.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062806,
        "masters": [],
        "serial": 2016062806,
        "id": "vagrant.local."
    },
    {
        "kind": "Master",
        "name": "128.28.172.in-addr.arpa",
        "dnssec": false,
        "url": "/servers/localhost/zones/128.28.172.in-addr.arpa.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062842,
        "masters": [],
        "serial": 2016062842,
        "id": "128.28.172.in-addr.arpa."
    },
    {
        "kind": "Master",
        "name": "dev.vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/dev.vagrant.local.",
        "account": "",
        "last_check": 0,
        "notified_serial": 2016062901,
        "masters": [],
        "serial": 2016062901,
        "id": "dev.vagrant.local."
    }
]
```

And as you can see the new master zone for dev.vagrant.local now exists.

And if we do a quick check on one of our slave nodes...

```bash
./pdns.py query_zones --apihost 172.28.128.4
```

Results...

```json
[
    {
        "kind": "Slave",
        "name": "prod.vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/prod.vagrant.local.",
        "account": "",
        "last_check": 1467154120,
        "notified_serial": 0,
        "masters": [
            "172.28.128.3"
        ],
        "serial": 2016062816,
        "id": "prod.vagrant.local."
    },
    {
        "kind": "Slave",
        "name": "test.vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/test.vagrant.local.",
        "account": "",
        "last_check": 1467152860,
        "notified_serial": 0,
        "masters": [
            "172.28.128.3"
        ],
        "serial": 2016062879,
        "id": "test.vagrant.local."
    },
    {
        "kind": "Slave",
        "name": "vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/vagrant.local.",
        "account": "",
        "last_check": 1467154240,
        "notified_serial": 0,
        "masters": [
            "172.28.128.3"
        ],
        "serial": 2016062806,
        "id": "vagrant.local."
    },
    {
        "kind": "Slave",
        "name": "128.28.172.in-addr.arpa",
        "dnssec": false,
        "url": "/servers/localhost/zones/128.28.172.in-addr.arpa.",
        "account": "",
        "last_check": 1467154060,
        "notified_serial": 0,
        "masters": [
            "172.28.128.3"
        ],
        "serial": 2016062842,
        "id": "128.28.172.in-addr.arpa."
    },
    {
        "kind": "Slave",
        "name": "dev.vagrant.local",
        "dnssec": false,
        "url": "/servers/localhost/zones/dev.vagrant.local.",
        "account": "",
        "last_check": 1467159949,
        "notified_serial": 0,
        "masters": [
            "172.28.128.3"
        ],
        "serial": 2016062901,
        "id": "dev.vagrant.local."
    }
]
```

Now that our zones are setup we are ready to create a few records. So
let's create some records using the following info:

```yaml
-   Zone: dev.vagrant.local
    -   name: test01.dev.vagrant.local
        -   recordType: A
        -   content: 172.28.128.161
    -   name: development.dev.vagrant.local
        -   recordType: CNAME
        -   content: test01.dev.vagrant.local
```

So let's create our A record.

```bash
./pdns.py add_records --zone dev.vagrant.local --name test01 --content 172.28.128.161 --recordType A
```

Results...

```json
{
    "kind": "Master",
    "name": "dev.vagrant.local",
    "dnssec": false,
    "url": "/servers/localhost/zones/dev.vagrant.local.",
    "account": "",
    "comments": [],
    "last_check": 0,
    "records": [
        {
            "disabled": false,
            "content": "172.28.128.3",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "172.28.128.4",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "172.28.128.5",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "node0.test.vagrant.local hostmaster.test.vagrant.local 2016062902 10800 3600 604800 3600",
            "type": "SOA",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "172.28.128.161",
            "type": "A",
            "name": "test01.dev.vagrant.local",
            "ttl": 3600
        }
    ],
    "soa_edit": "",
    "notified_serial": 2016062901,
    "masters": [],
    "soa_edit_api": "INCEPTION-INCREMENT",
    "serial": 2016062902,
    "id": "dev.vagrant.local."
}
```

And now we can create our CNAME record...

```bash
./pdns.py add_records --zone dev.vagrant.local --name development --content test01.dev.vagrant.local --recordType CNAME
```

Results...

```json
{
    "kind": "Master",
    "name": "dev.vagrant.local",
    "dnssec": false,
    "url": "/servers/localhost/zones/dev.vagrant.local.",
    "account": "",
    "comments": [],
    "last_check": 0,
    "records": [
        {
            "disabled": false,
            "content": "172.28.128.3",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "172.28.128.4",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "172.28.128.5",
            "type": "NS",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "node0.test.vagrant.local hostmaster.test.vagrant.local 2016062903 10800 3600 604800 3600",
            "type": "SOA",
            "name": "dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "test01.dev.vagrant.local",
            "type": "CNAME",
            "name": "development.dev.vagrant.local",
            "ttl": 3600
        },
        {
            "disabled": false,
            "content": "172.28.128.161",
            "type": "A",
            "name": "test01.dev.vagrant.local",
            "ttl": 3600
        }
    ],
    "soa_edit": "",
    "notified_serial": 2016062902,
    "masters": [],
    "soa_edit_api": "INCEPTION-INCREMENT",
    "serial": 2016062903,
    "id": "dev.vagrant.local."
}
```

So there you have it. So far this seems to be working quite nice. This
is obviously no where near complete so stay tuned on that. I will also
be following up another post soon on how to leverage my Ansible role
mentioned above to do these same tasks at a larger scale.

Enjoy!
