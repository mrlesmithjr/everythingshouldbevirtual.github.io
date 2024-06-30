---
title: Updating Git Project Structure
date: 2020-02-20 18:00:00
---

## Introduction

Creating and maintaining a consistent project structure is crucial for efficient collaboration and automation. In this post, I'll share how I've been using a [Cookiecutter](https://cookiecutter.readthedocs.io/ "https://cookiecutter.readthedocs.io/") template to streamline the creation of new [Ansible](https://ansible.com/ "https://ansible.com/") roles and how I plan to update my existing roles to fit this new structure.

## Creating the Cookiecutter Template

Lately, I have been working on putting together a Cookiecutter template to use when creating new Ansible roles. [This](https://github.com/mrlesmithjr/cookiecutter-ansible-role "https://github.com/mrlesmithjr/cookiecutter-ansible-role") template includes several key features:

- **Ansible Role Structure**
- **Continuous Integration (CI)**
    - GitHub Actions
    - GitLab CI/CD
    - Travis
- **Molecule Testing**
- **Documentation**
    - Code of Conduct
    - Contributing Guidelines
    - License
    - Readme

## Benefits of a Consistent Structure

Implementing a consistent structure has several advantages:

- **Ease of Use:** Developers can quickly start new projects with a familiar setup.
- **Maintainability:** Standardized practices make it easier to maintain and update roles.
- **Collaboration:** A standard structure improves collaboration among team members.

## Challenges and Solutions

This sounds great for creating new Ansible roles, but what about the hundreds of existing roles I already have? How will I incorporate all of them into this same new structure?

### Migration Strategy

Here are some steps to consider for migrating existing roles:

1. **Assessment:** Evaluate the existing roles to understand the scope of changes needed.
2. **Automation:** Use automation scripts to apply the new structure to existing roles.
3. **Testing:** Ensure thorough testing to validate that the migrated roles function correctly.

## Examples and Implementation

In this example, I will be working in my [ansible-control-machine](https://github.com/mrlesmithjr/ansible-control-machine.git "https://github.com/mrlesmithjr/ansible-control-machine.git") Ansible role.

The first thing I will do is clone the project, but I will be cloning to the `ansible-control-machine.orig` directory:

```bash
git clone git@github.com:mrlesmithjr/ansible-control-machine.git ansible-control-machine.orig
...
Cloning into 'ansible-control-machine.orig'...
remote: Enumerating objects: 20, done.
remote: Counting objects: 100% (20/20), done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 141 (delta 8), reused 13 (delta 5), pack-reused 121
Receiving objects: 100% (141/141), 35.10 KiB | 1.67 MiB/s, done.
Resolving deltas: 100% (50/50), done.
```

Let's check real quick to see what our original structure looked like:

```bash
ls -la ansible-control-machine.orig
...
total 32
drwxr-xr-x  14 larrysmithjr  staff   448 Feb 20 21:25 .
drwxr-xr-x  31 larrysmithjr  staff   992 Feb 20 21:31 ..
drwxr-xr-x  12 larrysmithjr  staff   384 Feb 20 21:25 .git
-rw-r--r--   1 larrysmithjr  staff  2427 Feb 20 21:25 .travis.yml
-rw-r--r--   1 larrysmithjr  staff  1486 Feb 20 21:25 .yamllint.yml
-rw-r--r--   1 larrysmithjr  staff  2050 Feb 20 21:25 README.md
drwxr-xr-x  15 larrysmithjr  staff   480 Feb 20 21:25 Vagrant
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:25 defaults
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:25 handlers
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:25 meta
-rwxr-xr-x   1 larrysmithjr  staff   419 Feb 20 21:25 setup_travis_tests.sh
drwxr-xr-x   6 larrysmithjr  staff   192 Feb 20 21:25 tasks
drwxr-xr-x  17 larrysmithjr  staff   544 Feb 20 21:25 tests
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:25 vars
```

Next, I will launch `cookiecutter` and use my [cookiecutter-ansible-role](https://github.com/mrlesmithjr/cookiecutter-ansible-role "https://github.com/mrlesmithjr/cookiecutter-ansible-role") template to create a new project called `ansible-control-machine`.

```bash
cookiecutter https://github.com/mrlesmithjr/cookiecutter-ansible-role.git
```

Following the prompts, I will fill in the details.

```bash
role_name [Enter Ansible role name]: ansible-control-machine
description [Enter description of Ansible role]: Ansible role to build an Ansible control machine
author [Your Name]: Larry Smith Jr.
company [Enter company name]:
email [me@example.com]: mrlesmithjr@gmail.com
website [http://example.com]: http://everythingshouldbevirtual.com
twitter [example]: mrlesmithjr
Select license:
1 - MIT
2 - BSD-3
3 - Apache Software License 2.0
Choose from 1, 2, 3 [1]:
min_ansible_version [2.8]:
year [2020]:
github_username [Enter your GitHub username]: mrlesmithjr
travis_username [Enter your Travis CI username]: mrlesmithjr
Select default_ci_badges:
1 - Y
2 - N
Choose from 1, 2 [1]:
```

Now I should have a new directory called `ansible-control-machine`. So, let's see what the new structure looks like:

```bash
ls -la ansible-control-machine
...
total 96
drwxr-xr-x  24 larrysmithjr  staff   768 Feb 20 21:31 .
drwxr-xr-x  31 larrysmithjr  staff   992 Feb 20 21:31 ..
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:31 .github
-rw-r--r--   1 larrysmithjr  staff     6 Feb 20 21:31 .gitignore
-rw-r--r--   1 larrysmithjr  staff   417 Feb 20 21:31 .gitlab-ci.yml
-rw-r--r--   1 larrysmithjr  staff   271 Feb 20 21:31 .travis.yml
-rw-r--r--   1 larrysmithjr  staff   617 Feb 20 21:31 .yamllint
-rw-r--r--   1 larrysmithjr  staff  3356 Feb 20 21:31 CODE_OF_CONDUCT.md
-rw-r--r--   1 larrysmithjr  staff   400 Feb 20 21:31 CONTRIBUTING.md
-rw-r--r--   1 larrysmithjr  staff    40 Feb 20 21:31 CONTRIBUTORS.md
-rw-r--r--   1 larrysmithjr  staff  1072 Feb 20 21:31 LICENSE.md
-rw-r--r--   1 larrysmithjr  staff  1037 Feb 20 21:31 README.md
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:31 defaults
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:31 files
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:31 handlers
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:31 meta
drwxr-xr-x   5 larrysmithjr  staff   160 Feb 20 21:31 molecule
-rw-r--r--   1 larrysmithjr  staff    87 Feb 20 21:31 playbook.yml
-rw-r--r--   1 larrysmithjr  staff    37 Feb 20 21:31 requirements-dev.txt
-rw-r--r--   1 larrysmithjr  staff    89 Feb 20 21:31 requirements.txt
-rw-r--r--   1 larrysmithjr  staff     0 Feb 20 21:31 requirements.yml
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:31 tasks
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:31 templates
drwxr-xr-x   3 larrysmithjr  staff    96 Feb 20 21:31 vars
```

As you can see, there is much more in the new structure than in the original.

Now is where the fun begins :)

Let's change into the `ansible-control-machine` directory:

```bash
cd ansible-control-machine
```

Now let's do a quick `git status`:

```bash
git status
...
fatal: not a git repository (or any of the parent directories): .git
```

Oh no! Where is my repo info? The answer is there has yet to be any in the new structure. Let's see how we can get that back into our new structure.

We will do that by simply copying the .git directory from our original project, which will bring all that to our new directory.

```bash
cp -Rv ../ansible-control-machine.orig/.git .
...
../ansible-control-machine.orig/.git -> ./.git
../ansible-control-machine.orig/.git/config -> ./.git/config
../ansible-control-machine.orig/.git/objects -> ./.git/objects
../ansible-control-machine.orig/.git/objects/pack -> ./.git/objects/pack
../ansible-control-machine.orig/.git/objects/pack/pack-35613ed67c65538902f0322dc253bcc6b19acd31.pack -> ./.git/objects/pack/pack-35613ed67c65538902f0322dc253bcc6b19acd31.pack
../ansible-control-machine.orig/.git/objects/pack/pack-35613ed67c65538902f0322dc253bcc6b19acd31.idx -> ./.git/objects/pack/pack-35613ed67c65538902f0322dc253bcc6b19acd31.idx
../ansible-control-machine.orig/.git/objects/info -> ./.git/objects/info
../ansible-control-machine.orig/.git/HEAD -> ./.git/HEAD
../ansible-control-machine.orig/.git/info -> ./.git/info
../ansible-control-machine.orig/.git/info/exclude -> ./.git/info/exclude
../ansible-control-machine.orig/.git/logs -> ./.git/logs
../ansible-control-machine.orig/.git/logs/HEAD -> ./.git/logs/HEAD
../ansible-control-machine.orig/.git/logs/refs -> ./.git/logs/refs
../ansible-control-machine.orig/.git/logs/refs/heads -> ./.git/logs/refs/heads
../ansible-control-machine.orig/.git/logs/refs/heads/master -> ./.git/logs/refs/heads/master
../ansible-control-machine.orig/.git/logs/refs/remotes -> ./.git/logs/refs/remotes
../ansible-control-machine.orig/.git/logs/refs/remotes/origin -> ./.git/logs/refs/remotes/origin
../ansible-control-machine.orig/.git/logs/refs/remotes/origin/HEAD -> ./.git/logs/refs/remotes/origin/HEAD
../ansible-control-machine.orig/.git/description -> ./.git/description
../ansible-control-machine.orig/.git/hooks -> ./.git/hooks
../ansible-control-machine.orig/.git/hooks/commit-msg.sample -> ./.git/hooks/commit-msg.sample
../ansible-control-machine.orig/.git/hooks/pre-rebase.sample -> ./.git/hooks/pre-rebase.sample
../ansible-control-machine.orig/.git/hooks/pre-commit.sample -> ./.git/hooks/pre-commit.sample
../ansible-control-machine.orig/.git/hooks/applypatch-msg.sample -> ./.git/hooks/applypatch-msg.sample
../ansible-control-machine.orig/.git/hooks/fsmonitor-watchman.sample -> ./.git/hooks/fsmonitor-watchman.sample
../ansible-control-machine.orig/.git/hooks/pre-receive.sample -> ./.git/hooks/pre-receive.sample
../ansible-control-machine.orig/.git/hooks/prepare-commit-msg.sample -> ./.git/hooks/prepare-commit-msg.sample
../ansible-control-machine.orig/.git/hooks/post-update.sample -> ./.git/hooks/post-update.sample
../ansible-control-machine.orig/.git/hooks/pre-merge-commit.sample -> ./.git/hooks/pre-merge-commit.sample
../ansible-control-machine.orig/.git/hooks/pre-applypatch.sample -> ./.git/hooks/pre-applypatch.sample
../ansible-control-machine.orig/.git/hooks/pre-push.sample -> ./.git/hooks/pre-push.sample
../ansible-control-machine.orig/.git/hooks/update.sample -> ./.git/hooks/update.sample
../ansible-control-machine.orig/.git/refs -> ./.git/refs
../ansible-control-machine.orig/.git/refs/heads -> ./.git/refs/heads
../ansible-control-machine.orig/.git/refs/heads/master -> ./.git/refs/heads/master
../ansible-control-machine.orig/.git/refs/tags -> ./.git/refs/tags
../ansible-control-machine.orig/.git/refs/remotes -> ./.git/refs/remotes
../ansible-control-machine.orig/.git/refs/remotes/origin -> ./.git/refs/remotes/origin
../ansible-control-machine.orig/.git/refs/remotes/origin/HEAD -> ./.git/refs/remotes/origin/HEAD
../ansible-control-machine.orig/.git/index -> ./.git/index
../ansible-control-machine.orig/.git/packed-refs -> ./.git/packed-refs
```

Once the copy is complete, we can do another `git status` to see how things look now:

```bash
git status
...
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
 modified:   .travis.yml
 deleted:    .yamllint.yml
 modified:   README.md
 deleted:    Vagrant/.gitignore
 deleted:    Vagrant/Vagrantfile
 deleted:    Vagrant/ansible.cfg
 deleted:    Vagrant/bootstrap.sh
 deleted:    Vagrant/bootstrap.yml
 deleted:    Vagrant/cleanup.bat
 deleted:    Vagrant/cleanup.sh
 deleted:    Vagrant/hosts
 deleted:    Vagrant/nodes.yml
 deleted:    Vagrant/playbook.yml
 deleted:    Vagrant/prep.sh
 deleted:    Vagrant/requirements.yml
 deleted:    Vagrant/roles/ansible-control-machine
 modified:   defaults/main.yml
 modified:   meta/main.yml
 deleted:    setup_travis_tests.sh
 deleted:    tasks/debian.yml
 modified:   tasks/main.yml
 deleted:    tasks/redhat.yml
 deleted:    tasks/setup.yml
 deleted:    tests/.ansible-lint
 deleted:    tests/Dockerfile.centos-7
 deleted:    tests/Dockerfile.debian-jessie
 deleted:    tests/Dockerfile.debian-stretch
 deleted:    tests/Dockerfile.fedora-24
 deleted:    tests/Dockerfile.fedora-25
 deleted:    tests/Dockerfile.fedora-26
 deleted:    tests/Dockerfile.fedora-27
 deleted:    tests/Dockerfile.fedora-28
 deleted:    tests/Dockerfile.fedora-29
 deleted:    tests/Dockerfile.ubuntu-bionic
 deleted:    tests/Dockerfile.ubuntu-trusty
 deleted:    tests/Dockerfile.ubuntu-xenial
 deleted:    tests/inventory
 deleted:    tests/test.yml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
 .github/workflows/default.yml
 .gitignore
 .gitlab-ci.yml
 .yamllint
 CODE_OF_CONDUCT.md
 CONTRIBUTING.md
 CONTRIBUTORS.md
 LICENSE.md
 files/.gitkeep
 molecule/default/Dockerfile.j2
 molecule/default/INSTALL.rst
 molecule/default/molecule.yml
 molecule/shared/converge.yml
 molecule/shared/tests/test_default.py
 molecule/vagrant/INSTALL.rst
 molecule/vagrant/molecule.yml
 molecule/vagrant/prepare.yml
 playbook.yml
 requirements-dev.txt
 requirements.txt
 requirements.yml
 templates/.gitkeep

no changes added to commit (use "git add" and/or "git commit -a")
```

Well, that looks better but scary. Of course not! Now, all we need to do is decide what we want to keep and what we want to get rid of. But before we do that, let's create a new branch so we are not messing with `master`. By doing this, we are ensuring that we don't mess anything up in `master` in case we do something wrong.

So, let's create a new branch called `updating-structure`:

```bash
git checkout -b updating-structure
...
Switched to a new branch 'updating-structure'
```

Now that we are in our `updating-structure` branch, we can start by checking out anything marked as deleted that we want to keep.

```bash
git status | grep deleted
...
 deleted:    .yamllint.yml
 deleted:    Vagrant/.gitignore
 deleted:    Vagrant/Vagrantfile
 deleted:    Vagrant/ansible.cfg
 deleted:    Vagrant/bootstrap.sh
 deleted:    Vagrant/bootstrap.yml
 deleted:    Vagrant/cleanup.bat
 deleted:    Vagrant/cleanup.sh
 deleted:    Vagrant/hosts
 deleted:    Vagrant/nodes.yml
 deleted:    Vagrant/playbook.yml
 deleted:    Vagrant/prep.sh
 deleted:    Vagrant/requirements.yml
 deleted:    Vagrant/roles/ansible-control-machine
 deleted:    setup_travis_tests.sh
 deleted:    tasks/debian.yml
 deleted:    tasks/redhat.yml
 deleted:    tasks/setup.yml
 deleted:    tests/.ansible-lint
 deleted:    tests/Dockerfile.centos-7
 deleted:    tests/Dockerfile.debian-jessie
 deleted:    tests/Dockerfile.debian-stretch
 deleted:    tests/Dockerfile.fedora-24
 deleted:    tests/Dockerfile.fedora-25
 deleted:    tests/Dockerfile.fedora-26
 deleted:    tests/Dockerfile.fedora-27
 deleted:    tests/Dockerfile.fedora-28
 deleted:    tests/Dockerfile.fedora-29
 deleted:    tests/Dockerfile.ubuntu-bionic
 deleted:    tests/Dockerfile.ubuntu-trusty
 deleted:    tests/Dockerfile.ubuntu-xenial
 deleted:    tests/inventory
 deleted:    tests/test.yml
```

Because this is for an Ansible role, I want to keep anything in `tasks` from the above. So I'll `checkout` those to keep them.

```bash
git checkout tasks
...
Updated 4 paths from the index
```

Let's check our `deleted` files once more to make sure we are good:

```bash
git status | grep deleted
...
 deleted:    .yamllint.yml
 deleted:    Vagrant/.gitignore
 deleted:    Vagrant/Vagrantfile
 deleted:    Vagrant/ansible.cfg
 deleted:    Vagrant/bootstrap.sh
 deleted:    Vagrant/bootstrap.yml
 deleted:    Vagrant/cleanup.bat
 deleted:    Vagrant/cleanup.sh
 deleted:    Vagrant/hosts
 deleted:    Vagrant/nodes.yml
 deleted:    Vagrant/playbook.yml
 deleted:    Vagrant/prep.sh
 deleted:    Vagrant/requirements.yml
 deleted:    Vagrant/roles/ansible-control-machine
 deleted:    setup_travis_tests.sh
 deleted:    tests/.ansible-lint
 deleted:    tests/Dockerfile.centos-7
 deleted:    tests/Dockerfile.debian-jessie
 deleted:    tests/Dockerfile.debian-stretch
 deleted:    tests/Dockerfile.fedora-24
 deleted:    tests/Dockerfile.fedora-25
 deleted:    tests/Dockerfile.fedora-26
 deleted:    tests/Dockerfile.fedora-27
 deleted:    tests/Dockerfile.fedora-28
 deleted:    tests/Dockerfile.fedora-29
 deleted:    tests/Dockerfile.ubuntu-bionic
 deleted:    tests/Dockerfile.ubuntu-trusty
 deleted:    tests/Dockerfile.ubuntu-xenial
 deleted:    tests/inventory
 deleted:    tests/test.yml
```

It looks good. So, now I will get rid of the `Vagrant` and `tests` directories because I know these are for testing, and I'll be replacing them with the new Molecule tests.

```bash
git add Vagrant/ tests/
```

And another quick `git status` shows:

```bash
On branch updating-structure
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
 deleted:    Vagrant/.gitignore
 deleted:    Vagrant/Vagrantfile
 deleted:    Vagrant/ansible.cfg
 deleted:    Vagrant/bootstrap.sh
 deleted:    Vagrant/bootstrap.yml
 deleted:    Vagrant/cleanup.bat
 deleted:    Vagrant/cleanup.sh
 deleted:    Vagrant/hosts
 deleted:    Vagrant/nodes.yml
 deleted:    Vagrant/playbook.yml
 deleted:    Vagrant/prep.sh
 deleted:    Vagrant/requirements.yml
 deleted:    Vagrant/roles/ansible-control-machine
 deleted:    tests/.ansible-lint
 deleted:    tests/Dockerfile.centos-7
 deleted:    tests/Dockerfile.debian-jessie
 deleted:    tests/Dockerfile.debian-stretch
 deleted:    tests/Dockerfile.fedora-24
 deleted:    tests/Dockerfile.fedora-25
 deleted:    tests/Dockerfile.fedora-26
 deleted:    tests/Dockerfile.fedora-27
 deleted:    tests/Dockerfile.fedora-28
 deleted:    tests/Dockerfile.fedora-29
 deleted:    tests/Dockerfile.ubuntu-bionic
 deleted:    tests/Dockerfile.ubuntu-trusty
 deleted:    tests/Dockerfile.ubuntu-xenial
 deleted:    tests/inventory
 deleted:    tests/test.yml

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
 modified:   .travis.yml
 deleted:    .yamllint.yml
 modified:   README.md
 modified:   defaults/main.yml
 modified:   meta/main.yml
 deleted:    setup_travis_tests.sh

Untracked files:
  (use "git add <file>..." to include in what will be committed)
 .github/workflows/default.yml
 .gitignore
 .gitlab-ci.yml
 .yamllint
 CODE_OF_CONDUCT.md
 CONTRIBUTING.md
 CONTRIBUTORS.md
 LICENSE.md
 files/.gitkeep
 molecule/default/Dockerfile.j2
 molecule/default/INSTALL.rst
 molecule/default/molecule.yml
 molecule/shared/converge.yml
 molecule/shared/tests/test_default.py
 molecule/vagrant/INSTALL.rst
 molecule/vagrant/molecule.yml
 molecule/vagrant/prepare.yml
 playbook.yml
 requirements-dev.txt
 requirements.txt
 requirements.yml
 templates/.gitkeep
```

I'm happy with this, so I'll now commit those changes:

```bash
git commit -m "Deleted old tests, etc. not needed"
...
[updating-structure a117fcf] Deleted old tests, etc. not needed
 28 files changed, 1202 deletions(-)
 delete mode 100644 Vagrant/.gitignore
 delete mode 100644 Vagrant/Vagrantfile
 delete mode 100644 Vagrant/ansible.cfg
 delete mode 100755 Vagrant/bootstrap.sh
 delete mode 100644 Vagrant/bootstrap.yml
 delete mode 100644 Vagrant/cleanup.bat
 delete mode 100755 Vagrant/cleanup.sh
 delete mode 120000 Vagrant/hosts
 delete mode 100644 Vagrant/nodes.yml
 delete mode 100644 Vagrant/playbook.yml
 delete mode 100755 Vagrant/prep.sh
 delete mode 100644 Vagrant/requirements.yml
 delete mode 120000 Vagrant/roles/ansible-control-machine
 delete mode 100644 tests/.ansible-lint
 delete mode 100644 tests/Dockerfile.centos-7
 delete mode 100644 tests/Dockerfile.debian-jessie
 delete mode 100644 tests/Dockerfile.debian-stretch
 delete mode 100644 tests/Dockerfile.fedora-24
 delete mode 100644 tests/Dockerfile.fedora-25
 delete mode 100644 tests/Dockerfile.fedora-26
 delete mode 100644 tests/Dockerfile.fedora-27
 delete mode 100644 tests/Dockerfile.fedora-28
 delete mode 100644 tests/Dockerfile.fedora-29
 delete mode 100644 tests/Dockerfile.ubuntu-bionic
 delete mode 100644 tests/Dockerfile.ubuntu-trusty
 delete mode 100644 tests/Dockerfile.ubuntu-xenial
 delete mode 100644 tests/inventory
 delete mode 100644 tests/test.yml
```

Now, I can start reviewing the changes made to the files marked as `modified`. So, I'll first check to see which files were modified:

```bash
git status | grep modified
...
 modified:   .travis.yml
 modified:   README.md
 modified:   defaults/main.yml
 modified:   meta/main.yml
```

As we can see, only four files have been modified. So, I'll take my time and go through each of those files with whatever editor I choose to see what to keep and what to discard. I personally use VSCode for this, as it makes it really easy to discard or keep any modifications.

Once I am done with the modified files, I'll also add/commit to those.

```bash
git status
...
On branch updating-structure
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
 modified:   .travis.yml
 modified:   README.md
 modified:   meta/main.yml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
 .github/workflows/default.yml
 .gitignore
 .gitlab-ci.yml
 .yamllint
 CODE_OF_CONDUCT.md
 CONTRIBUTING.md
 CONTRIBUTORS.md
 LICENSE.md
 files/.gitkeep
 molecule/default/Dockerfile.j2
 molecule/default/INSTALL.rst
 molecule/default/molecule.yml
 molecule/shared/converge.yml
 molecule/shared/tests/test_default.py
 molecule/vagrant/INSTALL.rst
 molecule/vagrant/molecule.yml
 molecule/vagrant/prepare.yml
 playbook.yml
 requirements-dev.txt
 requirements.txt
 requirements.yml
 templates/.gitkeep
```

```bash
git commit -m "Updated files, etc. after new structure"
...
 3 files changed, 58 insertions(+), 173 deletions(-)
 rewrite .travis.yml (93%)
 rewrite README.md (87%)
```

The final step is to go ahead and add the remaining `untracked files`. As these would be new files that are part of my new desired structure, I'll also want those to be committed.

We can add all `untracked files` by:

```bash
git add .
```

```bash
git status
...
On branch updating-structure
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
 new file:   .github/workflows/default.yml
 new file:   .gitignore
 new file:   .gitlab-ci.yml
 new file:   .yamllint
 new file:   CODE_OF_CONDUCT.md
 new file:   CONTRIBUTING.md
 new file:   CONTRIBUTORS.md
 new file:   LICENSE.md
 new file:   files/.gitkeep
 new file:   molecule/default/Dockerfile.j2
 new file:   molecule/default/INSTALL.rst
 new file:   molecule/default/molecule.yml
 new file:   molecule/shared/converge.yml
 new file:   molecule/shared/tests/test_default.py
 new file:   molecule/vagrant/INSTALL.rst
 new file:   molecule/vagrant/molecule.yml
 new file:   molecule/vagrant/prepare.yml
 new file:   playbook.yml
 new file:   requirements-dev.txt
 new file:   requirements.txt
 new file:   requirements.yml
 new file:   templates/.gitkeep
```

Now, we can commit these as well:

```bash
git commit -m "New files, etc. from new structure"
...
[updating-structure 9b5a5f2] New files, etc. from new structure
 22 files changed, 431 insertions(+)
 create mode 100644 .github/workflows/default.yml
 create mode 100644 .gitignore
 create mode 100644 .gitlab-ci.yml
 create mode 100644 .yamllint
 create mode 100644 CODE_OF_CONDUCT.md
 create mode 100644 CONTRIBUTING.md
 create mode 100644 CONTRIBUTORS.md
 create mode 100644 LICENSE.md
 create mode 100644 files/.gitkeep
 create mode 100644 molecule/default/Dockerfile.j2
 create mode 100644 molecule/default/INSTALL.rst
 create mode 100644 molecule/default/molecule.yml
 create mode 100644 molecule/shared/converge.yml
 create mode 100644 molecule/shared/tests/test_default.py
 create mode 100644 molecule/vagrant/INSTALL.rst
 create mode 100644 molecule/vagrant/molecule.yml
 create mode 100644 molecule/vagrant/prepare.yml
 create mode 100644 playbook.yml
 create mode 100644 requirements-dev.txt
 create mode 100644 requirements.txt
 create mode 100644 requirements.yml
 create mode 100644 templates/.gitkeep
```

Now, I am updating my project with a new desired structure without losing anything other than what I intended. So, you can now push those changes up to the `updating-structure` branch.

Let's ensure our `git remote` is still in place before doing so:

```bash
git remote -v
...
origin git@github.com:mrlesmithjr/ansible-control-machine.git (fetch)
origin git@github.com:mrlesmithjr/ansible-control-machine.git (push)
```

Awesome! So we are good to go and can now push them up.

```bash
git push
...
Enumerating objects: 43, done.
Counting objects: 100% (43/43), done.
Delta compression using up to 8 threads
Compressing objects: 100% (29/29), done.
Writing objects: 100% (38/38), 8.15 KiB | 2.72 MiB/s, done.
Total 38 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4), completed with 2 local objects.
remote:
remote: Create a pull request for 'updating-structure' on GitHub by visiting:
remote:      https://github.com/mrlesmithjr/ansible-control-machine/pull/new/updating-structure
remote:
To github.com:mrlesmithjr/ansible-control-machine.git
 * [new branch]      updating-structure -> updating-structure
```

Once that is completed, you can start testing and resolving any issues you may find from any CI tests if enabled. In my case, I have pushed to GitHub, and I have a GitHub Actions workflow that should kick off. Remember, this is something I included in the Cookiecutter template.

Finally, once you are happy with the state of your new updating-structure branch, you can create a Pull Request to merge the changes into your master branch. I want to stress once again that we have not touched our master branch at all, so it will remain with its original structure until the Pull Request is merged.

## Conclusion

This is an excellent exercise to get the new structure in place. However, initially, it can be daunting, as you may be worried about causing issues. But if you follow these steps, you should be just fine! So, there you have it. I'd love to hear from others on how they go about these types of scenarios and your experiences, so feel free to leave feedback.

Enjoy!
