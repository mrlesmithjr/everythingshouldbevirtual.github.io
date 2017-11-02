---
  title: Ansible - Generate Rundeck Inventory
---

I have been doing some testing with Jenkins and Rundeck along with
Ansible lately. I wanted a way to number one not have to manually update
resources.xml for Rundeck but my biggest desire was to have Rundeck
populated with the same hosts that Ansible was managing.

`group_vars/all/configs`:

```yaml
rundeck_generate_resources_xml: true  #will rebuild resources.xml and prepare for delivery to your rundeck_server
rundeck_resources_xml_file: resources.xml
rundeck_resources_xml_location: /var/lib/rundeck/var
rundeck_server: ci-pipeline
```

`generate_rundeck_resources.yml`:

{% raw %}

```yaml
---
- name: identifying hosts
  action: ping
  register: id_hosts

- name: checking for exsisting resources.xml
  stat: path='{{ rundeck_resources_xml_location }}/{{ rundeck_resources_xml_file }}'
  delegate_to: '{{ rundeck_server }}'
  register: resources_xml
  run_once: true

- name: creating resources.xml
  template: src='{{ rundeck_resources_xml_file }}.j2' dest='{{ rundeck_resources_xml_location }}/{{ rundeck_resources_xml_file }}' owner=rundeck group=rundeck mode=0755
  delegate_to: '{{ rundeck_server }}'
  run_once: true
  when: not resources_xml.stat.exists

- name: creating rundeck resources.xml (not rundeck_server)
  lineinfile: dest="{{ rundeck_resources_xml_location }}/{{ rundeck_resources_xml_file }}"
              line=""
              insertbefore="^"
              state=present
  delegate_to: '{{ rundeck_server }}'
  with_items: id_hosts.results
  when: (inventory_hostname != rundeck_server)

- name: creating rundeck resources.xml (rundeck_server)
  lineinfile: dest="{{ rundeck_resources_xml_location }}/{{ rundeck_resources_xml_file }}"
              line=""
              insertbefore="^"
              state=present
  delegate_to: '{{ rundeck_server }}'
  with_items: id_hosts.results
  when: (inventory_hostname == rundeck_server)

- name: cleaning up tags in resources.xml
  shell: "sed -i -e "s/[][]//g" {{ rundeck_resources_xml_location }}/{{ rundeck_resources_xml_file }}"
  delegate_to: '{{ rundeck_server }}'
  run_once: true

- name: cleaning up resources.xml
  shell: "sed -i -e "s/'//g" {{ rundeck_resources_xml_location }}/{{ rundeck_resources_xml_file }}"
  delegate_to: '{{ rundeck_server }}'
  run_once: true

- name: ensuring resources.xml is still owned by rundeck user
  file: path="{{ rundeck_resources_xml_location }}/{{ rundeck_resources_xml_file }}" owner=rundeck group=rundeck mode=0755
  delegate_to: '{{ rundeck_server }}'
  run_once: true
```

{% endraw %}
Enjoy!
