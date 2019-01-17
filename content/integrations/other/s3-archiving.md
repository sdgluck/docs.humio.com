---
title: "S3 Archiving"
categories: ["Integration", "OtherIntegration"]
pageImage: /integrations/aws.svg
aliases: ["ref/s3"]
---

Humio supports archiving ingested logs to [Amazon S3](https://aws.amazon.com/s3/). The archived logs are then available for further processing in any external system that integrate with S3.

Archiving works by running a periodic job inside all Humio nodes which looks for new unarchived segment files. The segment files are read from disk, streamed to a S3 bucket and marked as archived in Humio.

An admin user needs to setup archiving per repository. After selecting a repository on the Humio front page the configuration page is available under Settings.

{{% notice info %}}
For slow moving data sources it can take some time before segments files are completed on disk and then made available for the archiving job. In the worst case a segment file must either contain a gigabyte of uncompressed data or 7 days must pass before it's completed. This limitation will be removed in a future version of Humio.
{{% /notice %}}

More on [segments files]({{< relref "concepts/ingest-flow" >}}) and [data sources]({{< relref "concepts/datasources" >}}).

## S3 Layout

When uploading a segment file Humio creates the S3 object key based on the tags, start date and repository name of the segment file. The resulting object key makes the archived data browseable through the S3 management console.

The following pattern is used:

`REPOSITORY/TYPE/TAG_KEY_1/TAG_VALUE_1/../TAG_KEY_N/TAG_VALUE_N/YEAR/MONTH/DAY/START_TIME-SEGMENT_ID.gz`

Read more about [Tags]({{< ref "tagging.md" >}}).

## Format

The default archiving format is [NDJSON](http://ndjson.org) and optionally raw log lines. When using NDJSON the parsed fields will be available along with the raw log line. This incurs some extra storage cost compared to using raw log lines but gives the benefit of ease of use when processing the logs in an external system.

## On-premises Setup

For an on-premises installation of Humio an [IAM](https://aws.amazon.com/iam/) user with write access to the buckets used for archiving is needed. The user must have programmatic access to S3. E.g. when adding a new user through the AWS console make sure 'programmatic access' is ticked:

![Adding user with programmatic access.](/images/s3-archiving/add_user_1.png)

Later in the process you can retrieve access key and secret key:

![Access key and secret key.](/images/s3-archiving/add_user_2.png)

 Which in Humio is needed in the following configuration:

```shell
S3_ARCHIVING_ACCESSKEY=$ACCESS_KEY
S3_ARCHIVING_SECRETKEY=$SECRET_KEY
```

The keys are used for authenticating the user against the S3 service. For more guidance on how to retrieve S3 access keys see [AWS access keys](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys). More details on [creating a new user in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html).

Configuring the user to have write access to a bucket can be done by attaching a policy to the user.


### IAM User Example Policy

The following JSON is an example policy configuration.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:AbortMultipartUpload",
                "s3:PutObjectAcl",
                "s3:ListMultipartUploadParts"
            ],
            "Resource": "arn:aws:s3:::*/*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "s3:ListBucketMultipartUploads",
            "Resource": "arn:aws:s3:::*"
        }
    ]
}
```

The policy can be used as an inline policy attached directly to the user through the AWS console:

![Attach inline policy.](/images/s3-archiving/add_user_3.png)

## Cloud Setup

Enabling Humio Cloud to write to your S3 bucket means setting up AWS cross-account access. Follow these steps:

1. Log into the AWS console and navigate to the S3 service page.
2. Click on the name of the bucket where archived logs should be written.
3. Click on Permissions.
4. Click Add User.
5. Enter the canonical ID for Humio( `f2631ff87719416ac74f8d9d9a88b3e3b67dc4e7b1108902199dea13da892780` ) and check the Object Access Read and Write permission check boxes and click Save.
6. In Humio go to the repository you want to archive and select S3 Archiving
under Settings. Configure by giving bucket name, region etc. and Save.

## Tag Grouping

If tag grouping is defined for a repository the segment files will by split by
each unique combination of tags present in a file. This results in a file in S3
per each unique combination of tags. The same layout pattern is used as in the
normal case. The reason for doing this is to make it easier for a human operator
to determine whether a log file is relevant.

{{% notice note %}}
S3 archiving is only available in version >= 1.1.27.
{{% /notice %}}



