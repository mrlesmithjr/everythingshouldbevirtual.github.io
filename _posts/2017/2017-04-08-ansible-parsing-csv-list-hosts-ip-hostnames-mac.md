---
  title: Ansible - Parsing CSV List Of Hosts (IP, hostname(s), MAC)
  categories:
    - Automation
  tags:
    - Ansible
  redirect_from:
    - /ansible-parsing-csv-list-hosts-ip-hostnames-mac
---

Recently I had a need to take an already populated spreadsheet which
contained a list of hostnames, generic names, IP addresses and MAC
addresses and convert them to a usable YAML format to be used with
Ansible. This is something I had attempted previously but was not very
successful, however I gave it another shot. This time I went ahead and
took the time to create a Python script to use as a generator to
create the inventory in order to test many times with different datasets
and to also share with others to do the same. The good news is that I
was successful this time around so I wanted to share this with others.

The Ansible playbook is pretty straight forward as seen below. I
attempted the Ansible csvfile lookup at first and it was not as flexible
as I had hoped for so I ended up just leveraging the file lookup
instead.

{% gist mrlesmithjr/88398e09d872fcac19e96940325bde4b %}

The Jinja2 template ended up to be equally as straightforward as seen
below.

{% gist mrlesmithjr/3b1cde5ee12bff76f31b9317a3a7e2f9 %}

As mentioned in the beginning of the post, I wanted a way to generate
some random datasets using a Python script. And that script is below.

{% gist mrlesmithjr/df87dd3d019a734838e964ca52624922 %}

To generate a dataset using the above script all that is required is to
adjust the variables in the beginning of the script to my specific needs
and then execute the script as below:

```bash
./hostipmacgen.py
```

After running the above script we now have a random inventory generated
for 1000 hosts in a CSV file and ready for us to parse this to YAML in
the next section.

{% gist mrlesmithjr/4f08f143789555cc1c8f679d510c6b20 %}

Now that we have our CSV generated from above we can now run the Ansible
playbook to parse this data to YAML for us.

```bash
ansible-playbook interate_csv.yml
```

And once the above playbook runs (~5 secs.) we now have this handy
usable YAML file ready for us to do some Ansible goodness with. Your
options are wide open at this point!

{% gist mrlesmithjr/f118c5cb2335b1be86a8dbeb64217cb9 %}

And there you have it. An easy way to parse some data from a spreadsheet
exported as a CSV file and easily becoming available for consumption by
Ansible!

Enjoy!
