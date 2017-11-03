---
  title: vCenter SSO AD Gotcha
  date: 2013-08-01 19:01:23
---

So this past week I have been upgrading my home lab domain from AD 2008
R2 to AD 2012 and so far it has gone without a hitch (read
[here](http://everythingshouldbevirtual.com/server-2012-ad-upgrade-notes "http\://everythingshouldbevirtual.com/server-2012-ad-upgrade-notes")
on this). Well until today that is. I was attempting to login to vCenter
using the viclient and the screenshot below is what I was presented
with.

![13-21-32](../../assets/13-21-32-300x270.png)

"A general system error occurred: Authorize Exception". Well that
doesn't look good. :(

So a good Google search returned [this](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1015639 "http\://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1015639")
KB article about this. Which got me looking at my SSO configuration. So I
logged into my vCenter server and started looking through the event logs
and found this next.

![14-01-50](../../assets/14-01-50-300x194.png)

OK so my vcenter domain account(s) added into permissions are not
resolving correctly now. Which got me thinking. I demoted my Windows
2008 R2 domain controller last night could this really be the cause of
my issue? So I tested to make sure that I could launch the viclient fine
using the local administrator account. Success. So I now know that
vCenter is working but domain logins are not working. So I quickly
launched the vCenter web gui and went into my SSO configuration and look
what I found staring at me. I never even thought about this of course.

![13-25-41](../../assets/13-25-41-300x65.png)

Sure enough, my active directory identity source was still pointing to
the domain controller that I decommissioned last night. Well we can fix
this easy enough. Remove the old identity source and create a new one
pointing to my two new Server 2012 domain controllers.

![13-27-07](../../assets/13-27-07-263x300.png)

Test the connection and success. :)

Now back to the viclient I need to go look at the granted permissions
for my domain user(s).

![14-03-35](../../assets/14-03-35-300x63.png)

Sure enough they are all gone.

So let's add them all back.

![14-03-48](../../assets/14-03-48-300x241.png)

![14-03-56](../../assets/14-03-56-300x233.png)

![14-04-41](../../assets/14-04-41-300x125.png)

Ah ha! The accounts are now added back. So now let's check and make
sure that I can log into vCenter again. Success! All done!

So keep this in mind if you are going through a scenario like this.
Especially if you are decommissioning domain controllers. You will need
to make sure beforehand that you add your new domain controller(s) into
SSO as an identity source and remove the old domain controller(s) so you
do not get into a bind here like I did.

Enjoy!
