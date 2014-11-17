Backup Ubuntu (14.04) Server to Amazon S3
=========
A backup script that will install and setup AWS as well as create the required scripts to schedule a automatic backup every night at 1AM.

Install
---
Read, then run the script:

```bash
bash <(curl -s https://raw.githubusercontent.com/oliverfarrell/ubuntu-amazon-s3-backup/master/setup) 2>&1 | tee ~/aws_backup.log
```
**Until merged the url should be https://raw.githubusercontent.com/oliverfarrell/ubuntu-amazon-s3-backup/feature/all-in-one/setup** 

Debugging
---
The last execution of the script will be output the result to '~/aws_backup.log'. Read through it to see if you can debug the issue yourself. If not, copy the lines where the script failed into a new (GitHub Issue for us)[https://github.com/oliverfarrell/ubuntu-amazon-s3-backup/issues/new]. Or, attach the whole log file as an attachment.


## Other tidbits
Keeping every backup ever created in your S3 bucket could be costly and not particularly efficient. Amazon S3 has the ability to "expire" content in a bucket that is older than a set length of time. It can also be configured to move that content to Amazon's Glacier service if you don't wish to delete the content. For example, I expire all backups that are older than 14 days.

Have a read of the their docs to implement it on your bucket: http://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html

## Think you can do better?
This is a very early script and our first attempt at a full bash script, if you would rather try your hand at the manual version please take a look back at a previous branch **INSERT BRANCH HERE**. If you think you can do better Tweet us at [@oliverfarrell](http://twitter.com/oliverfarrell) or [@trickierstinky](http://twitter.com/trickierstinky), better yet fork and issue a pull request!
