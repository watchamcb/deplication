---
layout: post
title: Reducing VPC interface endpoint costs in dev/test environments
date: '2021-08-08T10:44:00.000+01:00'
author: Craig Watcham
tags:
- VPC Interface Endpoint
- Athena
- Cost saving
- AWS
modified_time: '2021-08-08T10:44:48.151+01:00'
---

Two quick tips for potentially reducing VPC interface endpoint costs.

Firstly an Athena query for finding unused VPC interface endpoints in cost and usage report data. This requires you to have [cost and usage reporting][cur] enabled (you should already) and configured for querying in Athena, see [this guide][cur-athena]. The query searches for all VPC interface endpoints that have not accrued any data transfer charges during the query period. VPC endpoints incur both hourly and data transfer [charges][pl-pricing], if you are not transferring any data it is very likely the endpoint is not being used and can be deleted. This will save around $7 a month ($0.01 x 24hours x 30 days) in each availability zone configured for the unused endpoint (by default all AZs). Annual cost saving potential of around $260 for each (3 AZ) endpoint removed in each account.

```
select 
  line_item_usage_account_id, line_item_resource_id
from 
  cur
where 
  line_item_usage_type like '%VpcEndpoint-Hour%'
  and year='2021'
  and month in ('7','07')
  and line_item_resource_id not in (
    select distinct(line_item_resource_id)
    from cur
    where line_item_usage_type like '%VpcEndpoint-Byte%'
    and year='2021'
    and month in ('7','07')
  )
group by line_item_usage_account_id, line_item_resource_id
```

The second approach is only recommended for dev/test environments where availability zone fault tolerance is not a requirement. When you create a VPC interface endpoint an ENI is created in each subnet (availability zone) unless you explicitly deselect the subnets. This is a best practice for interface endpoints as it provides continued availability in the case of an AZ failure - as long as you are using the (default) regional interface endpoint DNS name with [private DNS][private-dns]. When using private DNS the regional endpoint DNS name, ec2.eu-west-1.amazonaws.com for example, is resolved to the private IP of the VPC interface endpoint ENI rather than the public IP of the service. In the case where one AZ is not reachable the request will be routed to an interface endpoint ENI in a different AZ, allowing your client applications to continue functioning. The down side of this is that you will incur intra-region data transfer charges (of $0.01/GB) for sending requests to the ENI in a different AZ. The potential cost saving opportunity is to remove this availability mechanism to reduce the hourly interface endpoint charges.

As long as the data transferred to and from the interface endpoint is around 1GB/hour it is cheaper to configure the VPC interface endpoint to only be available in one (or two) availability zones. This makes sense for any API specific endpoints (EC2 for example) but may not make sense for high throughput data intensive endpoints (such as S3/Kinesis/Cloudwatch Logs). The cost and usage report data can once again be used to help identify interface endpoints where the data transfer is low enough to justify exposing the endpoint in only one availability zone. Note that the endpoint data charges are aggregated across all ENIs so the query will not correctly identify cases where traffic is not more-or-less equally balanced across AZs. The query assumes that the cost and usage report is configured for hourly granularity, the line_item_usage_amount would need to be modified for daily/monthly reports and would be less accurate. Annual cost saving potential would depend on data transfer and number of AZs provisioned but should be around $175 for API endpoints reduced from 3 AZs to 1 AZ.

```
select 
  line_item_usage_account_id, line_item_resource_id
from 
  cur
where 
  line_item_usage_type like '%VpcEndpoint-Byte%'
  and year='2021'
  and month in ('7','07')
  and line_item_usage_amount < 1.0
  and line_item_resource_id not in (
    select distinct(line_item_resource_id)
    from cur
    where line_item_usage_type like '%VpcEndpoint-Byte%'
    and year='2021'
    and month in ('7','07')
    and line_item_usage_amount > 1.0
  )
group by line_item_usage_account_id, line_item_resource_id
```

[cur]: https://docs.aws.amazon.com/cur/latest/userguide/what-is-cur.html
[cur-athena]: https://docs.aws.amazon.com/cur/latest/userguide/cur-query-athena.html
[pl-pricing]: https://aws.amazon.com/privatelink/pricing/
[private-dns]: https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-interface.html#vpce-private-dns