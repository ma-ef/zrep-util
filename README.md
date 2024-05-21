# zrep-util
Shell script to simplify administration of backups by utilizing zrep


## Usage/Examples

```shell
# we create a new dataset test to show the functionality of the tool
# on the source host: destroy data set if it exists and recreate with
# read permissions for zsbackup user
POOL="data0"
source# zfs destroy -r "$POOL/test"; zfs create "$POOL/test" && zfs allow -u zsbackup send,hold,snapshot,userprop data0/test

# on the destination host:
POOL="remote_backup"
destination# zfs destroy -r /test_parent; zfs create -o canmount=off remote_backup/test_parent && zfs allow -u zrbackup create,mount,receive,userprop remote_backup/test_parent && zfs create remote_backup/test_parent/data0
```
Now create the configuration file test.cfg for the script (usually on the destination host) with this content
```shell
####################
## CONFIGURATION  ##
####################

# pools to backup
pools=(data0/test)
# login at server of backup source
# specify in the same manner as you would specify for ssh
# and make sure that you are able to login via ssh and public key
# for local use, please specify user@localhost
# if @localhost is specified ssh is not used but commands are run locally
# note it is only possible to switch to a different user when you are root
source="zsbackup@sourceserver"
# where is zrep located
source_zrep_command="/usr/local/bin/zrep"
# login at server of backup destination
# specify in the same manner as you would specify for ssh
# and make sure that you are able to login via ssh and public key
# for local use, please specify user@localhost
# if @localhost is specified ssh is not used but commands are run locally
# note it is only possible to switch to a different user when you are root
destination=zrbackup@localhost
# parent dataset at backup destination (must exist)
destination_parent_dataset=remote_backup/test_parent
# where is zrep located
destination_zrep_command="/usr/bin/zrep"
# log file
logfile=/tmp/zrep-util.log
# use custom zrep tag
# normally the tag zrep is used to store data in the zfs dataset
# if the dataset is part of a dataset structure and both parent and child are synced
# this can lead to problems e.g. because setting the parent will also set the child
# in this case iz is better to let custom tags be generated which are unique to the dataset
# e.g. if dataset is home/user1 => zrep_tag is zrep_home_user1
use_custom_tag=true
```
Now execute the zrep-util for initialization:
```shell
zrep-util -c test.cfg initialize
```
To refresh the backup call zrep-util like this:
```shell
zrep-util -c test.cfg backup
```
