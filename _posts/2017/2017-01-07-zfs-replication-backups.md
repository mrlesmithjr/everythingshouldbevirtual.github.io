---
  title: ZFS - Replication Backups
---

I recently rebuilt my lab NAS hosts with Ubuntu 16.04 and used ZFS for
the storage pools (once again). In doing it this time around I wanted to
get a good method of leveraging ZFS snapshots and replicating them for
my backups. I ended up coming up with the following script that has been
working out quite well so far. As usual, I figured it might be
worthwhile to share this for others to leverage as well.

{% raw %}

```bash
#!/usr/bin/env bash

# ZFS Backups to remote server using replication
# Larry Smith Jr.
# @mrlesmithjr
# http://everythingshouldbevirtual.com

# Turn on verbose execution
set -x

# Source definitions
# ------------------
# Source ZFS Pool
SRCPOOL="TANK"

# Define ZFS dataset(s) to backup
# BACKUP=("Shares" "Data")
BACKUP=("Shares")

# Define snapshot name to use for most recent snapshot
CURRENTSNAP="replicate"

# Define snapshot name to use as previous snapshot
OLDSNAP="replicated"

# Define ZFS send options on source
SNDOPTS="-R -i"
# ------------------


# Destination server definitions
# ------------------------------
# Destination Pool on remote server
DSTPOOL="TANK"

# Define remote server to backup to
BACKUPSRV="nas02"

# Define dataset compression
COMPRESSION="lz4"

# Define main ZFS dataset to use as backup destination
DESTMAINDS="$DSTPOOL/Backups"

# Define dataset(s) on remote server to create under DESTMAINDS
# DESTDS=("Shares" "Data")
DESTDS=("Shares")

# Define remote server ZFS receive options
RCVOPTS="-vF"
# ------------------------------

# Ensure ZFS main dataset(s) exist on remote server
ssh $BACKUPSRV sudo zfs list | grep $DESTMAINDS
RC=$?
if [ $RC != 0 ]; then
  ssh $BACKUPSRV sudo zfs create -o compression=$COMPRESSION $DESTMAINDS
fi

# Ensure ZFS dataset(s) under DESTMAINDS exist on remote server
for DST in "${DESTDS[@]}"
do
  ssh $BACKUPSRV sudo zfs list | grep $DESTMAINDS/$DST
  RC=$?
# Create ZFS dataset(s) if they are missing on remote server
  if [ $RC != 0 ]; then
    ssh $BACKUPSRV sudo zfs create -o compression=$COMPRESSION $DESTMAINDS/$DST
  fi
done

# Execute backup
# Loop through backup dataset(s) to backup
for BACKUPFS in "${BACKUP[@]}"
do
# Check if destination backup dataset(s) exist
  ssh $BACKUPSRV sudo zfs list | grep $DESTMAINDS/$BACKUPFS
  RC=$?
  if [ $RC != 0 ]; then
# Send first initial replication of backup dataset(s) if they do not exist
    sudo zfs snapshot -r $SRCPOOL/$BACKUPFS@$CURRENTSNAP
    sudo zfs send -R $SRCPOOL/$BACKUPFS@$CURRENTSNAP | ssh $BACKUPSRV sudo zfs receive -F $DESTMAINDS/$BACKUPFS
  else
# Destroy already replicated snapshot(s)
    ssh $BACKUPSRV sudo zfs destroy -r $DESTMAINDS/$BACKUPFS@$OLDSNAP.7
    sudo zfs destroy -r $SRCPOOL/$BACKUPFS@$OLDSNAP.7
# Rename previous replicated snapshot(s) on remote server...Keeping 7 previous replicated snapshot(s)
    ssh $BACKUPSRV sudo zfs rename -r $DESTMAINDS/$BACKUPFS@$OLDSNAP.6 @$OLDSNAP.7
    ssh $BACKUPSRV sudo zfs rename -r $DESTMAINDS/$BACKUPFS@$OLDSNAP.5 @$OLDSNAP.6
    ssh $BACKUPSRV sudo zfs rename -r $DESTMAINDS/$BACKUPFS@$OLDSNAP.4 @$OLDSNAP.5
    ssh $BACKUPSRV sudo zfs rename -r $DESTMAINDS/$BACKUPFS@$OLDSNAP.3 @$OLDSNAP.4
    ssh $BACKUPSRV sudo zfs rename -r $DESTMAINDS/$BACKUPFS@$OLDSNAP.2 @$OLDSNAP.3
    ssh $BACKUPSRV sudo zfs rename -r $DESTMAINDS/$BACKUPFS@$OLDSNAP.1 @$OLDSNAP.2
    ssh $BACKUPSRV sudo zfs rename -r $DESTMAINDS/$BACKUPFS@$OLDSNAP @$OLDSNAP.1
    ssh $BACKUPSRV sudo zfs rename -r $DESTMAINDS/$BACKUPFS@$CURRENTSNAP @$OLDSNAP
# Rename previous replicated snapshot(s) locally...Keeping 7 previous replicated snapshot(s)
    sudo zfs rename -r $SRCPOOL/$BACKUPFS@$OLDSNAP.6 @$OLDSNAP.7
    sudo zfs rename -r $SRCPOOL/$BACKUPFS@$OLDSNAP.5 @$OLDSNAP.6
    sudo zfs rename -r $SRCPOOL/$BACKUPFS@$OLDSNAP.4 @$OLDSNAP.5
    sudo zfs rename -r $SRCPOOL/$BACKUPFS@$OLDSNAP.3 @$OLDSNAP.4
    sudo zfs rename -r $SRCPOOL/$BACKUPFS@$OLDSNAP.2 @$OLDSNAP.3
    sudo zfs rename -r $SRCPOOL/$BACKUPFS@$OLDSNAP.1 @$OLDSNAP.2
    sudo zfs rename -r $SRCPOOL/$BACKUPFS@$OLDSNAP @$OLDSNAP.1
    sudo zfs rename -r $SRCPOOL/$BACKUPFS@$CURRENTSNAP @$OLDSNAP
# Create new current snapshot(s)
    sudo zfs snapshot -r $SRCPOOL/$BACKUPFS@$CURRENTSNAP
# Replicate incremental snapshot(s) to remote server
    sudo zfs send $SNDOPTS $SRCPOOL/$BACKUPFS@$OLDSNAP $SRCPOOL/$BACKUPFS@$CURRENTSNAP | ssh $BACKUPSRV sudo zfs receive $RCVOPTS $DESTMAINDS/$BACKUPFS
  fi
  # ssh $BACKUPSRV sudo zfs set mountpoint=/$DESTMAINDS/$BACKUPFS $DESTMAINDS/$BACKUPFS
done
```

{% endraw %}

Enjoy!
