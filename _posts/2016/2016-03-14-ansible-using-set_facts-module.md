---
  title: Ansible - Using the set_facts module
  categories:
    - Automation
  tags:
    - Ansible
---

Today I was writing/updating an Ansible role for installing Cacti
monitoring and decided to add the ability to choose the back-end
webserver being used. The original role was written based on using
Apache as the webserver back-end. However I wanted the ability to choose
between Apache, NGINX or Lighttpd. Being that I have created Ansible
roles for each already I figured I may as well leverage those. So I
started messing with different variables and testing to see how things
would work and it just was becoming too complex and would be rather
difficult for someone potentially not understanding what all was going
on. Well I finally came around to a solution that for the most part
should remain quite dynamic and wanted to share that with everyone in
case you ever find yourself in this same situation.

So what did I need the variables to define? I needed a way to set
variables for the following:

-   web_group - which would define based on webserver type and OS type what the webserver groupname was.
-   web_owner - which would define based on webserver type and OS type what the webserver username was.
-   web_root - which would define based on webserver type and OS type what the webserver default root directory was.
-   webserver_handler - which would define based on webserver type and OS type what the servicename was in order to restart after configuration changes were made.

So as you can see this could get a little out of control when the
end-user needs to define all of these different settings and such. So I
finally settled on utilizing the
[set_facts](http://docs.ansible.com/ansible/set_fact_module.html)
Ansible module which was a perfect fit for my needs.

Now for what ended up working is what I will share below.

In the role's main task I added the following:

```yaml
---
# tasks file for ansible-cacti
- include: set_facts.yml

- include: debian.yml
  when: ansible_os_family == "Debian"

- include: redhat.yml
  when: ansible_os_family == "RedHat"

- include: users.yml

- include: database.yml

- include: config_cacti.yml

- include: templates.yml
  when: cacti_import_templates
```

If you notice the very first include is to execute a task called
_set_facts.yml_ and this is where the magic comes into play. So let's
take a look at that task's contents.
{% raw %}

```yaml
---
- name: setting fact Debian apache2
  set_fact:
    cacti_web_group: "www-data"
    cacti_web_owner: "www-data"
    cacti_web_root: "/var/www/html"
    cacti_webserver_handler: "apache2"
  when: >
        ansible_os_family == "Debian" and
        cacti_webserver_type == "apache2"

- name: setting fact Debian lighttpd
  set_fact:
    cacti_web_group: "www-data"
    cacti_web_owner: "www-data"
    cacti_web_root: "/var/www"
    cacti_webserver_handler: "lighttpd"
  when: >
        ansible_os_family == "Debian" and
        cacti_webserver_type == "lighttpd"

- name: setting fact Debian nginx
  set_fact:
    cacti_web_group: "www-data"
    cacti_web_owner: "www-data"
    cacti_web_root: "/usr/share/nginx/html"
    cacti_webserver_handler: "nginx"
  when: >
        ansible_os_family == "Debian" and
        cacti_webserver_type == "nginx"

- name: setting fact RedHat apache2
  set_fact:
    cacti_web_group: "apache"
    cacti_web_owner: "apache"
    cacti_web_root: "/var/www/html"
    cacti_webserver_handler: "httpd"
  when: >
        ansible_os_family == "RedHat" and
        cacti_webserver_type == "apache2"

- name: setting fact RedHat lighttpd
  set_fact:
    cacti_web_group: "lighttpd"
    cacti_web_owner: "lighttpd"
    cacti_web_root: "/srv/www"
    cacti_webserver_handler: "lighttpd"
  when: >
        ansible_os_family == "RedHat" and
        cacti_webserver_type == "lighttpd"

- name: setting fact RedHat nginx
  set_fact:
    cacti_web_group: "nginx"
    cacti_web_owner: "nginx"
    cacti_web_root: "/usr/share/nginx/html"
    cacti_webserver_handler: "nginx"
  when: >
        ansible_os_family == "RedHat" and
        cacti_webserver_type == "nginx"
```

{% endraw %}
And if we take a look at the above we will see that for each different
OS type (Debian or RedHat) we are setting the following facts:

```yaml
    cacti_web_group:
    cacti_web_owner:
    cacti_web_root:
    cacti_webserver_handler:
```

Doing this allows for us to leverage fewer tasks to accomplish the same
things. So for example if we are deploying on an Ubuntu or Debian OS and
would like to use NGINX for our back-end webserver we would set the
following variable in our _defaults/main.yml_:

```yaml
    cacti_webserver_type: 'nginx'  #defines web server type (apache2|lighttpd|nginx)
```

And if we were to apply the role the following variables would be set
for us to use throughout the Ansible play.

```yaml
    cacti_web_group: "www-data"
    cacti_web_owner: "www-data"
    cacti_web_root: "/usr/share/nginx/html"
    cacti_webserver_handler: "nginx"
```

And when we reach a task that requires one of these variables it would
be ready for use. For example:
{% raw %}

```yaml
    - name: config_cacti | setting site permissions
      file:
        path: "{{ cacti_web_root }}/cacti-{{ cacti_version }}"
        state: directory
        recurse: yes
        owner: "{{ cacti_web_owner }}"
        group: "{{ cacti_web_group }}"
```

{% endraw %}
And if we reach a task in which requires a notifier to fire off a
restart of the webserver service:
{% raw %}

```yaml
    - name: debian | installing php5
      apt:
        name: "php5"
        state: present
      notify:
        - 'restart {{ cacti_webserver_handler }}'
```

{% endraw %}
So as you can see this makes for a very nice way to handle defining
variables in a complex role much easier and dynamic. And if we decided
we would rather use Lighttpd as our back-end webserver all we would need
to do is define the following in our _defaults.yml_:

```yaml
    cacti_webserver_type: 'lighttpd'  #defines web server type (apache2|lighttpd|nginx)
```

And that is the only variable we would need to define in order to change
our deployment of the Cacti role. Pretty easy right?\
Now below is what an example playbook might look like in order to
leverage setting our back-end webserver:

```yaml
    ---
    - hosts: all
      become: true
      vars:
        - cacti_webserver_type: 'apache2'
        - pri_domain_name: 'vagrant.local'
      roles:
        - role: ansible-apache2
          when: cacti_webserver_type == "apache2"
        - role: ansible-lighttpd
          when: cacti_webserver_type == "lighttpd"
        - role: ansible-nginx
          when: cacti_webserver_type == "nginx"
        - role: ansible-mariadb-mysql
        - role: ansible-ntp
        - role: ansible-snmpd
        - role: ansible-timezone
        - role: ansible-cacti
      tasks:
```

And using the above playbook would deploy Cacti using Apache2 as our
back-end webserver.

So there you have it. A very good use-case of using the set_facts
Ansible module.

Enjoy!
