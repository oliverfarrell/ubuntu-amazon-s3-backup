Backup Ubuntu (14.04) Server to Amazon S3
=========

I needed a script that would backup all the sites and databases hosted on my web server running Ubuntu Linux 14.04 and push the backups to an Amazon S3 bucket.

The following outlines how to setup such backups and includes some useful tid-bits that are worth implementing on your S3 bucket.

## Prerequisites
* [s3cmd](http://s3tools.org/) – ```sudo apt-get install s3cmd``` then run ```s3cmd --configure``` to link your Amazon S3 account.

## Step one
Firstly we need to create a directory where the backups and the backup script will live. This can be anywhere, but I've put it in my root folder.

```mkdir -p ~/backup```

Then we need to create a directory where the files we're backing up will be stored before uploading to S3. This just makes managing multiple backup types a little easier.

```mkdir -p ~/backup/sites```

## Step two
Create a new file in your backup folder that will contain the backup script and be called by our cronjob later on.

```vi  ~/backup/run```

Now paste the following into the file. **Note:** variables are indicated with [square brackets].

```
#!/bin/sh

# grab all sites and create a tar.gz archive
tar -cvpzf ~/backup/sites/`date +%d-%m-%Y`.tar.gz /srv/users/[user]/apps # or any other directory

# create a folder where we can dump all the raw .sql files
mkdir -p ~/backup/databases/raw/ # this is a temp directory, it'll be deleted later

# grab all mysql databases
~/backup/mysql # a python script

# grab all raw .sql files and create a tar.gz archive
tar -zcvf ~/backup/databases/$(date '+%d-%m-%Y').tar.gz ~/backup/databases/raw/

# delete the /raw/ directory once archived
rm -rf ~/backup/databases/raw/

# push the files to S3
s3cmd put --recursive ~/backup/sites s3://[bucket-name]/sub-folder/
s3cmd put --recursive ~/backup/databases s3://[bucket-name]/sub-folder/

# delete the contents of the other directories as we don't need to store them once backed up
rm -rf ~/backup/sites/*
rm -rf ~/backup/databases/*
```

Save and exit.

You can test whether the script works by running ```~/backup/run```

## Step three
You'll see reference in the above script to a ```~/backup/mysql```. This pretty much does what is says on the tin – creates a backup of all your MySQL databases and stores them in a directory on the server. It looks like this:

```
#!/usr/bin/env python
import os
import time
username = '[user]'
password = '[password]'
hostname = 'localhost'
filestamp = time.strftime('%Y%m%d')
database_list_command = "mysql -u%s -p%s -h%s --silent -N -e 'show databases'" % (username, password, hostname)
for database in os.popen(database_list_command).readlines():
    database = database.strip()
    if database == 'information_schema' or database == 'performance_schema' or database == 'mysql':
        continue
    filename = "~/backup/databases/raw/%s-%s.sql" % (database, filestamp)
    print "Backing up %s" % filename
    os.popen("mysqldump -u%s -p%s -h%s -e --opt -c %s | gzip -c -9 > %s.gz" % (username, password, hostname, database, filename))
    print (".. done")
```

**Note:** I happened across this script whilst researching how this could be done (sorry, can't rememeber where – I'll link it up if I find it). I am aware that there are much smarter ways of doing this, I just couldn't get any of them to work with my setup.

## Step four
Now we'll want to create a cronjob so this script runs automatically at a given interval. I wanted this to run at 1am every day of the week. Start by typing the following and pressing return.

```crontab -e```

Then paste the following into the file.

```
MAILTO=[email@address.com]
0 1 * * * ~/backup/run
```

The ```MAILTO``` isn't necesary but I like to know when the script has been successful or not. Save and exit.

## Other tidbits
Keeping every backup ever created in your S3 bucket could be costly and not particularly efficient. Amazon S3 has the ability to "expire" content in a bucket that is older than a set length of time. It can also be configured to move that content to Amazon's Glacier service if you don't wish to delete the content. For example, I expire all backups that are older than 14 days.

Have a read of the their docs to implement it on your bucket: http://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html

## Think you can do better?
If you think you could improve this – and I've not doubt that you could – let me know. Tweet me [@oliverfarrell](http://twitter.com/oliverfarrell).
