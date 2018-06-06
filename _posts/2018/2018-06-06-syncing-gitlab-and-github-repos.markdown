---
title: "Syncing GitLab and GitHub Repos"
date: "2018-06-06 00:22"
categories:
  - Git
tags:
  - GitHub
  - GitLab
---

## Purpose

With all of the latest panic of Microsoft buying GitHub, and being that I have
been a user of both GitHub and GitLab for years I wanted to prepare myself in
the off chance that I may decide to abandon GitHub as well. I have definitely
invested a huge amount of time and effort into my GitHub repos so that move may
never happen but just in case it does I wanted to get this together and start
planning. GitLab definitely has a very easy way to import only certain GitHub repos
or you can import all of them with one click of a button (which I did) and for
me that was nearly 400 repos so it took some time but was very simple. One thing
to note is that when you do import your GitHub repos into GitLab, the repos will
by default be setup as `private` repos (a very safe move).

In this scenario we will be keeping GitHub as our primary source with GitLab being
setup to sync (push) only (for now!).

> NOTE: The repos must already exist on GitHub and GitLab. You can easily import
> the GitHub repo into GitLab using their import functionality.

## Preparing

Check current Git Remotes:

```bash
git remote -v
origin	https://github.com/mrlesmithjr/Ansible.git (fetch)
origin	https://github.com/mrlesmithjr/Ansible.git (push)
```

In my scenario I will be switching to using SSH rather than HTTPS for simplicity
of various GitLab accounts that I have which must remain seperate. So you must
first add your SSH key to each GitHub and GitLab.

Remove the current `origin` remote:

```bash
git remote remove origin
```

> NOTE: You do not need to do this if you are not changing either SSH or HTTPS.
> But you can also just as easy switch this without removing `origin`:
>
> ```bash
> git remote set-url origin git@github.com:mrlesmithjr/Ansible.git
> ```

Add new remote `github`:

```bash
git remote add github git@github.com:mrlesmithjr/Ansible.git
```

Add new remote `gitlab`:

```bash
git remote add gitlab git@gitlab.com:mrlesmithjr/Ansible.git
```

Now check your current Git remotes:

```bash
git remote -v
github	git@github.com:mrlesmithjr/Ansible.git (fetch)
github	git@github.com:mrlesmithjr/Ansible.git (push)
gitlab	git@gitlab.com:mrlesmithjr/Ansible.git (fetch)
gitlab	git@gitlab.com:mrlesmithjr/Ansible.git (push)
```

Now let's add back our `origin` remote:

```bash
git remote add origin git@github.com:mrlesmithjr/Ansible.git
```

Now we will add gitlab as an additional `origin` remote (for push only):

```bash
git remote set-url --add --push origin git@gitlab.com:mrlesmithjr/Ansible.git
```

And if we now check our Git remotes:

```bash
git remote -v
github	git@github.com:mrlesmithjr/Ansible.git (fetch)
github	git@github.com:mrlesmithjr/Ansible.git (push)
gitlab	git@gitlab.com:mrlesmithjr/Ansible.git (fetch)
gitlab	git@gitlab.com:mrlesmithjr/Ansible.git (push)
origin	git@github.com:mrlesmithjr/Ansible.git (fetch)
origin	git@github.com:mrlesmithjr/Ansible.git (push)
origin	git@gitlab.com:mrlesmithjr/Ansible.git (push)
```

And finally we must set our upstream remote because we removed our original `origin`
remote:

```bash
git push --set-upstream origin master
Branch 'master' set up to track remote branch 'master' from 'origin'.
Everything up-to-date
Branch 'master' set up to track remote branch 'master' from 'origin'.
Everything up-to-date
```

And as you can see from the above we have now pushed to both GitHub and GitLab.
Obviously there were not any changes but from here on out when you make changes,
commit, and push the changes, both GitHub and GitLab will be in sync.

You may have also noticed we added remotes named `github` and `gitlab` individually
when we could have simply just added the new `origin` remote. This is only to
give us the ability to specifically fetch/push to the respective remote if needed.

## Conclusion

This was mainly put together for my own personal reference but maybe it might
also benefit others at the same time. So as always, ENJOY!

> NOTE: Don't just jump on the bandwagon of [#movingtogitlab](https://twitter.com/hashtag/movingtogitlab) but definitely
> prepare yourself if it makes sense and keep OSS alive.
