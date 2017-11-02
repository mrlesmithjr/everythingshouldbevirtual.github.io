---
  title: Quick Script to inject SSH Keys for hostnames and ip address
---

While playing around with [Rundeck](http://rundeck.org) I wanted to run
some quick [Ansible](http://ansible.com) playbooks. I have not found a
good way to integrate Ansible other than running local commands that
actually run the playbooks (early learning issue I assume :) ).

So with this I had an issue running the Ansible playbooks because of
ssh-key's not known. I can successfully run commands within Rundeck to
those hosts without issue but when running the Ansible script I ran into
the issue. So here I am sharing a script that I created within Rundeck.

```bash
cd /tmp
# create hosts
echo ans-test-1 > hosts
echo ans-test-2 >> hosts
echo ans-test-3 >> hosts
echo ans-test-4 >> hosts
echo ans-test-5 >> hosts
# grab IP addresses from hosts
for node in $(cat hosts); do
  nslookup $node | grep 'Address\:' | awk 'NR==2 {print $2}' >> hosts
done
# add ssh keys for both hostname and ip for each host
for node in $(cat hosts); do
  ssh-keyscan -H $node >> ~/.ssh/known_hosts
done
# Cleanup known_hosts for duplicate entries
sort -u ~/.ssh/known_hosts > ~/.ssh/known_hosts.clean
mv ~/.ssh/known_hosts ~/.ssh/known_hosts.backup
cp ~/.ssh/known_hosts.clean ~/.ssh/known_hosts
# make sure destination exists and is writable for rundeck user... This was erroring out with permission denied for rundeck user.
sudo mkdir /var/lib/rundeck/.ansible/
sudo chown -R rundeck:rundeck /var/lib/rundeck/.ansible/
cd /tmp/ansible
ansible-playbook -i hosts ansible-test.yml --user rundeck
```

Hope this helps someone.

Enjoy!
