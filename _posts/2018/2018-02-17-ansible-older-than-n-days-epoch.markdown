---
title: "Ansible - Older Than N-Days Epoch"
date: "2018-02-17 01:27"
categories:
  - Automation
tags:
  - Ansible
---

I recently had a scenario in which I needed to determine if a device's last check-in
time had been longer than 30 days. The real fun with this is that the result of
the last check-in time was in [Epoch](https://en.wikipedia.org/wiki/Unix_time).
Oh this is going to be fun. So I started messing around and finally came up with
something that actually works quite well. So well that I also tested with times
that were not only 30, 60, and 90 days old, but also what does it look like with
future dates. Just for fun of course. So anyways below is what I came up with by
using a simple `Ansible` playbook for testing.

{% raw %}

```yaml
---
- hosts: localhost
  gather_facts: true
  become: false
  vars:
    example_epoch_times:
      - 1514764800
      - 1519862400
      - 1516253185
      - 1511481600
      - 1595548800
      - 1517443199
  tasks:
    - set_fact:
        _thirty_days_ago_epoch: "{{ (ansible_date_time['epoch']|int)-(86400*30) }}"
        _sixty_days_ago_epoch: "{{ (ansible_date_time['epoch']|int)-(86400*60) }}"
        _ninety_days_ago_epoch: "{{ (ansible_date_time['epoch']|int)-(86400*90) }}"

    - name: Displaying Older Than 30 Days
      debug:
        msg: "{{ '%Y-%m-%d %H:%M:%S' | strftime(item) }} has been longer than 30 days"
      with_items: "{{ example_epoch_times }}"
      when: (item/86400)+25569 < (_thirty_days_ago_epoch|int/86400)+25569

    - name: Displaying Older Than 60 Days
      debug:
        msg: "{{ '%Y-%m-%d %H:%M:%S' | strftime(item) }} has been longer than 60 days"
      with_items: "{{ example_epoch_times }}"
      when: (item/86400)+25569 < (_sixty_days_ago_epoch|int/86400)+25569

    - name: Displaying Older Than 90 Days
      debug:
        msg: "{{ '%Y-%m-%d %H:%M:%S' | strftime(item) }} has been longer than 90 days"
      with_items: "{{ example_epoch_times }}"
      when: (item/86400)+25569 < (_ninety_days_ago_epoch|int/86400)+25569

    - name: Displaying Less Than 30 Days
      debug:
        msg: "{{ '%Y-%m-%d %H:%M:%S' | strftime(item) }} has been less than 30 days"
      with_items: "{{ example_epoch_times }}"
      when: >
            (item/86400)+25569 > (_thirty_days_ago_epoch|int/86400)+25569 and
            (item/86400)+25569 < (ansible_date_time['epoch']|int/86400)+25569

    - name: Displaying Future
      debug:
        msg: "{{ '%Y-%m-%d %H:%M:%S' | strftime(item) }} is in the future"
      with_items: "{{ example_epoch_times }}"
      when: (item/86400)+25569 > (ansible_date_time['epoch']|int/86400)+25569
```

{% endraw %}

And below is what my results were:

```bash
PLAY [localhost] ***************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************
ok: [localhost]

TASK [set_fact] ****************************************************************************************************************************************************************
ok: [localhost]

TASK [Displaying Older Than 30 Days] *******************************************************************************************************************************************
ok: [localhost] => (item=1514764800) => {
    "changed": false,
    "item": 1514764800,
    "msg": "2017-12-31 19:00:00 has been longer than 30 days"
}
skipping: [localhost] => (item=1519862400)
ok: [localhost] => (item=1516253185) => {
    "changed": false,
    "item": 1516253185,
    "msg": "2018-01-18 00:26:25 has been longer than 30 days"
}
ok: [localhost] => (item=1511481600) => {
    "changed": false,
    "item": 1511481600,
    "msg": "2017-11-23 19:00:00 has been longer than 30 days"
}
skipping: [localhost] => (item=1595548800)
skipping: [localhost] => (item=1517443199)

TASK [Displaying Older Than 60 Days] *******************************************************************************************************************************************
skipping: [localhost] => (item=1514764800)
skipping: [localhost] => (item=1519862400)
skipping: [localhost] => (item=1516253185)
ok: [localhost] => (item=1511481600) => {
    "changed": false,
    "item": 1511481600,
    "msg": "2017-11-23 19:00:00 has been longer than 60 days"
}
skipping: [localhost] => (item=1595548800)
skipping: [localhost] => (item=1517443199)

TASK [Displaying Older Than 90 Days] *******************************************************************************************************************************************
skipping: [localhost] => (item=1514764800)
skipping: [localhost] => (item=1519862400)
skipping: [localhost] => (item=1516253185)
skipping: [localhost] => (item=1511481600)
skipping: [localhost] => (item=1595548800)
skipping: [localhost] => (item=1517443199)

TASK [Displaying Less Than 30 Days] ********************************************************************************************************************************************
skipping: [localhost] => (item=1514764800)
skipping: [localhost] => (item=1519862400)
skipping: [localhost] => (item=1516253185)
skipping: [localhost] => (item=1511481600)
skipping: [localhost] => (item=1595548800)
ok: [localhost] => (item=1517443199) => {
    "changed": false,
    "item": 1517443199,
    "msg": "2018-01-31 18:59:59 has been less than 30 days"
}

TASK [Displaying Future] *******************************************************************************************************************************************************
skipping: [localhost] => (item=1514764800)
ok: [localhost] => (item=1519862400) => {
    "changed": false,
    "item": 1519862400,
    "msg": "2018-02-28 19:00:00 is in the future"
}
skipping: [localhost] => (item=1516253185)
skipping: [localhost] => (item=1511481600)
ok: [localhost] => (item=1595548800) => {
    "changed": false,
    "item": 1595548800,
    "msg": "2020-07-23 20:00:00 is in the future"
}
skipping: [localhost] => (item=1517443199)

PLAY RECAP *********************************************************************************************************************************************************************
localhost                  : ok=6    changed=0    unreachable=0    failed=0
```

So there you have it. Hopefully this will be useful to others as well. But if not,
at least I now have a decent bookmark of how to do this.

Enjoy!
