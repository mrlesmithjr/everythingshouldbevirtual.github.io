---
  title: Python - NMAP/SSH Fun
  categories:
    - Code
  tags:
    - Python
  redirect_from:
    - /python-nmapssh-fun
---

I am currently working on some Ansible/Python project(s) _(More to come
on that soon)_ and am working on integrating some functionality between
some of the moving components. One of them is scanning subnet(s) with
NMAP and testing SSH access _(May not be the most efficient, but time
will tell)_ along with spitting out some JSON results. So I figured I
would share the Python script that I am currently working on for this.

{% raw %}

```python
#!/usr/bin/env python
"""
Performs NMAP Scan(s) and tests SSH Connections (If desired)
"""

import argparse
import json
import socket
import nmap
import paramiko

__author__ = "Larry Smith Jr."
__email___ = "mrlesmithjr@gmail.com"
__maintainer__ = "Larry Smith Jr."
__status__ = "Development"
# https://everythingshouldbevirtual.com
# @mrlesmithjr

def main(args):
    """
    Main execution and setup of variables
    """
    common_ports = {'21', '22', '23', '25', '53', '69', '80', '109', '110',
                    '123', '137', '138', '139', '143', '156', '389', '443',
                    '445', '546', '547', '993', '995', '3306', '3389'}
    targethosts = args.hosts
    sshuser = args.sshuser
    sshpass = args.sshpass
    if (args.startport) and args.endport:
        start_port = str(args.startport)
        end_port = str(args.endport)
        targetports = start_port+'-'+end_port
    else:
        targetports = ', '.join(common_ports)
    if args.options:
        scan_options = str(args.options)
    else:
        scan_options = '-sV'
    scanned_hosts = {}
    scanned_hosts['hosts'] = {}
    ssh_hosts = []
    scan(scanned_hosts, ssh_hosts, targethosts, targetports, scan_options)
    if args.testssh == "true":
        ssh_validation(ssh_hosts, scanned_hosts, sshuser, sshpass)
    scan_results(scanned_hosts)
    if args.savetofile:
        save_results(scanned_hosts)

def read_cli_args():
    """
    Setup/Read command-line arguments
    """
    parser = argparse.ArgumentParser(description='Scan some MF\'n Hosts...')
    parser.add_argument('hosts', help='Define host(s) to scan [IP, FQDN, HostName or CIDR]')
    parser.add_argument('startport', nargs='?', help='Define starting port to scan')
    parser.add_argument('endport', nargs='?', help='Define ending port to scan')
    parser.add_argument('--options', help='Define additional NMAP options \
                        [ex. define as --options="-A"]')
    parser.add_argument('--savetofile', help='Define file to save results to')
    parser.add_argument('--sshpass', help='Define SSH Password (If --testssh)')
    parser.add_argument('--sshuser', help='Define SSH User (If --testssh)')
    parser.add_argument('--testssh', help='Defines if SSH connection should be tested',
                        choices=['true', 'false'])
    args_check = parser.parse_args()
    if args_check.testssh and args_check.sshuser is None and args_check.sshpass is None:
        parser.error("--testssh requires --sshuser and --sshpass.")
    return parser

def save_results(scanned_hosts):
    """
    Save scan results to file specified in JSON format
    """
    with open(args.savetofile, 'w') as outfile:
        json.dump(scanned_hosts, outfile, sort_keys=True, indent=4, ensure_ascii=False)

def scan(scanned_hosts, ssh_hosts, targethosts, targetports, scan_options):
    """
    Performs scan(s) and returns results
    """
    nm = nmap.PortScanner()
    nm.scan(targethosts, targetports, arguments=scan_options)
    nm.command_line()
    nm.scaninfo()
    for host in nm.all_hosts():
        ssh_hosts.append(host)
        try:
            resolve_host = socket.gethostbyaddr(host)
            resolved_address = repr(resolve_host[0])
            scanned_hosts['hosts'][host] = ({'host': host, 'hostname': resolved_address})
            for proto in nm[host].all_protocols():
                scanned_hosts['hosts'][host]['protocol'] = proto
            lport = nm[host][proto].keys()
            lport.sort()
            scan_port = []
            for port in lport:
                port_name = nm[host][proto][port]['name']
                port_product = nm[host][proto][port]['product']
                port_state = nm[host][proto][port]['state']
                scan_port.append({'port': port, 'state': port_state,
                                  'name': port_name, 'product': port_product})
            scanned_hosts['hosts'][host]['scanned_ports'] = scan_port
        except socket.herror:
            resolved_address = 'Lookup failed'
            scanned_hosts['hosts'][host] = ({'host': host, 'hostname': resolved_address})
            for proto in nm[host].all_protocols():
                scanned_hosts['hosts'][host]['protocol'] = proto
            lport = nm[host][proto].keys()
            lport.sort()
            scan_port = []
            for port in lport:
                port_name = nm[host][proto][port]['name']
                port_product = nm[host][proto][port]['product']
                port_state = nm[host][proto][port]['state']
                scan_port.append({'port': port, 'state': port_state,
                                  'name': port_name, 'product': port_product})
            scanned_hosts['hosts'][host]['scanned_ports'] = scan_port

def scan_results(scanned_hosts):
    """
    Display scan results in JSON format
    """
    print json.dumps(scanned_hosts, indent=4)

def ssh_validation(ssh_hosts, scanned_hosts, sshuser, sshpass):
    """
    Tests SSH connections using supplied username/password
    """
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    for ssh_host in ssh_hosts:
        try:
            ssh.connect(ssh_host, username=sshuser, password=sshpass)
            ssh_connection_status = "Successful"
        except paramiko.AuthenticationException:
            ssh_connection_status = "Authentication Failed"
        except paramiko.SSHException:
            ssh_connection_status = "Failed"
        except paramiko.ssh_exception.NoValidConnectionsError:
            ssh_connection_status = "Unable to Connect"
        scanned_hosts['hosts'][ssh_host]['ssh_connection_status'] = ssh_connection_status
        ssh.close()

if __name__ == "__main__":
    read_cli_args()
    args = read_cli_args().parse_args()
    main(args)
```

{% endraw %}
And if I were to run this against a subnet such as (Only scanning hosts
for an SSH port)...

```bash
nmap_fun.py 172.28.128.0/24 22 22 --sshuser vagrant --sshpass vagrant --testssh true
```

I would get the following results...Now I could easily use these
results elsewhere :)

```json
{
    "hosts": {
        "172.28.128.11": {
            "scanned_ports": [
                {
                    "product": "OpenSSH",
                    "state": "open",
                    "port": 22,
                    "name": "ssh"
                }
            ],
            "host": "172.28.128.11",
            "hostname": "Lookup failed",
            "protocol": "tcp",
            "ssh_connection_status": "Successful"
        },
        "172.28.128.10": {
            "scanned_ports": [
                {
                    "product": "OpenSSH",
                    "state": "open",
                    "port": 22,
                    "name": "ssh"
                }
            ],
            "host": "172.28.128.10",
            "hostname": "Lookup failed",
            "protocol": "tcp",
            "ssh_connection_status": "Successful"
        },
        "172.28.128.13": {
            "scanned_ports": [
                {
                    "product": "OpenSSH",
                    "state": "open",
                    "port": 22,
                    "name": "ssh"
                }
            ],
            "host": "172.28.128.13",
            "hostname": "Lookup failed",
            "protocol": "tcp",
            "ssh_connection_status": "Successful"
        },
        "172.28.128.12": {
            "scanned_ports": [
                {
                    "product": "OpenSSH",
                    "state": "open",
                    "port": 22,
                    "name": "ssh"
                }
            ],
            "host": "172.28.128.12",
            "hostname": "Lookup failed",
            "protocol": "tcp",
            "ssh_connection_status": "Successful"
        },
        "172.28.128.9": {
            "scanned_ports": [
                {
                    "product": "OpenSSH",
                    "state": "open",
                    "port": 22,
                    "name": "ssh"
                }
            ],
            "host": "172.28.128.9",
            "hostname": "Lookup failed",
            "protocol": "tcp",
            "ssh_connection_status": "Successful"
        },
        "172.28.128.1": {
            "scanned_ports": [
                {
                    "product": "",
                    "state": "closed",
                    "port": 22,
                    "name": "ssh"
                }
            ],
            "host": "172.28.128.1",
            "hostname": "Lookup failed",
            "protocol": "tcp",
            "ssh_connection_status": "Unable to Connect"
        }
    }
}
```

But maybe instead of checking for an SSH connection I would like to scan
a subnet for the common ports defined in the Python script variable
_common_ports_ and return those results in JSON.

```bash
nmap_fun.py 10.0.102.0/24
```

I would get some additional information in my JSON results..

```json
{
    "hosts": {
        "10.0.102.1": {
            "scanned_ports": [
                {
                    "product": "",
                    "state": "filtered",
                    "port": 21,
                    "name": "ftp"
                },
                {
                    "product": "OpenSSH",
                    "state": "open",
                    "port": 22,
                    "name": "ssh"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 23,
                    "name": "telnet"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 25,
                    "name": "smtp"
                },
                {
                    "product": "",
                    "state": "open",
                    "port": 53,
                    "name": "domain"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 69,
                    "name": "tftp"
                },
                {
                    "product": "nginx",
                    "state": "open",
                    "port": 80,
                    "name": "http"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 109,
                    "name": "pop2"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 110,
                    "name": "pop3"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 123,
                    "name": "ntp"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 137,
                    "name": "netbios-ns"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 138,
                    "name": "netbios-dgm"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 139,
                    "name": "netbios-ssn"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 143,
                    "name": "imap"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 156,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 389,
                    "name": "ldap"
                },
                {
                    "product": "nginx",
                    "state": "open",
                    "port": 443,
                    "name": "http"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 445,
                    "name": "microsoft-ds"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 546,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 547,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 993,
                    "name": "imaps"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 995,
                    "name": "pop3s"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 3306,
                    "name": "mysql"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 3389,
                    "name": "ms-wbt-server"
                }
            ],
            "host": "10.0.102.1",
            "hostname": "Lookup failed",
            "protocol": "tcp"
        },
        "10.0.102.129": {
            "scanned_ports": [
                {
                    "product": "",
                    "state": "closed",
                    "port": 21,
                    "name": "ftp"
                },
                {
                    "product": "OpenSSH",
                    "state": "open",
                    "port": 22,
                    "name": "ssh"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 23,
                    "name": "telnet"
                },
                {
                    "product": "Postfix smtpd",
                    "state": "open",
                    "port": 25,
                    "name": "smtp"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 53,
                    "name": "domain"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 69,
                    "name": "tftp"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 80,
                    "name": "http"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 109,
                    "name": "pop2"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 110,
                    "name": "pop3"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 123,
                    "name": "ntp"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 137,
                    "name": "netbios-ns"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 138,
                    "name": "netbios-dgm"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 139,
                    "name": "netbios-ssn"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 143,
                    "name": "imap"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 156,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 389,
                    "name": "ldap"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 443,
                    "name": "https"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 445,
                    "name": "microsoft-ds"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 546,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 547,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 993,
                    "name": "imaps"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 995,
                    "name": "pop3s"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 3306,
                    "name": "mysql"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 3389,
                    "name": "ms-wbt-server"
                }
            ],
            "host": "10.0.102.129",
            "hostname": "'ansible-control.etsbv.internal'",
            "protocol": "tcp"
        },
        "10.0.102.128": {
            "scanned_ports": [
                {
                    "product": "",
                    "state": "closed",
                    "port": 21,
                    "name": "ftp"
                },
                {
                    "product": "OpenSSH",
                    "state": "open",
                    "port": 22,
                    "name": "ssh"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 23,
                    "name": "telnet"
                },
                {
                    "product": "Postfix smtpd",
                    "state": "open",
                    "port": 25,
                    "name": "smtp"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 53,
                    "name": "domain"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 69,
                    "name": "tftp"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 80,
                    "name": "http"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 109,
                    "name": "pop2"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 110,
                    "name": "pop3"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 123,
                    "name": "ntp"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 137,
                    "name": "netbios-ns"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 138,
                    "name": "netbios-dgm"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 139,
                    "name": "netbios-ssn"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 143,
                    "name": "imap"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 156,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 389,
                    "name": "ldap"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 443,
                    "name": "https"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 445,
                    "name": "microsoft-ds"
                },
                {
                    "product": "",
                    "state": "filtered",
                    "port": 546,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 547,
                    "name": ""
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 993,
                    "name": "imaps"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 995,
                    "name": "pop3s"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 3306,
                    "name": "mysql"
                },
                {
                    "product": "",
                    "state": "closed",
                    "port": 3389,
                    "name": "ms-wbt-server"
                }
            ],
            "host": "10.0.102.128",
            "hostname": "'smtp.etsbv.internal'",
            "protocol": "tcp"
        }
    }
}
```

So there you have it...Stay tuned for more on this and more...

Enjoy!
