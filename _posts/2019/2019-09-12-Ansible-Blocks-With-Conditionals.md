---
title: "Ansible Blocks With Conditionals"
date: "2019-09-12 13:18:00"
categories:
  - Automation
tags:
  - Ansible
---

## Ansible Blocks With Conditionals

### Background

Recently I had an interesting issue when using Ansible [Blocks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_blocks.html) in a playbook
which was skipping tasks within the block itself. I had no idea
that this was to be expected so I am sharing this here in case others stumble
upon this.

The playbook that I had created was importing another playbook using a loop
(Because Ansible does not support loops of tasks any other way that I know of)
a defined number of times. As part of the `Block`, I had a conditional which
evaluated whether or not something was true or false. And based on this
condition, additional tasks within the block were to be executed. Well I was
definitely caught by surprise once I understood what was occurring. And actually,
it is actually quite interesting to know.

### Example

For the context of this example I will not be using a looping playbook, but
instead a simplistic playbook to demonstrate what I saw.

#### Example 1

This example will demonstrate how I had originally put the playbook together and
we can see the results.

Playbook:

```yaml
---
- hosts: localhost
  gather_facts: false
  connection: local
  vars:
    debugging: true
  tasks:
    - block:
        - name: First Task
          debug:
            msg: First Task

        - name: Set debugging To False
          set_fact:
            debugging: false

        - name: Second Task
          debug:
            msg: Second Task

      when: debugging|bool
```

Based on the above playbook, I had expected both `First Task` and `Second Task`
to execute. But what we will see is, that this is not true at all.

Execution:

```bash
PLAY [localhost] *********************************************************************************************************************************************

TASK [First Task] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "First Task"
}

TASK [Set debugging To False] ********************************************************************************************************************************
ok: [localhost]

TASK [Second Task] *******************************************************************************************************************************************
skipping: [localhost]

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

Now if you review the results from above, you notice that the `Second Task`
was skipped. My initial thought was `WTF`, why did the `Second Task`
not execute because when I entered the block, `debugging` was set to `true`. So,
I would have assumed that even though I set `debugging: false` in the `set_fact`
task, that it would have still executed that `Second Task`. Not at all true.

#### Example 2

For this example, I will simply switch the tasks around to show that it works.
I know this is poor example because there really is not any reason that it **SHOULDN'T**
work. But again, for a simplistic demonstration it serves just fine because
our real concern is what we learned from [Example 1](#example-1).

Playbook:

```yaml
---
- hosts: localhost
  gather_facts: false
  connection: local
  vars:
    debugging: true
  tasks:
    - block:
        - name: First Task
          debug:
            msg: First Task

        - name: Second Task
          debug:
            msg: Second Task

        - name: Set debugging To False
          set_fact:
            debugging: false

      when: debugging|bool
```

Execution:

```bash
PLAY [localhost] *********************************************************************************************************************************************

TASK [First Task] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "First Task"
}

TASK [Second Task] *******************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Second Task"
}

TASK [Set debugging To False] ********************************************************************************************************************************
ok: [localhost]

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

As you can see, this worked as expected. Again, the real focus is on the [Example 1](#example-1) results.

### Learnings

Based on what I saw in my real use case in addition to these examples is the
following:

- If you are using a conditional as part of your `block`, ensure that you do
  not set the variable which defines whether the condition is met or not prior to
  any additional tasks within the block. **ALWAYS** set the condition as the last
  task inside the `block`.
- Ansible actually evaluates the conditional which is defined as part of the
  `block` for each and every task inside the `block` itself.

### Updated Example

I wanted to share this with others after I had someone reach out and ask about
using blocks and conditionals with roles. I felt this was a great question and
wanted to share the playbook we came up with.

{% gist mrlesmithjr/0bc55ad12743bbdead4cfc89d834a27e %}

### Conclusion

Hope this is of some use to others in case you ever run into this unexpected
scenario.

And as always, **ENJOY!!**
