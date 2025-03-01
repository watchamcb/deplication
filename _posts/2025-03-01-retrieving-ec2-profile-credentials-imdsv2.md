---
layout: post
title: Retrieving EC2 instance profile credentials with IMDSv2
date: '2025-03-01T09:13:00.000+00:00'
author: Craig Watcham
tags:
- IMDSv2
- AWS CLI
modified_time: '2025-03-01T09:13:00.000+00:00'
---

Quick example of fetching EC2 instance profile credentials using IMDSv2.
 
```
TOKEN=`curl --no-progress-meter -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
curl --no-progress-meter -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/identity-credentials/ec2/security-credentials/ec2-instance
```
\
The response should look something like this:
```
{
  "Code" : "Success",
  "LastUpdated" : "2025-02-21T12:13:35Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAZKEYKEYKEY",
  "SecretAccessKey" : "SECRETSECRETSECRET",
  "Token" : "AVERYLONGTOKENSTRING",
  "Expiration" : "2025-02-21T18:37:11Z"
}
```
\
To extract the credentials into environment variables for use with the AWS CLI:
```
CREDS=`curl --no-progress-meter -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/identity-credentials/ec2/security-credentials/ec2-instance`
export AWS_ACCESS_KEY_ID=`echo $CREDS | jq -r .AccessKeyId`
export AWS_SECRET_ACCESS_KEY=`echo $CREDS | jq -r .SecretAccessKey`
export AWS_SESSION_TOKEN=`echo $CREDS | jq -r .Token`
```
\
After which you can use the CLI as usual until the credentials expire and are rotated.  

```
aws sts get-caller-identity
{
    "UserId": "112233445566:aws:ec2-instance:i-1122334455667788",
    "Account": "112233445566",
    "Arn": "arn:aws:sts::112233445566:assumed-role/aws:ec2-instance/i-1122334455667788"
}
```