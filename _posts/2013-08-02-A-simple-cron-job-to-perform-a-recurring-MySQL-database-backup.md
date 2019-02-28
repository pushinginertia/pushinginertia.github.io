---
layout: post
title: A simple cron job to perform a recurring MySQL database backup
tags: [DevOps]
---

I've written a simple script that performs a MySQL database dump and can be defined as a cron job to run at regular intervals.

This script will dump the entire database to a file and gzip it, while also creating a log of its activities.

Before starting, identify which user the script should run as. I chose to run this as the root user because the script contains the database's password in it and only the root user should have access to this information.

First, create a directory for the script to log to. This should ideally be a subdirectory of `/var/log`, such as `/var/log/db-backup`. Also give it ownership and write permission to the user executing the script. If you're running the script as root, the chown step is not necessary.

```sh
$ mkdir -p /var/log/db-backup
$ chown root /var/log/db-backup
```

Second, create a directory for the script to write its backup files to. I chose the directory `/root/db-backup`.

```sh
$ mkdir -p /root/db-backup
```

Third, the script.  Create a new file `/root/cron/db-backup.sh` and give both the file and its directory (`/root/cron`) permission 700 so that only the executing user can read/write/execute the script.

`/root/cron/db-backup.sh`

```sh
# set your database and password here
DB=<database>
PW=<password>
DATE=`date +"%Y-%m-%d.%H%M"`
FN=/root/db-backup/db-backup.${DATE}.sql;
 
echo `date +"%Y-%m-%d %T"` - Creating database backup in file: $FN >> /var/log/db-backup/db-backup.log
mysqldump --routines --user=${DB} --password=${PW} --result-file=${FN} --opt ${DB}
  
echo `date +"%Y-%m-%d %T"` - Gzipping file: $FN >> /var/log/hsb/db-backup.log
gzip -f ${FN}
```

Finally, add the script to your crontab file by executing the following command to edit the crontab file.

```sh
$ crontab -e
```

Then add a line to execute the script. This example line runs the script at 21:45 daily.

```sh
45 21 * * * /root/cron/db-backup.sh
```

The following command can be used to import the dump file into a new database. This assumes that the data exported in the above steps uses UTF-8 character encoding.

```sh
$ mysql -u <USERNAME> --password=<PASSWORD> \
  --default_character_set utf8 <DATABASE> < <FILENAME>
```
