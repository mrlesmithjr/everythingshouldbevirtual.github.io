---
  title: Back to Sophos/Astaro
  date: 2013-01-15 09:05:15
---

Well after a few months of running my own rolled version of a UTM using
Ubuntu I have officially gone back to Sophos/Astaro for the time being. 
There are several reasons for this with the main reason being that I
absolutely love the interface and reporting of Sophos/Astaro. I am also
doing some testing on HTTPS web filtering after a few mishaps of some
google images searches showing inappropriate images for small children. 
I know I can do this with my own rolled UTM, but I am sure it will be
much harder. This same issue also applies to corporate web filtering
because once logged into google your google images searches and normal
searches are all secure which bypasses traditional HTTP web filtering. 
There are many complications with implementing HTTPS web filtering
though because your firewall will act as a man in the middle which will
break all HTTPS communications. To implement this you will have to
deploy the SSL cert for your firewall to all clients that you want to
implement HTTPS web filtering on. For Windows clients this is a non
issue because you can deploy the cert using a GPO if you are using
Active Directory. This method will add the cert automatically for IE
and Google Chrome, but Firefox and other browsers will need to have the
cert manually imported. The first few things I noticed before disabling
HTTPS web filtering is that Dropbox and Google Drive stopped syncing
because the HTTPS web filter is obviously intercepting that traffic as
well. So exceptions would need to be made for those two services based
on destination addresses. As you can figure from this is that it would
take a lot of effort to keep up with these as they can potentially
change a lot. So I will be doing some additional testing with that. 
One way is to also implement the Proxy Server Agent on the Sophos/Astaro
and have dropbox use the proxy, but that doesn't work for Google
Drive. More testing to be done on that. Again the biggest downfall for
me which would also apply to others that do extensive testing at home is
the 50 IP limit for the free home version of Sophos/Astaro. So I think
I would just fire up PFsense again during those times that I may be
doing lots of testing or fence off the testing vms so they do not
traverse the firewall using up IPs counting against the 50 IP limit.

Stay tuned as I will be posting more info on this in the upcoming weeks.
