---
  title: Server 2012 AD Upgrade Notes
  date: 2013-07-25 19:34:55
---

So I am in the process of going through an Active Directory 2008 R2
upgrade to Server 2012 which has been going very smooth so far. So far I
have prepped the domain and forest and added two new Server 2012 Domain
Controllers and transferred the FSMO roles from the Windows 2008 R2
Domain controller. Migrated DHCP pools and created failover pools in a
load balanced mode (default) which is cool. Hard to believe it has taken
Microsoft this long to get something like this to work. :( The next
thing on my list was to get rid of the Windows 2008 R2 WSUS server which
is deployed via a GPO out to all clients and get all clients reporting
to the new Server 2012 WSUS server. Well this did not work initially
because the GPO was using <http://wsusserver> for checking updates and
reporting. For a day and a half nothing was reporting at all. Come to
find out (RTFM) WSUS 4.0 uses port http (8530) and https (8531) so the
GPO needed to be updated to <http://wsusserver:8530> and now all clients
are starting to populate into the new WSUS server. Now onto the next bit
of additional testing and implementation phases.

I will be updating this post with additional notes and such as I go
through this upgrade process. So stay tuned.

> Update: 07/31/2013 - All has been good so far. I actually just
> demoted my Windows 2008 R2 domain controller in preparation of
> decommissioning it.

Enjoy!
