---
layout: post
title: Using s3cmd to synchronize local files against an S3 bucket
tags: [DevOps]
excerpt: How to install and test ntpd on CentOS
---

I use the following command to synchronize a local directory in a web project containing static resources with an S3 bucket.

```sh
s3cmd \
 --ssl \
 --skip-existing \
 --recursive \
 --delete-removed \
 --no-preserve \
 --guess-mime-type \
 --add-header=Expires:2037-12-31T00:00:00Z \
 --add-header=Cache-Control:max-age=155520000,public \
 --encoding=UTF-8 \
 sync ./src/main/webapp/static/ s3://bucketname
 ```

HTTP response headers are defined per-file rather than per-bucket, and therefore any desired response headers must be created at the time of upload.

This command does the following:

* Forces SSL communication (although this is the default).
* Does not upload files that already exist with the same hash.
* Recurses subdirectories.
* Deletes files removed locally from the S3 bucket.
* File system attributes are not stored in the file’s metadata on S3.
* MIME type is usually set correctly for well-known file extensions.
* `Expires` HTTP response header is set well in the future to prevent cache expiration.
* `Cache-Control` HTTP response header is set to approximately 5 years to prevent cache expiration.
* UTF-8 encoding is forced for text files.
* Note that the local directory must contain a trailing slash or synchronization will not work correctly and s3cmd will not recognize existing files.

