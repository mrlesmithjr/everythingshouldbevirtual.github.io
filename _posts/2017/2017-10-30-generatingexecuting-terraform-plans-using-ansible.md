---
title: Generating/Executing Terraform Plans Using Ansible
date: 2017-10-30
categories:
  - Automation
tags:
  - Ansible
  - Terraform
redirect_from:
  - /generatingexecuting-terraform-plans-using-ansible
---

Recently I have been working on a little project of my own based on
provisioning a [vSphere](https://www.vmware.com/products/vsphere.html)
environment using [Ansible](https://www.ansible.com) as the primary
automation tool. My intention of this project was to use
[IaC](https://en.wikipedia.org/wiki/Infrastructure_as_Code) as the main
driver.

> NOTE: You can view this project on GitHub.
> [ansible-vsphere-management](https://github.com/mrlesmithjr/ansible-vsphere-management).

As part of this project involved provisioning numerous Core Services VMs
to provide services such as:

- DNS
- DHCP
- IPAM
- Active Directory (Samba based AD)

The above Core Services are provisioned using Ansible to drive PowerCLI
scripts for the most part. I took this route because I found that the
vSphere Ansible modules were not flexible enough for my needs. However,
once the Core Services were deployed. I wanted to take an easier
approach to provisioning additional VMs and etc. This is where
[Terraform](https://www.terraform.io) comes into play. Terraform is a
great tool for defining your infrastructure as code, provisioning the
infrastructure and also provides the ability to tear it all down.
However, being that this project is based on IaC, I needed a way to
leverage variable definitions already present in the project to also
drive the Terraform provisioning. So how did I accomplish this? Quite
easily actually. I took the foundation of what Terraform files are
required to define the infrastructure that I wanted to provision and
used Ansible to generate those configuration files by using Jinja2
templates. By taking this approach it has proven to be a very dynamic
way to build out my infrastucture. With all of this being said, I will
be outlining with examples on how I was able to accomplish all of this.
Hopefully, this post will be beneficial to others as well.

## Terraform Variables

### Leveraging Existing group_vars To Define Terraform Variables

Here we will be defining our Terraform `variables.tf` file. Because all
of the variables are already defined throughout the project we can
leverage those and use a Jinja2 template to generate our `variables.tf`
file.

So for example, in our `group_vars/all/all.yml` Ansible variables we
have some of the following defined:

{% raw %}

```yaml
vsphere_dns_servers:
  - "{{ vsphere_dnsdist_vm_ips[0] }}"
  - "{{ vsphere_dnsdist_vm_ips[1] }}"

vsphere_dvswitches:
  - name: dvSwitch0
    active_uplinks:
      - dvUplink1
      - dvUplink2
      - dvUplink3
      - dvUplink4
    datacenter: "{{ vsphere_vcenter_datacenter['name'] }}"
    discovery_protocol:
      status: enabled
      type: LLDP
      operation: both
    hosts: "{{ groups['vsphere_hosts'] }}"
    load_balancing_policy: LoadBalanceSrcId
    mtu: 1500
    nics:
      - vmnic2
      - vmnic3
      - vmnic4
      - vmnic5
    port_groups:
      - name: VDS-VLAN-101
        num_ports: 128
        state: present
        type: earlyBinding
        vlan_id: 101
      - name: VDS-VLAN-102
        num_ports: 128
        state: present
        type: earlyBinding
        vlan_id: 102
      - name: VDS-VLAN-201
        num_ports: 128
        state: present
        type: earlyBinding
        vlan_id: 201
    standby_uplinks:
      []
      # - dvUplink3
      # - dvUplink4
    state: present
    unused_uplinks:
      []
      # - dvUplink4
    uplink_ports: 4

vsphere_linux_vapp_template_name: ubuntu-16.04-template

pdns_api_vip: "{{ vsphere_lb_vips[0] }}"

pdns_webserver_port: 8081

vsphere_pri_domain_name: lab.etsbv.internal
```

{% endraw %}

### Terraform VMs

We also have our `terraform_vms` defined as below which is what we will
be using to also generate our VM naming and etc. within our
`variables.tf` file. The `terraform_vms` definition will also be used
for [Terraform VM Resources](#terraform-vm-resources).

`group_vars/all/terraform_vms.yml`:

```yaml
---
# If defining addl_disk, size is in GB
terraform_vms:
  - group: docker_lbs
    count: 2
    inventory_parent_folder: Docker
    memory_mb: 512
    naming: docker-lb
    vcpu: 1
  - group: docker_storage
    addl_disk:
      - size: 100
    count: 2
    inventory_parent_folder: Docker
    memory_mb: 512
    naming: docker-storage
    vcpu: 1
  - group: docker_swarm_managers
    count: 3
    inventory_parent_folder: Docker
    memory_mb: 1024
    naming: docker-mgr
    vcpu: 1
  - group: docker_swarm_workers
    count: 4
    inventory_parent_folder: Docker
    memory_mb: 4096
    naming: docker-wrk
    vcpu: 1
```

Now let's break down the `terraform_vms` variable a bit to understand
what is going on here and what is being defined.

`group`: Defines the `Ansible` group as well as the Terraform
`vsphere_virtual_machine` resource name which we will touch on further
down when we get to Terraform VM Resources.

`count`: Defines the number of VMs which should be generated for each
group.

`inventory_parent_folder`: Defines the `vCenter` folder where the VMs
will be located when provisioned.

`memory_mb`: Defines the amount of memory in MB to allocate each VM
within the group.

`naming`: Defines the VM naming scheme to use when generating the VMs.

`vcpu`: Defines the number of CPUs to allocate to each VM within the
group.

### Terraform Variables Jinja2 Template

The `terraform_variables.tf.j2` template below is what we will use for
generating our `variables.tf` file based on the variables defined in
[Leveraging Existing group_vars To Define Terraform
Variables](#leveraging-existing-group_vars-to-define-terraform-variables):

{% raw %}

```raw
variable "dns_servers" {
  default = [
{% for dns_server in vsphere_dns_servers %}
    "{{ dns_server }}",
{% endfor %}
  ]
}
variable "dvSwitches" {
  default = [
    "{{ vsphere_dvswitches[0]['name'] }}",
  ]
}
variable "esxi_hosts" {
  default = [
  {% for host in groups['vsphere_hosts'] %}
    "{{ host }}",
  {% endfor %}
  ]
}
variable "linux_vm_template" {
  default = "{{ vsphere_linux_vapp_template_name }}"
}
variable "pdns_api_key" {}
variable "pdns_server_url" {
default = "http://{{ pdns_api_vip }}:{{ pdns_webserver_port }}/api/v1"
}
variable "pri_domain_name" {
  default = "{{ vsphere_pri_domain_name }}"
}
{% for vms in terraform_vms %}
variable "vms_{{ vms['group'] }}" {
  default = [
  {%   for vm in range(vms['count']) %}
  {%     if loop.index < 10 %}
    "{{ vms['naming'] }}-0{{ loop.index }}.{{ vsphere_pri_domain_name }}",
  {%     elif loop.index >= 10 %}
    "{{ vms['naming'] }}-{{ loop.index }}.{{ vsphere_pri_domain_name }}",
  {%     endif %}
  {%   endfor %}
  ]
}
{% endfor %}
variable "vsphere_datacenter" {
  default = "{{ vsphere_vcenter_datacenter['name'] }}"
}
variable "vsphere_datacenter_cluster" {
  default = "{{ vsphere_vcenter_cluster['name'] }}"
}
variable "vsphere_default_vm_datastore" {
  default = "{{ vsphere_vm_services_datastore }}"
}
variable "vsphere_default_vm_network" {
  default = "{{ vsphere_vm_services_vswitch }}"
}
variable "vsphere_host_license_key" {}
variable "vsphere_networks" {
  default = [
  {% for switch in vsphere_dvswitches %}
  {%   for net in switch['port_groups'] %}
    "{{ net['name'] }}",
  {%   endfor %}
  {% endfor %}
  ]
}
variable "vsphere_password" {}
variable "vsphere_server" {
  default = "{{ vsphere_vcsa_network_ip }}"
}
variable "vsphere_username" {}
variable "vsphere_vcenter_license_key" {}
```

{% endraw %}

### Generated `variables.tf`

And if we were to generate our `variables.tf` based on the above
variables and Jinja2 template it would look similar to below:

{% raw %}

```raw
variable "dns_servers" {
  default = [
    "10.0.102.40",
    "10.0.102.41",
  ]
}
variable "dvSwitches" {
  default = [
    "dvSwitch0",
  ]
}
variable "esxi_hosts" {
  default = [
    "esxi-01.lab.etsbv.internal",
    "esxi-02.lab.etsbv.internal",
  ]
}
variable "linux_vm_template" {
  default = "ubuntu-16.04-template"
}
variable "pdns_api_key" {}
variable "pdns_server_url" {
  default = "http://10.0.102.100:8081/api/v1"
}
variable "pri_domain_name" {
  default = "lab.etsbv.internal"
}
variable "vms_docker_lbs" {
  default = [
    "docker-lb-01.lab.etsbv.internal",
    "docker-lb-02.lab.etsbv.internal",
  ]
}
variable "vms_docker_storage" {
  default = [
    "docker-storage-01.lab.etsbv.internal",
    "docker-storage-02.lab.etsbv.internal",
  ]
}
variable "vms_docker_swarm_managers" {
  default = [
    "docker-mgr-01.lab.etsbv.internal",
    "docker-mgr-02.lab.etsbv.internal",
    "docker-mgr-03.lab.etsbv.internal",
  ]
}
variable "vms_docker_swarm_workers" {
  default = [
    "docker-wrk-01.lab.etsbv.internal",
    "docker-wrk-02.lab.etsbv.internal",
    "docker-wrk-03.lab.etsbv.internal",
    "docker-wrk-04.lab.etsbv.internal",
  ]
}
variable "vsphere_datacenter" {
  default = "LAB"
}
variable "vsphere_datacenter_cluster" {
  default = "LAB-Cluster"
}
variable "vsphere_default_vm_datastore" {
  default = "vSphere_iSCSI_Datastore_1"
}
variable "vsphere_default_vm_network" {
  default = "VSS-VLAN-102"
}
variable "vsphere_host_license_key" {}
variable "vsphere_networks" {
  default = [
    "VDS-VLAN-101",
    "VDS-VLAN-102",
    "VDS-VLAN-201",
  ]
}
variable "vsphere_password" {}
variable "vsphere_server" {
  default = "10.0.102.60"
}
variable "vsphere_username" {}
variable "vsphere_vcenter_license_key" {}
```

{% endraw %}

So as you can see we easily leveraged existing Ansible defined variables
along with our Jinja2 template to define our Terraform `variables.tf`
file. And remember why this is important and useful. We now have a
streamlined methodology to not only provision out our infrastructure
using Ansible but also to define our Terraform provisioning
consistently.

### Terraform Secret Variables

We now also need to maintain some secret variables which we would never
want to keep in our version control system. And again we can use a
Jinja2 template to generate out these super secret variables as well.
For example, the Jinja2 template below could be used to generate our
`terraform.tfvars` config:

`terraform.tfvars.j2`:

{% raw %}

```raw
pdns_api_key = "{{ pdns_webserver_password }}"
vsphere_host_license_key = ""
vsphere_password = "{{ vsphere_vcsa_sso_user_info['password'] }}"
vsphere_username = "{{ vsphere_vcsa_sso_user_info['username'] }}"
vsphere_vcenter_license_key = ""
```

{% endraw %}

Which would produce the following `terraform.tfvars`:

```raw
pdns_api_key = "changeme"
vsphere_host_license_key = ""
vsphere_password = "VMw@re1!"
vsphere_username = "Administrator@vsphere.local"
vsphere_vcenter_license_key = ""
```

## Terraform Data Sources

We now will use a standard `data_sources.tf` file which will not be
generated using Ansible. However, it will be configured to leverage our
`variables.tf` file which we already generated. Below is an example of
what our `data_sources.tf` might look like.

{% raw %}

```raw
data "vsphere_datacenter" "dc" {
  name = "${var.vsphere_datacenter}"
}

data "vsphere_host" "esxi_hosts" {
  count = "${length(var.esxi_hosts)}"
  name = "${var.esxi_hosts[count.index]}"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

data "vsphere_distributed_virtual_switch" "dvs" {
  count = "${length(var.dvSwitches)}"
  name = "${var.dvSwitches[count.index]}"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

data "vsphere_network" "net" {
  count = "${length(var.vsphere_networks)}"
  name = "${var.vsphere_networks[count.index]}"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
```

{% endraw %}

As you can see from above, we are leveraging pre defined variables.

## Terraform Providers

We are only focusing on the `vsphere` provider as part of this example
so we actually will use a static `vsphere_profider.tf` config which
would look something like:

{% raw %}

```raw
provider "vsphere" {
  user           = "${var.vsphere_username}"
  password       = "${var.vsphere_password}"
  vsphere_server = "${var.vsphere_server}"

  # if you have a self-signed cert
  allow_unverified_ssl = true
}
```

{% endraw %}

## Terraform Inventory Resources

For our Terraform inventory resources we will be using yet another
Jinja2 template that will be generated for us. This file will define the
`vCenter` folder(s) for our VMs to be defined within. If you remember in
the [Terraform VMs](#terraform-vms) section we discussed what the
`inventory_parent_folder` was defined for. Well here is the Jinja2
template that will define our inventory resources.

{% raw %}

```raw
resource "vsphere_folder" "terraform_deployed" {
  path = "Terraform Deployed"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

{% set _folders = [] %}
{% for folder in terraform_vms %}
{%   if folder['inventory_parent_folder'] is defined %}
{%     set _folders = _folders.append(folder['inventory_parent_folder']) %}
{%   endif %}
{% endfor %}
{% set folders = _folders|unique|list %}
{% for folder in folders %}
resource "vsphere_folder" "{{ folder }}" {
  path = "${vsphere_folder.terraform_deployed.path}/{{ folder }}"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

{% endfor %}
{% for folder in terraform_vms %}
{%   if folder['inventory_parent_folder'] is not defined %}
resource "vsphere_folder" "{{ folder['group'] }}" {
  path = "${vsphere_folder.terraform_deployed.path}/{{ folder['group'] }}"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
{%   endif %}

{% endfor %}
{% for folder in terraform_vms %}
{%   if folder['inventory_parent_folder'] is defined %}
resource "vsphere_folder" "{{ folder['group'] }}" {
  path = "${vsphere_folder.{{ folder['inventory_parent_folder'] }}.path}/{{ folder['group'] }}"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
{%   endif %}
{% endfor %}
```

{% endraw %}

What we are doing above is first iterating all of our top level parent
folders and ensuring that we do not have any duplicates, and then
iterating through each child folder to define our complete inventory
resources structure. And after this file is generated it would look like
below:

{% raw %}

```raw
resource "vsphere_folder" "terraform_deployed" {
  path = "Terraform Deployed"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

resource "vsphere_folder" "Docker" {
  path = "${vsphere_folder.terraform_deployed.path}/Docker"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
resource "vsphere_folder" "docker_lbs" {
  path = "${vsphere_folder.Docker.path}/docker_lbs"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
resource "vsphere_folder" "docker_storage" {
  path = "${vsphere_folder.Docker.path}/docker_storage"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
resource "vsphere_folder" "docker_swarm_managers" {
  path = "${vsphere_folder.Docker.path}/docker_swarm_managers"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
resource "vsphere_folder" "docker_swarm_workers" {
  path = "${vsphere_folder.Docker.path}/docker_swarm_workers"
  type = "vm"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
```

{% endraw %}

## Terraform VM Resources

Now for our VM resources. We already touched on how the variables are
defined for `terraform_vms` above in [Terraform VMs](#terraform-vms). So
now let's see what this would like here now. Because those variables
are already defined we can use another Jinja2 template to generate our
`vm_resources.tf` config. The following is an example of what that
Jinja2 template might look like:

`terraform_vm_resources.tf.j2`:

{% raw %}

```raw
{% for vms in terraform_vms %}
resource "vsphere_virtual_machine" "{{ vms['group'] }}" {
  count = "${length(var.vms_{{ vms['group'] }})}"
  name = "${var.vms_{{ vms['group'] }}[count.index]}"
  datacenter = "${data.vsphere_datacenter.dc.name}"
  disk {
    datastore = "${var.vsphere_default_vm_datastore}"
    template = "${var.linux_vm_template}"
    type = "thin"
  }
{%   if vms['addl_disk'] is defined %}
{%     for addl_disk in vms['addl_disk'] %}
  disk {
      datastore = "${var.vsphere_default_vm_datastore}"
      size = "{{ addl_disk['size'] }}"
      name = "${var.vms_{{ vms['group'] }}[count.index]}_{{ loop.index }}"
      type = "thin"
    }
{%     endfor %}
{%   endif %}
  dns_servers = "${var.dns_servers}"
  domain = "${var.pri_domain_name}"
  folder = "${vsphere_folder.{{ vms['group'] }}.path}"
  memory = {{ vms['memory_mb'] }}
  network_interface {
    label = "${var.vsphere_default_vm_network}"
  }
  vcpu = {{ vms['vcpu'] }}
}
{% endfor %}
```

{% endraw %}

And if we were to run this through Ansible to generate our
`vm_resources.tf` config it would look similar to below:

{% raw %}

```raw
resource "vsphere_virtual_machine" "docker_lbs" {
  count = "${length(var.vms_docker_lbs)}"
  name = "${var.vms_docker_lbs[count.index]}"
  datacenter = "${data.vsphere_datacenter.dc.name}"
  disk {
    datastore = "${var.vsphere_default_vm_datastore}"
    template = "${var.linux_vm_template}"
    type = "thin"
  }
  dns_servers = "${var.dns_servers}"
  domain = "${var.pri_domain_name}"
  folder = "${vsphere_folder.docker_lbs.path}"
  memory = 512
  network_interface {
    label = "${var.vsphere_default_vm_network}"
  }
  vcpu = 1
}
resource "vsphere_virtual_machine" "docker_storage" {
  count = "${length(var.vms_docker_storage)}"
  name = "${var.vms_docker_storage[count.index]}"
  datacenter = "${data.vsphere_datacenter.dc.name}"
  disk {
    datastore = "${var.vsphere_default_vm_datastore}"
    template = "${var.linux_vm_template}"
    type = "thin"
  }
  disk {
    datastore = "${var.vsphere_default_vm_datastore}"
    size = "100"
    name = "${var.vms_docker_storage[count.index]}_1"
    type = "thin"
  }
  dns_servers = "${var.dns_servers}"
  domain = "${var.pri_domain_name}"
  folder = "${vsphere_folder.docker_storage.path}"
  memory = 512
  network_interface {
    label = "${var.vsphere_default_vm_network}"
  }
  vcpu = 1
}
resource "vsphere_virtual_machine" "docker_swarm_managers" {
  count = "${length(var.vms_docker_swarm_managers)}"
  name = "${var.vms_docker_swarm_managers[count.index]}"
  datacenter = "${data.vsphere_datacenter.dc.name}"
  disk {
    datastore = "${var.vsphere_default_vm_datastore}"
    template = "${var.linux_vm_template}"
    type = "thin"
  }
  dns_servers = "${var.dns_servers}"
  domain = "${var.pri_domain_name}"
  folder = "${vsphere_folder.docker_swarm_managers.path}"
  memory = 1024
  network_interface {
    label = "${var.vsphere_default_vm_network}"
  }
  vcpu = 1
}
resource "vsphere_virtual_machine" "docker_swarm_workers" {
  count = "${length(var.vms_docker_swarm_workers)}"
  name = "${var.vms_docker_swarm_workers[count.index]}"
  datacenter = "${data.vsphere_datacenter.dc.name}"
  disk {
    datastore = "${var.vsphere_default_vm_datastore}"
    template = "${var.linux_vm_template}"
    type = "thin"
  }
  dns_servers = "${var.dns_servers}"
  domain = "${var.pri_domain_name}"
  folder = "${vsphere_folder.docker_swarm_workers.path}"
  memory = 4096
  network_interface {
    label = "${var.vsphere_default_vm_network}"
  }
  vcpu = 1
}
```

{% endraw %}

And once again we can see how easy this has been to maintain consistency
across our project.

## Terraform Ansible Playbook

Below is an example of the Ansible playbook which we will use to put
everything together in order to generate all of our Terraform
configuration files and then also plan and deploy.

{% raw %}

```yaml
---
- hosts: localhost
  connection: local
  become: false
  gather_facts: false
  vars:
    terraform_apply: false
    terraform_destroy: false
    terraform_dir: ../terraform
  tasks:
    - name: Capturing Terraform Command
      command: which terraform
      register: _terraform_command
      changed_when: false
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_init
        - terraform_inventory
        - terraform_plan

    - name: Setting Terraform Command Path
      set_fact:
        terraform_command: "{{ _terraform_command['stdout'] }}"
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_init
        - terraform_inventory
        - terraform_plan

    - name: Generating Terraform Variables
      template:
        src: templates/terraform_variables.tf.j2
        dest: "{{ terraform_dir }}/variables.tf"
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_init
        - terraform_inventory
        - terraform_plan

    - name: Generating Secret Terraform Variables
      template:
        src: templates/terraform.tfvars.j2
        dest: "{{ terraform_dir }}/terraform.tfvars"
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_init
        - terraform_inventory
        - terraform_plan

    - name: Generating Terraform Inventory Resources
      template:
        src: templates/terraform_inventory_resources.tf.j2
        dest: "{{ terraform_dir }}/inventory_resources.tf"
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_init
        - terraform_inventory
        - terraform_plan

    - name: Generating Terraform VM Definitions
      template:
        src: templates/terraform_vm_resources.tf.j2
        dest: "{{ terraform_dir }}/vm_resources.tf"
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_init
        - terraform_inventory
        - terraform_plan

    - name: Generating Terraform PDNS Resources
      template:
        src: templates/terraform_pdns_resources.tf.j2
        dest: "{{ terraform_dir }}/pdns_resources.tf"
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_init
        - terraform_inventory
        - terraform_plan

    - name: Capturing Terraform Init State
      command: "{{ terraform_command }} init -get=true -input=false"
      register: _terraform_init_state
      changed_when: false
      args:
        chdir: "{{ terraform_dir }}"
      tags:
        - terraform_init

    - name: Running Terraform Plan
      command: "{{ terraform_command }} plan -out=tfplan -input=false -detailed-exitcode"
      register: _terraform_plan
      changed_when: false
      args:
        chdir: "{{ terraform_dir }}"
      tags:
        - terraform_apply
        - terraform_plan
      failed_when: _terraform_plan['rc'] > 2

    - name: Displaying Terraform Plan
      debug: var=_terraform_plan
      tags:
        - terraform_apply
        - terraform_plan

    - name: Applying Terraform Plan
      command: "{{ terraform_command }} apply -input=false -auto-approve=true tfplan"
      register: _terraform_apply
      changed_when: false
      args:
        chdir: "{{ terraform_dir }}"
      tags:
        - terraform_apply
      when: >
        _terraform_plan['rc'] == 2 and
        terraform_apply

    - name: Destroying Terraform Plan
      command: "{{ terraform_command }} destroy -force"
      register: _terraform_destroy
      changed_when: false
      args:
        chdir: "{{ terraform_dir }}"
      tags:
        - terraform_destroy
      when: terraform_destroy

    - name: Capturing Terraform State
      command: "{{ terraform_command }} state pull"
      register: _terraform_state
      changed_when: false
      args:
        chdir: "{{ terraform_dir }}"
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_inventory

    - name: Generating Terraform Ansible Inventory
      template:
        src: terraform.inv.j2
        dest: "{{ vsphere_inventory_directory }}/terraform.inv"
      tags:
        - terraform_apply
        - terraform_destroy
        - terraform_inventory
```

{% endraw %}

## Terraform Ansible Inventory

And of course none of this would be beneficial to any post Terraform
deployment in order to provision our VMs with some additional Ansible
playbooks without an inventory. In order to dynamically generate this
during our Terraform playbook example above we have to capture the
Terraform State at the very end of our playbook.

> NOTE: Capturing the Terraform State actually just displays what is in
> the `terraform.tfstate` file in our project folder. So you can also
> get creative with this a bit more as well.

If you look at the tasks below which are from the playbook example above
you will see how we are accomplishing this.

{% raw %}

```yaml
- name: Capturing Terraform State
  command: "{{ terraform_command }} state pull"
  register: _terraform_state
  changed_when: false
  args:
    chdir: "{{ terraform_dir }}"
  tags:
    - terraform_apply
    - terraform_destroy
    - terraform_inventory

- name: Generating Terraform Ansible Inventory
  template:
    src: terraform.inv.j2
    dest: "{{ vsphere_inventory_directory }}/terraform.inv"
  tags:
    - terraform_apply
    - terraform_destroy
    - terraform_inventory
```

{% endraw %}

And the `terraform.inv.j2` Jinja2 template might look something like:
{% raw %}

```yaml
{% set _groups = [] %}
{% for key, value in (_terraform_state['stdout']|from_json)['modules'][0]['resources'].items() %}
{%   if 'vsphere_virtual_machine' in key %}
{%     set _group = key.split('.') %}
{%     set _groups = _groups.append(_group[1]) %}
{%   endif %}
{% endfor %}
{% set groups = _groups|unique|list %}
{% for group in groups %}
[{{ group }}]
{%   for key, value in (_terraform_state['stdout']|from_json)['modules'][0]['resources'].items() %}
{%     if 'vsphere_virtual_machine' in key %}
{%       set _group = key.split('.') %}
{%       if _group[1] == group %}
{{ value['primary']['attributes']['name'] }}
{%       endif %}
{%     endif %}
{%   endfor %}

{% endfor %}
[terraform_vms]
{% for key, value in (_terraform_state['stdout']|from_json)['modules'][0]['resources'].items() %}
{%   if 'vsphere_virtual_machine' in key %}
{{ value['primary']['attributes']['name'] }} ansible_host={{ value['primary']['attributes']['network_interface.0.ipv4_address'] }} mac_address={{ value['primary']['attributes']['network_interface.0.mac_address'] }} uuid={{ value['primary']['attributes']['uuid'] }}
{%   endif %}
{% endfor %}
```

{% endraw %}
And once we generate our inventory with Ansible it might look a little
something like below:

`terraform.inv`:

```bash
[docker_storage]
docker-storage-02.lab.etsbv.internal
docker-storage-01.lab.etsbv.internal

[docker_swarm_managers]
docker-mgr-01.lab.etsbv.internal
docker-mgr-02.lab.etsbv.internal
docker-mgr-03.lab.etsbv.internal

[docker_swarm_workers]
docker-wrk-03.lab.etsbv.internal
docker-wrk-04.lab.etsbv.internal
docker-wrk-01.lab.etsbv.internal
docker-wrk-02.lab.etsbv.internal

[docker_lbs]
docker-lb-02.lab.etsbv.internal
docker-lb-01.lab.etsbv.internal

[terraform_vms]
docker-storage-02.lab.etsbv.internal ansible_host=10.0.102.178 mac_address=00:50:56:aa:01:a0 uuid=422ad693-162f-1c32-b90c-eee1b0a73d2b
docker-storage-01.lab.etsbv.internal ansible_host=10.0.102.162 mac_address=00:50:56:aa:5f:cb uuid=422a264a-0816-60ff-475c-23af6c0b9d0e
docker-mgr-01.lab.etsbv.internal ansible_host=10.0.102.171 mac_address=00:50:56:aa:a9:c0 uuid=422abe05-4483-88f8-34b7-e354fdc7a211
docker-mgr-02.lab.etsbv.internal ansible_host=10.0.102.166 mac_address=00:50:56:aa:ba:a0 uuid=422a5d74-4de2-1df8-646b-ca62311f98ab
docker-mgr-03.lab.etsbv.internal ansible_host=10.0.102.179 mac_address=00:50:56:aa:e3:06 uuid=422a8d34-68f7-a7be-9c6a-18949ce809ed
docker-wrk-03.lab.etsbv.internal ansible_host=10.0.102.207 mac_address=00:50:56:aa:6b:d5 uuid=422a809a-0cd2-cd84-756b-0822bc3f813a
docker-wrk-04.lab.etsbv.internal ansible_host=10.0.102.183 mac_address=00:50:56:aa:b5:43 uuid=422aae57-67e8-50d8-66f6-3a11bdc87a78
docker-wrk-01.lab.etsbv.internal ansible_host=10.0.102.155 mac_address=00:50:56:aa:e4:93 uuid=422a87fe-baa2-75d6-e666-36c15f351269
docker-wrk-02.lab.etsbv.internal ansible_host=10.0.102.201 mac_address=00:50:56:aa:e0:36 uuid=422a49e3-746c-b550-189c-0e0179c60418
docker-lb-02.lab.etsbv.internal ansible_host=10.0.102.160 mac_address=00:50:56:aa:f8:25 uuid=422adcf8-347a-e9a5-e113-00114c1d2de9
docker-lb-01.lab.etsbv.internal ansible_host=10.0.102.163 mac_address=00:50:56:aa:9c:b3 uuid=422a4adb-e7d3-ea74-a69a-3ff10c13063f
```

## Final Notes

So there you have it. A creative way to use Ansible to generate/execute
you Terraform deployments. I have been using this methodology for a
short while now and it has been incredibly useful. Is it perfect? Of
course not. I would love to hear your thoughts and etc.

Enjoy!
