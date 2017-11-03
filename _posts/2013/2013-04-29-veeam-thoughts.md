---
  title: Veeam thoughts
  date: 2013-04-29 16:22:35
---

I have now been using [Veeam](http://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup "http\://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup") Backup and Replication for a few months now and have to say it
is a really amazing solution. During this time my thoughts along the way have
changed in several ways. What I mean by that is that just using [Veeam](http://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup "http\://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup") has changed my view on using several other technologies. One
major change of view is that I have become to really appreciate what Windows
Server 2012 brings to the table especially around storage. When I originally
started using [Veeam](http://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup "http\://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup") I was a firm believer in stating that I would never rely on a
Windows OS to manage a huge amount of data (Regardless of what I was hearing from
others including several Vendors :), Not dropping any names, but you know who
you are), I don't mean just a few GB's or a few TB's, but rather more like
50TB+. I have spent a good bit of time just configuring and testing the use of
a Windows Server 2012 server being used as a backup repository. One of the most
impressive things using this solution is the use of post-process deduplication.
Windows Server 2012 has native built in deduplication which can be turned on and
enabled for the volume(s) that are being used for
[Veeam](http://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup "http\://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup") backup repositories. What this buys you is the fact that you
can utilize [Veeam](http://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup "http\://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup") in-line deduplication up front and then reap the benefits of a
post-process dedupe schedule to run after 1-day (configurable) of storing the data.
In doing this you can store an enormous number or restore points including weekly
synthetic-full backups. You will in effect need to have enough storage available
to contain the amount equal to two full backup sets plus 10-15% change data for
each vm you will be backing up. At the end of the process the on disk storage
will equal the amount of one full backup set. For example I have been backing
up a 1.5TB vm for about 6 weeks now which includes a weekly synthetic-full
backup, so if I was not utilizing any type of deduplication I would basically
need a 10TB volume to contain these backups plus any incremental backup sets.
Instead I have allocated a 4TB volume for use and at the end of the post-process
deduplication the on disk size is about 2.5TB. This number is higher
because I also have some other vm's that are being backed up to this
same repository. But what I have at this point is 6 full backup recovery
points as well as any incremental recovery points along the way, very
impressive. :) I am also leaning towards using a physical backup server
instead of a virtual backup server. This could be the same server that
is used as the backup repository. The reason for this is obviously
because of the off chance that you were to lose your host(s) for some
unknown reason. I know the chances of that are slim, but! :) At the very
least however if you were to use a virtual backup server I would
definitely recommend not storing your backup repository within a VMware
datastore using a traditional vmdk strategy, but rather use an in guest
iSCSI solution pointing to your storage NAS/SAN. This way you can still
get access to the volumes which contain your backups in case of a
datastore issue. You will be able to quickly stand up a new backup
server and then repoint your iSCSI volumes too the new server which will
be formatted as NTFS. I would also recommend changing the location of
the backup server config information to be stored on these volumes as
well. Then you can import the config for your backup server. This works
really well as I have tested this too. You can of course use Fiber LUNS
instead of iSCSI too if that fits your environment better. The point is
there are numerous ways to configure your environment to satisfy your
requirements. However I feel that using a solution that will also allow
you to do post-process deduplication would be your best bet to keep your
storage needs to a minimum. In the end no matter which way you go I am
positive you will be happy with your
[Veeam](http://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup "http\://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup") implementation.

For a previous post which shows my initial thoughts on setting up a
Veeam solution go [here](https://everythingshouldbevirtual.com/veeam-backup-and-replication-to-nexenta-nfs "http\://everythingshouldbevirtual.com/veeam-backup-and-replication-to-nexenta-nfs").

For a design scenario example go [here](https://everythingshouldbevirtual.com/veeam-br-and-hp-3par-brainstorming "http\://everythingshouldbevirtual.com/veeam-br-and-hp-3par-brainstorming").

Get your copy of Veeam today [here](http://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup "http\://www.veeam.com/cloud-backup-vmware-hyper-v.html?utm_source=everythingshould&utm_medium=banner&utm_campaign=cloudbackup").

Enjoy!

I encourage everyone to share your thoughts and examples here in the
comments. I would be interested as I am sure everyone else would be too
to see how others have implemented a solution.
