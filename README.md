Ubuntu (14.04) Amazon S3 Backup
=========

A bash script to backup any folder/s on a Ubuntu (14.04) Linux server and upload the Amazon S3.

## Prerequisites
* s3cmd: http://s3tools.org/s3cmd

## Step one
Firstly we need to create a directory for our backups and the backup script to live. This can be anywhere, but I've put it in the root folder.

```mkdir -p ~/backup```

Then we need to create a directory where the files we're backing up will be stored before uploading to S3.

```mkdir -p ~/backup/sites```

## Step two

Coming soon.

## Step three

```
#!/bin/sh

# grab all sites and create a tar.gz archive
tar -cvpzf ~/backup/sites/`date +%d-%m-%Y`.tar.gz /srv/users/[user]/apps

# create a folder where we can dump all the raw .sql files
mkdir -p ~/backup/databases/raw/ # this is a temp directory, it'll be deleted later

# grab all mysql databases
~/backup/databases # a python script

# grab all raw .sql files and create a tar.gz archive
tar -zcvf ~/backup/databases/$(date '+%d-%m-%Y').tar.gz ~/backup/databases/raw/

# delete the /raw/ directory once archived
rm -rf ~/backup/databases/raw/

# push the files to S3
s3cmd put --recursive ~/backup/sites s3://[bucket-name]/sub-folder/
s3cmd put --recursive ~/backup/databases s3://[bucket-name]/sub-folder/

# delete the contents of the other directories â€“ we don't need to store them once backed up
rm -rf ~/backup/sites/*
rm -rf ~/backup/databases/*
```
