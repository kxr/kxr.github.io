---
layout: post
title: AWS Certified Solutions Architect Professional Prepartion Notes
---

## Elastic Compute Cloud (EC2)

- There is a limit on sending emails (outgoing smtp) from the EC2 (hard to find exact number of the limit), for both auto-assigned public IP and static ElasticIP addresses. This limit can be lifted off by submitting a request form.
- ElasticIP addreses can have their reverse record set by submitting a request form. Corresponding forward DNS record pointing to that Elastic IP address must exist.
- There is a limit of number of on-demand and reserved instances that can be run. Overall, you are limited to running up to 20 On-Demand instances, purchasing 20 Reserved Instances, and requesting Spot Instances. The limit is also per instance type. For all instances type, reserved limit is 20 and on-demand limit varies from 2 (for larger instaces like i2.8xlarge) to 20 (for smaller instances like t2.micro). The limits can be increased by contacting Amazon. 
- All the hardware underlying Amazon EC2 uses ECC memory.
- M4/C4 instance types do not have instance store (EBS only), where as M3/C3 instance types does.
- 5 Elastic IP addresses per region limit, which can be increased. ElasticIPs are charged when not associated to a running instance.
- One Availability Zone name (for example, us-east-1a) in two AWS customer accounts may relate to different physical Availability Zones.
- Enhanced networking support is available for selected instances types using SR-IOV. It is only available in HVM virtualization. X1 instances use Elastic Network Adapter (ENA) interface which uses "ena" linux driver. C3, C4, R3, I2, M4 and D2 instances use Intel 82599g Virtual Function Interface, which uses the “ixgbevf” Linux driver. These drivers are there by default in Amazon Linux AMI. Instructions for Linux/Windows AMI that do not have these drivers are available.
- Snapshots are only available through the Amazon EC2 APIs, not through S3 APIs.
- Auto Scaling Service cannot scale past the Amazon EC2 limit of instances that you can run.
- If you delete an Auto Scaling group, the instances will be terminated.
- Instances can also be reserved with a schedule.
- Reserved instance attributes can be modified i.e the availability zone and the instance type (with in the same family).
- Reserved instances apply across all accounts in case of consolidated billing.
- Spot instances limits are dynamic per region.
- Spot instances get 2 mintues before they are terminated
- Spot instances can be launched with a guarentee of upto 6 hours runtime (Block duration).

## Simple Storage Service (S3)

## Glacier

## Virtual Private Cloud (VPC)

## Direct Connect

## Relational Database Service (RDS)

## Elastic Load Balancer (ELB)

- ELB supports IPv6. Each Elastic Load Balancer has an associated IPv4, IPv6, and dualstack (both IPv4 and IPv6) DNS name, However, IPv6 is not supported in VPC at this time.
- ELB supports AMIs from AWS Marketplace but not from Amazon DevPay site.


## Auto Scaling

## Identity and Access Management (IAM)

## Route53

## CloudFront

## CloudFormation

## Elastic BeanStalk

## AWS OpsWorks

## Amazon Cognito

## DynamoDB

## ElastiCache

## Elastic MapReduce

## Kinesis

## Data Pipeline

## RedShift

## CloudTrail

## CloudWatch 

## Simple Queue Service (SQS)

## Simple Email Service (SES)

## Simple Workflow Service (SWF)

## Simple Notifications Service (SNS)

## Lambda 
