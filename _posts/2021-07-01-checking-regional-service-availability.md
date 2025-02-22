---
layout: post
title: Checking regional service availability with the AWS CLI
date: '2021-07-01T09:52:00.000+01:00'
author: Craig Watcham
tags:
- SSM
- CLI
- AWS
modified_time: '2021-07-01T09:52:54.661+01:00'
---

Using [System Manager public parameters][ssm-params] a quick CLI one liner for checking if an AWS service is available in a region:

```
aws ssm get-parameters-by-path \
--path /aws/service/global-infrastructure/services/[SERVICE]/regions/[REGION] \
--output text
```

As an example, checking if Athena is available in ap-northeast-3:

```
aws ssm get-parameters-by-path --path /aws/service/global-infrastructure/services/athena/regions/ap-northeast-3 --output text
```

An empty result indicates the service is not available. The list of service names can be retrieved as below (from the documentation linked above):
```
aws ssm get-parameters-by-path --path /aws/service/global-infrastructure/services --query 'Parameters[].Name | sort(@)'
```

Similarly the list of regions can be found using:
```
aws ssm get-parameters-by-path --path /aws/service/global-infrastructure/regions --query 'Parameters[].Name'
```

[ssm-params]: https://docs.aws.amazon.com/systems-manager/latest/userguide/parameter-store-public-parameters-global-infrastructure.html