---
  title: Ansible - Elasticsearch Curator Cron
---

Just wanted to put this together as I was finally just able to get this
working with the newest version of curator. The syntax has changed quite
a bit for defining jobs.

Define your `curator_max_keep_days` and `curator_close_after_days`
variables or accept the default values.
{% raw %}

```yaml
curator_max_keep_days: 14
curator_close_after_days: 7

---
# Install dependancies
- apt: name=python-pip state=present

# Install Curator
- pip: name=elasticsearch-curator

- name: remove old curator crontab
  file: path=/etc/cron.d/{{ item }} state=absent
  tags: [cron]
  with_items:
    - 'curator'
    - 'curator_clean'
    - 'curator_close'
    - 'curator_delete'

- name: install curator crontab
  cron: name='curator_delete'
        minute='0' hour='10'
        user='root'
        job='/usr/local/bin/curator --host {{ es_fqdn }} delete indices --time-unit days --timestring "\%Y.\%m.\%d" --older-than {{ curator_max_keep_days | default(360) }}'
        cron_file='curator_delete'
        state='present'
  tags: [cron]

- name: install curator crontab
  cron: name='curator_close'
        minute='45' hour='9'
        user='root'
        job='/usr/local/bin/curator --host {{ es_fqdn }} close indices --time-unit days --timestring "\%Y.\%m.\%d" --older-than {{ curator_close_after_days | default(14) }}'
        cron_file='curator_close'
        state='present'
  tags: [cron]
```

{% endraw %}

Enjoy!
