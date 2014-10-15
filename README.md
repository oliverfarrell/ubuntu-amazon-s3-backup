Ubuntu (14.04) Amazon S3 Backup
=========

I needed a script that would backup all the sites (and later databases) hosted on my DigitalOcean[http://digitalocean.com] server running Ubuntu Linux 14.04 and push the backups to an Amazon S3 bucket.

The following outlines how to setup such backups and includes some useful tid-bits that are worth implementing on your S3 bucket.

## Prerequisites
* s3cmd[http://s3tools.org/] (http://s3tools.org/s3cmd) – ```sudo apt-get install s3cmd``` then run ```s3cmd --configure``` to link your Amazon S3 account.

## Step one
Firstly we need to create a directory where the backups and the backup script will live. This can be anywhere, but I've put it in my root folder.

```mkdir -p ~/backup```

Then we need to create a directory where the files we're backing up will be stored before uploading to S3. This just makes managing multiple backup types a little easier.

```mkdir -p ~/backup/sites```

## Step two
Create a new file in your backup folder that will contain the backup script and be called by our cronjob later on.

```vi  ~/backup/run```

Now paste the following into the file. **Note::** variables are indicated with [square brackets].

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

Save and exit.

You can test whether the script works by running ```~/backup/run```

## Step three
Now we'll want to create a cronjob so this script runs automatically at a given interval. I wanted this to run at 1am every day of the week. Start by typing the following and pressing return.

```crontab -e```

Then paste the following into the file.

```
MAILTO=[email@address.com]
0 1 * * * ~/backup/run
```

The ```MAILTO``` isn't necesary but I like to know when the script has been successful or not.

## Other tidbits
Keeping every backup ever created in your S3 could be costly and not particularly efficient. Amazon S3 as the ability to "expire" content in a bucket that is older than a set length of time. It can also be configured to move that content to Amazon's Glacier service if you don't wish to delete the content.

Have a read of the their docs to implement it on your bucket: http://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html
