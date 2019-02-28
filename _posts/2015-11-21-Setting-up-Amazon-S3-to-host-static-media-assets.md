---
layout: post
title: Setting up Amazon S3 to host static media assets
tags: [DevOps]
---

I have a server with many gigabytes of static images that are starting to consume all of its available disk space.

Unfortunately, the server is underpowered and not regularly backed up, which means that requests are not served as quickly as they could be and a system failure means I could lose all of this data. The server is also presently not configured to support SSL.

I was faced with the dilemma of either upgrading the server to a larger disk or moving the static content to a more appropriate home. Upgrading does not provide me with a backup solution, doesn't scale, and would not improve performance.

Amazon S3 is a very durable and low cost file storage solution that automatically replicates the data across multiple regions and would also serve the content faster. It even gives me SSL as a bonus.

I work with various AWS services in my day job, but there's a difference between working with something that's already been built and building it from scratch. I've had limited opportunity to build something from scratch on the AWS platform and this is a perfect opportunity learn a few new things.

The content is user-uploaded and always growing and I'd eventually like to have users upload directly to S3, but for the moment, my need is to migrate all of the existing data. I achieved this by writing a short python script that uploads each file and updates a flag in its corresponding database record to indicate that it is now hosted on S3. The web server reads the records from the database to identify the content to present and references either the image on S3 or the one hosted locally. For the short term, I'll schedule my migration script to run periodically until I change the server to have users submit their content straight to S3.

I got started by configuring a new AWS account. After this, I set up an S3 bucket and IAM user that exists for the sole purpose of populating the bucket with files. I got stuck for a while writing my migration script because I kept getting Access Denied errors, and this ultimately was due to me creating a policy that permitted my IAM user to perform PUT requests but not list all of the available buckets. I eventually created an IAM policy as shown below. I linked this to an IAM user created specifically for the purpose of uploading files on a specific S3 bucket.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1448143511000",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::bucketname",
                "arn:aws:s3:::bucketname/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "arn:aws:s3:::*"
        }
    ]
}
```

This worked great to upload the files, and the next step was to open up access to GET requests. While I was at it, I wanted to restrict access only to requests referred from my site via the HTTP Referer header or containing no referrer at all. This is possible by creating a policy on the bucket itself as follows.

```json
{
	"Version": "2012-10-17",
	"Id": "Permit GET requests with HTTP Referer validation",
	"Statement": [
		{
			"Sid": "Allow get requests originating from example.com.",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::bucketname/*",
			"Condition": {
				"StringLike": {
					"aws:Referer": [
						"http://*.example.com/*",
						"http://example.com/*"
					]
				}
			}
		},
		{
			"Sid": "Allow get requests without a referrer.",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::bucketname/*",
			"Condition": {
				"Null": {
					"aws:Referer": true
				}
			}
		}
	]
}
```
