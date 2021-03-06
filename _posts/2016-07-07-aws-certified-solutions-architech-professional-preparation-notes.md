---
layout: post
title: AWS Certified Solutions Architect Professional Prepartion Notes
---

## Elastic Compute Cloud (EC2)

- Instance store boot times are a lot slower than EBS. The boot/OS partition for instance store volumes is also limited to 10G.
- Instance store volume data persists in case of a reboot. The data is only lost in case of shutdown/termination/disk failure.
- Creating AMI from a instance store based EC2 instance requires the AMI toolkit to be used (not one button/one API call like EBS).
- There is a limit on sending emails (outgoing smtp) from the EC2 (hard to find exact number of the limit), for both auto-assigned public IP and static ElasticIP addresses. This limit can be lifted off by submitting a request form.
- ElasticIP addreses can have their reverse record set by submitting a request form. Corresponding forward DNS record pointing to that Elastic IP address must exist.
- There is a limit of number of on-demand and reserved instances that can be run. Overall, you are limited to running up to 20 On-Demand instances, purchasing 20 Reserved Instances, and requesting Spot Instances. The limit is also per instance type. For all instances type, reserved limit is 20 and on-demand limit varies from 2 (for larger instaces like i2.8xlarge) to 20 (for smaller instances like t2.micro). The limits can be increased by contacting Amazon. 
- All the hardware underlying Amazon EC2 uses ECC memory.
- M4/C4 instance types do not have instance store (EBS only), where as M3/C3 instance types does.
- D2 instancy type provides lowest cost per disk throughput performance.
- 5 Elastic IP addresses per region limit, which can be increased. ElasticIPs are charged when not associated to a running instance.
- One Availability Zone name (for example, us-east-1a) in two AWS customer accounts may relate to different physical Availability Zones.
- Enhanced networking support is available for selected instances types using SR-IOV. It is only available in HVM virtualization. X1 instances use Elastic Network Adapter (ENA) interface which uses "ena" linux driver. C3, C4, R3, I2, M4 and D2 instances use Intel 82599g Virtual Function Interface, which uses the “ixgbevf” Linux driver. These drivers are there by default in Amazon Linux AMI. Instructions for Linux/Windows AMI that do not have these drivers are available.
- A placement group can span peered VPCs; however, you will not get full-bisection bandwidth between instances in peered VPCs.
- Snapshots are only available through the Amazon EC2 APIs, not through S3 APIs.
- Auto Scaling Service cannot scale past the Amazon EC2 limit of instances that you can run.
- If you delete an Auto Scaling group, the instances will be terminated.
- Instances can also be reserved with a schedule.
- Reserved instance attributes can be modified i.e the availability zone and the instance type (with in the same family).
- Reserved instances apply across all accounts in case of consolidated billing.
- Spot instances limits are dynamic per region.
- Spot instances get 2 mintues before they are terminated
- Spot instances can be launched with a guarentee of upto 6 hours runtime (Block duration).
- ELB supports IPv6. Each Elastic Load Balancer has an associated IPv4, IPv6, and dualstack (both IPv4 and IPv6) DNS name, However, IPv6 is not supported in VPC at this time.
- ELB supports AMIs from AWS Marketplace but not from Amazon DevPay site.
- Cross-zone load balancing reduces the need to maintain equivalent numbers of back-end instances in each AZ, and improves your application's ability to handle the loss of one or more back-end instances (by distributing incoming requests evenly across all back-end instances, regardless of AZ they are in).
- You can perform load balancing for the following TCP ports: 25, 80, 443, and 1024-65535.
- You can't merge placement groups. Instead, you must terminate the instances in one placement group, and then relaunch those instances into the other placement group.
- When launching an instance you can specify the ENI to attach to the instance for both the primary (eth0) and additional elastic network interfaces.
- When you change the launch configuration for your Auto Scaling group, any new instances are launched using the new configuration parameters, but existing instances are not affected.
- There is a significant increase in latency when you first access each block of data on a new EBS volume that was restored from a snapshot SSD/PSSD/Magnetic. You can avoid this by accessing each block in advance (initialization/pre-warming).
- Encryption is supported with all EBS volume types and you can expect the same IOPS performance with a minimal effect on latency. You can access encrypted volumes the same way that you access existing volumes.

## Simple Storage Service (S3)

- Individual Amazon S3 objects can range in size from a minimum of 0 bytes to a maximum of 5 terabytes.

- Individual Amazon S3 objects can range in size from a minimum of 0 bytes to a maximum of 5 terabytes.
- For objects greater than 100 MB, it is recommended, and for objects greater than 5 GB, it is cumpulsory to use multipart upload.
- Any publicly available data in Amazon S3 can be downloaded via the BitTorrent protocol by adding the ?torrent parameter at the end of the GET request.
- S3 data storage is calculated in TimedStorage-ByteHrs which are added up at the end of the month.
- Data at rest can be encrypted by using SSE-S3, SSE-KMS or SSE-C (Server Side Encryption with Customer-Provide Keys) via Amazon provided encryption technology; Alternatively you can use your own encryption libraries to encrypt data before storing.
- Server-side encryption encrypts only the object data. Any object metadata is not encrypted.
- S3 stores objects lexicographically (alphabetical order) based on key name. Using a sequential prefix (e.g. timestamp) increases the likelihood that S3 will only target a specific partition, overwhelming the I/O capacity of the partition. Introducing some randomness in your key name/prefixes, will cause S3 to distribute across multiple partitions.
- Access can be restricted based on an aspect of the request, such as HTTP referrer and IP address.
- Data that is deleted from Standard - IA within 30 days will be charged for a full 30 days.
- Amazon S3 objects that are stored using the Amazon Glacier option are only accessible through the Amazon S3 APIs or Amazon S3 Management Console but not using Amazon Glacier APIs.
- Typical restore from glacier takes 3 to 5 hours. The restore request creates a temporary copy of the data in RRS. You can specify the amount of time in days for which the temporary copy is stored in RRS.
- CRR (Cross-Region Replication) is a bucket-level configuration, that replicates every object uploaded to a destination bucket in a different AWS region that you choose. You can chose to replicate subset of the objects in a bucket by specifying prefixes. Existing data in the bucket prior to enabling CRR is not replicated.
- CRR replicates: new objects, ACL updates, DELETE without version ID, SSE objects with S3-managed encryption key.
- CRR doesn't replicate: DELETE with version ID, SSE-C or SSE-KMS encrypted objects, lifecycle configurations, replica objects (if CRR is setup as A -> B -> C, objects replicted from A to B won't be replicated to C).
- CRR can be enabled back in a loop i.e. from A -> B and from B -> A. Note: replica objects will not be replicated only newer objects created in each bucket will be replicated.
- CRR requires versioning to be enabled on both the source and destination bucket.
- CRR can be cross account, you should create 
- CRR replication status can be seen on x-amz-replication-status header when calling GET/HEAD on the replication enabled object. The values can be PENDING, COMPLETED, or FAILED.
- S3 event notifications can be sent through Amazon SNS, Amazon SQS, or directly to AWS Lambda.
- Objects are past their expiration date are queued for removal. You will not be billed for objects after their expiration date, though the objects might be accessible while they are in queue before they are removed.
- If you upload several multipart object parts, but never commit them, you will still be charged for that storage. Use lifecycle policy that expires incomplete multipart uploads.
- S3 Transfer Acceleration (applied per bucket) enables fast/secure transfers leveraging Edge Locations. Once enabled, you can point your Amazon S3 PUT/GET requests to the s3-accelerate endpoint domain name e.g: <bucketname>.s3-accelerate.amazonaws.com. It can make storage gateway and/or any 3rd party client that connects to S3 directly perform faster. If objects/data set are smaller than 1GB, consider CloudFront's PUT/POST commands for optimal performance.
- When hosting a static website on S3, you need to enable CORS (Cross Origin Resource Sharing) if you want your website to call external domains.
- You can track bucket level operation using cloud trail and additionally use cloud watch with cloud trail e.g. get notified when the buckets get deleted or life cycle policy get changed.
- S3 has storage metrics in cloud watch available where by you can see metrics like total objects, total bytes in RRS/IA/Std/Glacier storage class etc. You can additionally setup alarms on these metrics. These metrics are emitted daily.
- Retrieving an archive from Glacier is a two-step process: Initiate an archive retrieval job; After the job completes, download the bytes.
- http://docs.aws.amazon.com/AmazonS3/latest/dev/access-control-auth-workflow-bucket-operation.html
- All data in the Glacier will be encrypted on the server side. Glacier handles key management and key protection. Customers wishing to manage their own keys can encrypt data prior to uploading it.
- You can retrieve a specific range of an archive from Glacier. Range retrievals are similar to regular retrievals in Amazon Glacier. Both require the initiation of a retrieval job that typically completes within 3-5 hours. You can use range retrievals to reduce or eliminate your retrieval fees. Using range retrievals, you provide a byte range that can start at zero (beginning), or at any 1MB interval thereafter (e.g. 1MB, 2MB, 3MB, etc).
- AWS account that owns a bucket can grant another AWS account permission to manage access policy. It allows that account to change anything in the policy.
- Buckets in all Regions provide read-after-write consistency for PUTS of new objects and eventual consistency for overwrite PUTS and DELETES.

	

## Virtual Private Cloud (VPC)

- Internet gateway is horizontally-scaled, redundant, and highly available, imposes no bandwidth constraints.
- Internet gateway is not required to establish a hardware VPN connection or VPC peering connections.
- Amazon does not enforce any restrictions on VPN throughput.
- VPCs currently cannot be addressed from IPv6 IP address ranges.
- VPC supports VPCs between /28 and /16 in size. Default VPC is of range 172.31.0.0/16 and with subnets of /20 netblocks. To change the size of a VPC you must terminate and create a new one.
- By default you are limited to 200 subnets per VPC which can be increased.
- Amazon reserves 5 IP addresses per subnet, the first 4 the last one, for IP networking purposes.
- Each instance has a default ENI. You can attach only one more network interface to your instance during launch. You can attach more later if your instance type supports it.
- The primary private IP address (can be set manually at launch) are retained for the instance's or interface's lifetime. Secondary private IP addresses can be assigned/unassigned or moved between interfaces or instances at any time.
- An IP address assigned to a running instance can only be used again by another instance once that original running instance is in a terminated state.
- You cannot assign IP addresses for multiple instances simultaneously, You can specify the IP address of one instance at a time when launching the instance.
- Public IPs can only be assigned to instances with one network interface.
- You can attach an ENI in one subnet to an instance in another subnet in the same VPC; however, both the elastic network interface and the instance must reside in the same Availability Zone.
- You can assigne multiple EIP addresses to EC2 instances. EIP addresses should only be used on instances in subnets configured to route their traffic directly to the Internet gateway. EIPs cannot be used on instances in subnets configured to use a NAT.
- You can create a default route for each subnet. The default route can direct traffic to egress the VPC via the IGW/VPG/NAT.
- Each route table has an un-deletable rule that routes anything to the VPCs CIDR block to a target of "local", i.e., the VPCs router. The router knows about all the VPCs attached subnets and how to route amongst them, and there’s no way to change that.
- Peering connections are only available between VPCs in the same region. Peering can be done with VPC in a different AWS account. Peered VPCs must have non-overlapping IP ranges.
- You cannot use AWS Direct Connect or hardware VPN connections to access VPCs you are peered with.
- Transitive peering relationships are not supported. i.e. If you peer VPC A to VPC B and VPC B to VPC C, that doesn't mean VPCs A and C are peered.
- Each network ACL includes a rule whose rule number is an asterisk. This rule ensures that if a packet doesn't match any of the other rules, it's denied. You can't modify or remove this rule.
- You cannot reference a security group from the peer VPC as a source/destination for ingress/egress rules in your security group. Instead, reference CIDR blocks of the peer VPC.
- The Amazon DNS server cannot resolve private DNS hostnames if your VPC's IP address range falls outside of the private IP addresses ranges specified by RFC 1918. 

## Direct Connect

- Direct Connect supports 1Gbps and 10Gbps ports. Speeds of 50Mbps, 100Mbps, 200Mbps, 300Mbps, 400Mbps, and 500Mbps can be ordered from any APN partners supporting AWS Direct Connect.
- AWS Direct Connect do not offer a SLA at this time
- You cannot connect to internet using Direct Connect.
- When creating a virtual interface to work with AWS services using public IP space (a.k.a. public virtual interface), you will receive all Amazon IP prefixes for the region that you are connecting to. Direct Connect customers in the US will receive the public IP prefixes for all US Regions.
- You get great consistent performance using DirectConnect, but unverifiable security (AWS can't guarantee any security past its connection to the colocation facility), and completely verifiable security with VPN but unpredictable performance. Using VPN over Direct Connect gives you the best of both worlds.

## Relational Database Service (RDS)

- By default, there is a limit of a total of 40 RDS DB instances (OnDemand/Reserved). Of those 40, up to 10 can be Oracle or SQL Server DB Instances under the "License Included" model.
- Oracle RDS limits you to 1 DB per RDS instance (no limit on schemas/DB), SQL Server has a limit of 30 databases per instance. Aurora, MySQL, MariaDB, PostgreSQL impose no software limit.
- You can access the MySQL slow query logs for your database by setting the "slow_query_log" DB Parameter and querying the mysql.slow_log table.
- You can use the Oracle trace file data to identify slow queries.
- You can use the client side SQL Server traces to identify slow queries.
- RDS Reserved Instances are purchased for the Region rather than for the AZ with the following attributes: DB Engine, DB Instance class, Deployment type, License Model and Region. Instance reservation can also be applied to a Read Replica.
- You can scale the RDS by modifying the instance type and/or storage. Changes will apply during maintenance window unless "apply-immediately" flag is selected. SQL Server does not currently support increasing storage.
- RDS uses EBS volumes automatically striped across multiple EBS volumes to enhance IOPS. For MySQL and Oracle you may observe some I/O capacity improvement if you scale up your storage.
- Storage can be increased while maintaining DB Instance availability. Compute resource scale will cause a temporary (few minutes) unavailability.
- Automated backups performs a full daily snapshot of your data and captures transaction logs. You can initiate a point-in-time restore and specify any second during your retention period, up to the Latest Restorable Time which is typically within the last five minutes.
- Automated backups can be turned off by setting the retention period to 0. Automated backups are deleted when the DB Instance is deleted, manual snapshots or final deletion snapshot are reatained.
- RDS supports two types of snapshot copies. Copy an automated DB snapshot to manual DB snapshot in the same region (so the DB snapshot is retained), Copy automated/manual DB snapshot to another region.
- It is strongly recommended to use the DNS Name to connect to your DB Instance as the underlying IP address can change (e.g., during a failover).
- You can update/modify a DB Subnet Group but explicitly changing the DB Subnet Group of a deployed DB instance is not currently allowed.
- You can encrypt DB connections using SSL. Amazon RDS generates an SSL certificate for each DB Instance.
- Amazon RDS supports encryption at rest for all database engines, using KMS. Currently encrypting an existing DB Instance is not supported.
- MultiAZ replication is synchronous, read replica replication is asynchronous.
- Read replicas are only supported for MySQL and PostgreSQL.
- MultiAZ instance latencies can be slightly higher than the standard deployments.
- You can initiate a MultiAZ failover by rebooting the primary.
- You may experience increased I/O latency (typically lasting a few minutes) during backups for both Single-AZ and Multi-AZ deployments.
- Backups must remain enabled for Read Replicas to work.
- MySQL and PostgreSQL currently allow you to create up to 5 Read Replicas. Cross-region Read Replicas are also supported.
- MySQL supports a second-tier Read Replica from an existing first-tier Read Replica, PostgreSQL doesn't.
- Read Replicas cannot be MultiAZ at this time.
- MySQL can be configured to permit DDL SQL statements against a Read Replica.
- Read Replica can be promoted into a stand alone instance.
- If an existing Read Replica has fallen too far behind, you can delete it and create a new one with the same endpoint by using the same DB Instance Identifier and Source DB Instance Identifier as the deleted Read Replica.
- You cannot take snapshots or configure automated backups on a Read Replica, use MultiAZ.
- Deleting source DB instance do not delete the Read Replica(s). In case of PostgreSQL, all read replicas will be promoted to stand alone automatically.
- MySQL doesn't provide access to binary logs, similarly PostgreSQL doesn't provide access to WAL files, to setup replication on your own.
- Enhanced Monitoring supports every instance type except t1.micro and m1.small.
- You cannot reduce the amount of storage once it has been allocated. The only way to reduce the amount of storage allocated to a DB instance is to dump the data out, create a new DB instance with less storage space, and then load the data into the new DB instance.
- Magnetic storage is not reserved for a single DB instance, so performance can vary greatly depending on the demands placed on shared resources by other customers.
- 

## Identity and Access Management (IAM)

- Groups cannot belong to other groups.
- An IAM user can have following types of credentials: access key, X.509 certificate, SSH key, password, MFA. Currently these user based SSH keys work only with AWS CodeCommit to access their repositories (not as EC2 SSH keys).
- You can organize users and groups under paths e.g. /mycompany/division/project/joe
- IAM usernames can be email addresses.
- You can only act as one IAM role when making requests to AWS services.
- You cannot add IAM role to IAM group.
- You can add up to 10 managed policies to a user, role, or group. Inline policies for user, role and group are limited by characters, 2048, 10240 and 5120 characters respectively.
- You are limited to 250 IAM roles under your AWS account, limit increase can be requested.
- IAM role for EC2 instance cannot be more than one, and cannot be attached after launch.
- When using IAM role for EC2 isntance, AWS SDK automatically uses the access keys (temporary security credentials) provided by the STS. A custom application can also retrieve these access keys from the EC2 meta-data.
- Temporary security credential cannot be revoked prior to its expiration. However if the request was made by an IAM user, you can revoke permissions of that IAM user, which will revoke privileges for all temporary security credentials issued by that IAM user.
- You are limited to 250 IAM roles under your AWS account, limit increase can be requested.
- There is no limit to the number of federated users who can be given access to the console.
- MFA-protected API access is supported by all AWS services that support temporary security credentials.
- The following API calls return temporary security credentials: AssumeRole(15m/1hr/1hr), AssumeRoleWithSAML(15m/1hr/1hr), AssumeRoleWithWebIdentity(15m/1hr/1hr), GetFederationToken(IAMuser:15m/36hr/12hr Root:15m/1hr/1hr), GetSessionToken(IAMuser:15m/36hr/12hr Root:15m/1hr/1hr).
- 

## Route53

- You can create multiple hosted zones for the same domain name. e.g. hosted zone Z1234 can be a test version of example.com hosted on name servers ns-1, ns-2, ns-3, and ns-4. Similarly, hosted zone Z5678 can be your production version of example.com, hosted on ns-5, ns-6, ns-7, and ns-8. Route 53 will answer DNS queries for example.com differently depending on which name server you send the DNS query to.
- You can use Route 53 to configure split-view DNS / split-horizon DNS. You can create internal and external versions of the same website or application (e.g. for testing), you can configure public and private hosted zones to return different internal and external IP addresses for the same domain name.
- Route 53 support wildcard entries e.g. \*.example.com.
- Route 53 does not support DNSSEC at this time.
- Route 53 supports both forward (AAAA) and reverse (PTR) IPv6 records. However, the Route 53 service itself is not available over IPv6 at this time.
- Latency Based Routing for minimizing end-user latency. Geo DNS for compliance, localization etc.
- Geo DNS provides three levels of geographic granularity: continent, country, and state; And a global record which is served in cases where an end user’s location doesn’t match any of the specific Geo DNS records you have created.
- Amazon Route 53 Traffic Flow is an easy-to-use and cost-effective global traffic management service. You create a policy record which associates the traffic policy with the appropriate DNS name within an Amazon Route 53 hosted zone that you own.
- You can associate multiple VPCs with a single hosted zone (private).
- Route 53 health checks consider an HTTP 3xx code to be a successful response, so they don’t follow the redirect.
- If there are no healthy endpoints remaining in a resource record set, Route 53 will behave as if all health checks are passing.
- You can use "Enable String Matching" option to check for the presence of a designated string in a server response when configuring Route 53 health checks.

## CloudFront

- Geo Restriction feature lets you specify a list of countries in which your users can or cannot access your content. CloudFront responds with 403 to a request from a restricted country.
- Streaming Summary: If you have media files HLS format or Microsoft Smooth Streaming format prior to storing in Amazon S3 (or a custom origin), you can use an Amazon CloudFront web distribution to stream in that format without having to run any media servers. In addition you can also run a third party streaming server (e.g. Wowza Media Server available on AWS Marketplace) on Amazon EC2 which can convert a media file to the required HTTP streaming format. This server can then be designated as the origin for an Amazon CloudFront web distribution. Another option, if you want to stream using RTMP, is to store your media files on Amazon S3 and use it as the origin for an Amazon CloudFront RTMP distribution.
- You can enable access logging to write detailed log information in a W3C extended format into S3, containing detailed information about each request, including: object requested, edge location, client IP, referrer, user agent, cookie header, and result type (cache hit/miss/error).
- Signed URLs use cases: RTMP distribution (signed cookies aren't supported for RTMP distributions), Restrict access to individual files, Client doesn't support cookies.
- Signed Cookies use cases: Provide access to multiple restricted files, You don't want to change your current URLs.
- A cache behavior lets you configure a variety of CloudFront functionality for a given URL path pattern for files on your website.
- When CloudFront updates bucket permissions to grant the specified origin access identity the permission to read objects in your bucket, it does not remove existing permissions. If users currently have permission to access the objects in your bucket using Amazon S3 URLs, they will still have that permi0ssion after CloudFront updates your bucket permissions.
- CloudFormation allows you to name some resources (such as Amazon S3 buckets), not all. You can assign logical names to AWS resources in a template. When a stack is created, AWS CloudFormation binds the logical name to the name of the corresponding actual AWS resource. Actual resource names are a combination of the stack and logical resource name. This allows multiple stacks to be created from a template without fear of name collisions between AWS resources.
- If you configure CloudFront to use HTTPS when communicating with your origin...CloudFront verifies that your certificate was issued by an established third-party certificate authority...You cannot use a self-signed certificate.
- Reserved Capacity gives you the option to commit to a minimum monthly usage level for 12 months or longer. Reserved Capacity agreements begin at a minimum of 10 TB of data transfer per month from a single region.

## CloudFormation

- CloudFormation provides a WaitCondition resource that acts as a barrier, blocking the creation of other resources until a completion signal is received from an external source such as your application, or management system.
- CloudFormation provides three deletion policies: Delete (any resource), Retain (any resource) and Snapshot (EC2/RDS/RedShift).
- During a stack update, you cannot add or update a DeletionPolicy by itself. You can add or update a DeletionPolicy only when you include changes that add, modify, or delete resources. If you need to add or modify a DeletionPolicy and don't want to make any changes to a resource, you can use a dummy resource, such as AWS::CloudFormation::WaitConditionHandle.
- You can pause a CloudFormation template using wait conditions. For EC2 and Auto Scaling resources, it is recommended that you use a CreationPolicy attribute instead of wait conditions. Add a CreationPolicy attribute to those resources and use the cfn-signal helper script to signal when an instance has been successfully created.
- When updating a stack, before making changes to your resources, you can generate a change set, which is summary of your proposed changes. Change sets allow you to see how your changes might impact your running resources, especially for critical resources, before implementing them. Change set doesn't evaluate resource limits or permissions.
- If stack creation fails, CloudFormation rolls back your changes by deleting the resources that it created.
- If uploading template from local computer, Cloudfront will automatically create upload and store the template in S3.
- CloudFormation templates that are stored in an S3 bucket must be accessible to the user who is creating the stack, and must exist in the same region as the stack being created.
- The AWS::Region pseudo parameter enables you to get the region where the stack is created.
- The intrinsic function Fn::FindInMap returns the value corresponding to keys in a two-level map that is declared in the Mappings section.
- The intrinsic function Fn::GetAZs lists Availability Zones for a region enabling template authors to write templates that adapt to the calling user's access.
- DependsOn attribute enables you to specify that one resource must be created after another e.g. EC2 instance with a public IP address is dependent on the VPC-gateway attachment, An EC2 instance might be depedent on a service role etc.
- The Metadata attribute enables you to associate structured data with a resource enabling you to add data in JSON format to the resource declaration.
- AWS::CloudFormation::Stack enables you to nest another stack as a resource within your template.
- When you launch stacks, you can install and configure software applications on Amazon EC2 instances by using the cfn-init helper script and the AWS::CloudFormation::Init resource. By using AWS::CloudFormation::Init, you can describe the configurations that you want rather than scripting procedural steps. You can also update configurations without recreating instances.
- By default, the cfn-hup daemon runs every 15 minutes, so it may take up to 15 minutes for the application to change once the stack has been updated.
- Each instance will run the cfn-hup hooks on its own schedule; there is no coordination between the instances in the stack. You should consider the following: If the cfn-hup changes run on all Amazon EC2 instances in the Auto Scaling group at the same time, your service might be unavailable during the update. If the cfn-hup changes run at different times, old and new versions of the software may be running at the same. To avoid these issues, consider forcing a rolling update on your instances in the Auto Scaling group



## Elastic BeanStalk

- General use case: Beanstalk makes it easier for developers to quickly deploy and manage applications in the AWS cloud without experience in cloud computing.
- Beanstalk supports Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker web applications.
- The current version of AWS Elastic Beanstalk uses the Amazon Linux AMI or the Windows Server 2012 R2 AMI.
- You can: Choose OS, Choose database and storage options, Enable login access to Amazon EC2, run in more than one AZ, Use HTTPS on ELB, Access built-in CloudWatch monitoring, Adjust application server settings, pass environment variables, Run custom appliactions like memcache in EC2, Access log files without logging in to the application servers.
- One environment cannot support two different environment tiers because each requires its own set of resources; a worker environment tier and a web server environment tier each require an Auto Scaling group, but Elastic Beanstalk supports only one Auto Scaling group per environment.
- When updating Elastic Beanstalk with Git deployment, only the modified files are transmitted to AWS Elastic Beanstalk and updates typically take a few seconds.
- Application can be configured to automatically scale tens or even hundreds of times based on thresholds such as CPU utilization or network bandwidth.
- If an underlying EC2 instance fails for any reason, AWS Elastic Beanstalk will use Auto Scaling to automatically launch a new instance.
- Beanstalk stores your application files and optionally server log files in S3. S3 bucket will be automatically created for you and the uploaded files will be automatically copied from your local client to Amazon S3.
- You may configure Elastic Beanstalk to copy your server log files every hour to Amazon S3. You do this by editing the environment configuration settings.
- Beanstalk can automatically provision an RDS DB Instance. The connectivity information to the DB Instance is exposed to your application by environment variables.
- Application can be configured to automatically scale tens or even hundreds of times based on thresholds such as CPU utilization or network bandwidth.
- By default, your application is available publicly at myapp.elasticbeanstalk.com for anyone to access. You can add restrictions using security group rules, network ACLs, and custom route tables.
- You can use IAM users/policies for restricting access to Beanstalk. Beanstalk offers two templates: a read-only access template and a full-access template. You can allow or deny permissions to specific AWS Elastic Beanstalk resources such as applications, application versions, and environments.
- You can specify a maintenance window to update Beanstalk environments automatically to the latest version (for new patch and minor platform versions) of the underlying platform running your application. Beanstalk will not automatically perform major platform version updates (you must manually initiate the update).
- Managed platform updates uses an immutable deployment mechanism to perform the updates, ensuring that no changes are made to the existing environment until a parallel fleet of Amazon EC2 instances, with the updates installed, is ready to be swapped in with the existing instances, which are then terminated.

## AWS OpsWorks

- OpsWorks is an integrated configuration management platform for IT administrators and DevOps engineers who want a high degree of customization and control over operations. AWS OpsWorks users leverage Chef recipes to automate operations like software configurations, package installations, database setups, server scaling, and code deployment.
- OpsWorks support on-premises servers.
- OpsWorks currently supports Amazon Linux, Ubuntu 12.04 LTS, Ubuntu 14.04 LTS, and Windows Server 2012 R2. You can use your own AMIs, using your own Windows AMIs is not currently supported. 
- OpsWorks can retrieve the code from Git, Subversion, HTTP and S3 bundles. You can also use Chef recipes to deploy your apps from anywhere you like using rsync or scp.
- You can create instances in multiple availability zones, and your load balancer will route traffic among your instances. If any instance fails, OpsWorks’ auto healing can replace it
- The following lifecycle events are supported: Setup (when the instance has successfully booted), Configure (when stack changes state e.g. add instance to LB), Deploy (whenever an application is deployed), Undeploy (when you delete an application), Shutdown (sent to an instance 45 seconds before actually stopping the instance).
- Rebooting an instance does not trigger any lifecycle events.
- Configure event occurs on all of the stack's instances when one of the following occurs: An instance enters/leaves the online state, add/remove EIP, Attach/detache ELB to a layer.
- You cannot use user data to initialize the instance.
- OpsWorks supports automatic time and load-based instance scaling to adapt the number of running instances to match your load.
- OpsWorks supports IAM users, permissions, and roles. You can designate permissions by user, including view, deploy, and manage.
- The pricing for each on-premises server on which you install the OpsWorks agent is $0.02 per hour. There is no additional charge for Amazon EC2 instances supported by AWS OpsWorks.
- Upgrade Operating System (Linux only) Upgrades the instance's Amazon Linux or RHEL operating systems to the latest version.


## DynamoDB

- Table names must be unique within each region. That is, you can have two tables with same name if you create the tables in different regions.
- When reading data from Amazon DynamoDB, users can specify whether they want the read to be eventually consistent (default) or strongly consistent.
- DynamoDB supports fast in-place atomic updates.
- DynamoDB runs exclusively on Solid State Drives.
- There is no limit to the amount of data you can store in an Amazon DynamoDB table.
- There is no theoretical limit on the maximum throughput you can achieve. If you wish to exceed throughput rates of 10,000 writes/second or 10,000 reads/second, you must first contact Amazon. DynamoDB removes the need to partition across database tables for throughput scalability.
- DynamoDB table has provisioned read-throughput and write-throughput associated with it. You are billed by the hour whether or not you are sending requests to your table. You can increase provisioned throughput as often as you want but decrease it four times per day only. DynamoDB is designed to scale its provisioned throughput up or down while still remaining available.
- There is no limit to the number of attributes that an item can have, however the total size of an item, including attribute names and attribute values, cannot exceed 400KB.
- DynamoDB Streams provides a time-ordered sequence of item-level changes made to data in a table in the last 24 hours. You can access a stream using SDK or Kinesis Client Library (KCL).
- DynamoDB cross-region replication allows you to maintain identical copies (called replicas) of a DynamoDB table (called master table) in single master mode, in one or more AWS regions. It is setup using the replication management app that you launch from the AWS CloudFormation console. There are no limits on the number of replicas tables from a single master table. The provisioned capacity for read replicas is independent of the masater. Replicas are updated asynchronously.
- DynamoDB cross-region replication, when launched, uses DynamoDB streams to keep the data in sync. Additionally it also uses an EC2 instance of your choice to host the replication application and an SQS queue that queues control commands.
- The replication application will not create or propagate changes made on secondary indices on the master table to replica tables.
- DynamoDB Triggers is a feature which allows you to execute custom actions (Lambda functions) based on item-level updates on a DynamoDB table. To create a trigger, you associate a Lambda function to the stream. When the updates are published, Lambda function reads it and executes the code in the function.
- DynamoDB supports four scalar data types: Number, String, Binary, and Boolean. Additionally, DynamoDB supports collection data types: Number Set, String Set, Binary Set, heterogeneous List and heterogeneous Map. DynamoDB also supports NULL values. DynamoDB supports key-value and document data structures. For JSON data type, you can use the document SDK to pass JSON data directly to DynamoDB
- DynamoDB supports GET/PUT operations using a user-defined primary key (specified when creating a table). DynamoDB also provides flexible querying on non-primary key attributes using Global Secondary Indexes and Local Secondary Indexes.
- A primary key can either be a single-attribute partition key (e.g. UserID) or a composite partition-sort key (e.g. UserID<partition> + Timestamp<sort>).
- Amazon DynamoDB supports two types of secondary indexes: Local secondary index, an index that has the same partition key as the table, but a different sort key. Global secondary index — an index with a partition or a partition-and-sort key that can be different from those on the table.
- You can have maximum of 5 Local secondary indexes and 5 GSI Global secondary indexes. You need to create a LSI at the time of table creation. It can’t currently be added later on.
- Fine Grained Access Control (FGAC) gives control over data in the table i.e. who (caller) can access which items or attributes of the table and perform what actions. FGAC is used in concert with IAM.
- DynamoDB support conditional operations, you can specify a condition that must be satisfied for a put, update, or delete operation to be completed on an item.
- DynamoDB allows atomic increment and decrement operations on scalar values.
- Scan operation iterates over items with 1 MB size limit of items scanned per operation.
- For Query and Scan calls, DynamoDB uses the cumulative size of the processed items to calculate provisioned throughput.
- When uploading data to a table, you get better performance if you upload data to all the allocated servers (DynamoDB partitions your table data on multiple servers) simultaneously. To fully utilize all of the throughput capacity that has been provisioned for your tables, you need to distribute your workload across your hash attribute values.
- DynamoDB provides some flexibility in the per-partition throughput provisioning. When you are not fully utilizing a partition's throughput, DynamoDB reserves a portion of your unused capacity for later "bursts" of throughput usage. This reserved throughput can be consumed even faster than the provisioned throughput capacity. You should not design your application dependent on burst capacity being available at all times. DynamoDB can and does use burst capacity for background maintenance and other tasks without prior notice.
- LSI are created at the same time that you create a table. You cannot add a local secondary index to an existing table, nor can you delete any local secondary indexes that currently exist.

## ElastiCache

- You can update an existing Cache Subnet Group but changing the Cache Subnet Group of a deployed Cache Cluster is not currently allowed.
- All clients to an ElastiCache Cluster must be within the Amazon EC2 network, and authorized via security groups.
- Access to your cache nodes is controlled using the VPC Security Group and the Cache Subnet Group.

### Memecached

- You can run a maximum of 50 Cache Nodes per region. 
- You can associate an SNS topic with your Cache Cluster while creating it. You'll receive cluster notifications of this topic for e.g. Cache Node recovery occurred etc.
- You could add more Cache Nodes to your existing Cache Cluster by using the "Add Node" option on "Nodes" tab for your Cache Cluster 
- For each cluster amazon will create a unique configuration endpoint (DNS record) that can be queried to get the nodes that belong in that cluster. An Auto Discovery capable memecache client (provided by Amazon for Java and PHP), can use this feature to discover all nodes in the cluster. It will automtically discover addition/deletion/failure of cluster nodes. You will not get the Auto Discovery feature with the existing Memcached clients.

### Redis

- ElastiCache for Redis node may take on a primary or a read replica role. A primary node can be replicated to multiple read replica nodes. An ElastiCache for Redis cluster is a collection of one or more ElastiCache for Redis nodes of the same role; the primary node will be in the primary cluster and the read replica node will be in a read replica cluster. An ElastiCache for Redis replication group encapsulates the primary and read replica clusters for a Redis installation. A replication group will have only one primary cluster and zero or many read replica clusters. All nodes within a replication group (and consequently cluster) will be of the same node type, and have the same parameter and security group settings.
- You can achieve persistence by snapshotting your Redis data using the Backup and Restore feature.
- Read Replicas serve two purposes in Redis: Failure Handing and Read Scaling.
- Your read replica may only be provisioned in the same or different Availability Zone of the same Region as your cache node primary.
- ElastiCache allows you to create up to five (5) read replicas for a given primary cache node.
- Creating a read replica of another read replica is not supported.
- Read Replicas use Redis’s asynchronous replication technology.
- If an existing read replica has fallen too far behind to meet your requirements, you can reboot it.
- You can use Multi-AZ if you are using ElastiCache for Redis and have a replication group consisting of a primary node and one or more read replicas. If the primary node fails, ElastiCache will automatically detect the failure, select one from the available read replicas(with the smallest asynchronous replication lag), and promote it to become the new primary.
- A customer-initiated reboot of a primary cluster does not trigger automatic failover. Other reboots and failures do trigger automatic failover.
- In a Replication Group, you can choose to backup the primary or any of the read-replica clusters.
- You can specify the S3 location of your RDB file during cluster creation to use your own RDB snapshots to seed the cluster.

## Simple Queue Service (SQS)

- SQS message retention period is configurable and can be set anywhere from 1 minute to 2 weeks. The default is 4 days and once the message retention limit is reached your messages will be automatically deleted.
- SQS Doesn't guarentee the order. If order is critical, you should place sequencing information to order it your self.
- Amazon SQS supports up to 12 hours maximum visibility timeout.
- Amazon SQS allows you to send up to 10 attributes (meta data) on each message apart from the message body.
- Using long polling may reduce the cost of using SQS, as you can reduce the number of empty receives.
- If your application has a single thread polling multiple queues, switching from short polling to long polling will likely not work, as the single thread will wait for the long poll timeout on any empty queues, delaying the processing of any queues which may contain messages.
- The maximum long poll timeout is 20 seconds.
- AmazonSQSBufferedAsync client (only supported for Java) supports automatic batching of multiple SendMessage, DeleteMessage or ChangeMessageVisibility requests into batches of each type, without any changes required by the application. Additionally, it supports prefetching of messages into a local buffer. Overall it increases the throughput and reduces the latency of your application, while saving you money by making fewer SQS requests.
- You can subscribe one or multiple SQS queues to SNS topic. SNS will deliver the message to all the SQS queues that are subscribed to the topic.
- Amazon SQS has its own resource-based permissions system identical to IAM.
- After a message is read from a queue, it will not be visible to any other reader until the configured visibility timeout. During which it should be processed and deleted. If the component processing the message fails, the message will again become visible to any component reading the queue once the visibility timeout ends. 
- If you issue a DeleteMessage request on a previously deleted message,  SQS returns a "success" response.
- You can set the MaximumMessageSize attribute anywhere from 1024 bytes (1KB), up to 262144 bytes (256KB) to configure the maximum message size.
- Typically, you set up a DLQ to receive messages after a max number of processing atttempts have been reached. DLQs provides the ability to isolate messages that could not be processed for later analysis.
- The Permissions API provides a simple interface for developers to share access to a queue but it cannot allow for conditional access and more advanced use cases. The SetQueueAttributes operation supports the full access policy language.
- SQS in each region is totally independent in message stores and queue names. 
- Delay queues allow you to postpone the delivery of new messages in a queue for a specific number of seconds (0-900). Any message send to that queue will be invisible to consumers for the duration of the delay period. You can also turn an existing queue into a delay queue by using SetQueueAttribute to set the queue's DelaySeconds attribute.

## Simple Workflow Service (SWF)

- Amazon SWF presents a task-oriented API, whereas Amazon SQS offers a message-oriented API.
- SWF ensures that a task is assigned only once and is never duplicated.
- 

## Kinesis

- The maximum size of a data blob (the data payload before Base64-encoding) within one record is 1 MB.
- Each shard provides a capacity of 1MB/sec (1000 TPS max) data input and 2MB/sec (5 TPS max) data output. You can monitor shard-level metrics in Amazon Kinesis Streams and add or remove shards from your stream dynamically as your data throughput changes by resharding the stream.
- Partition key is used to segregate and route records to different shards of a stream.
- You can add data to Kinesis stream via PutRecord and PutRecords operations, Kinesis Producer Library (KPL), or Kinesis Agent (java based application available for Amazon Linux/RHEL).
- Kinesis Application is a data consumer that reads and processes data from an Amazon Kinesis stream.
- Kinesis Client Library (KCL) for Java/Python/Ruby/Node.js/.NET is a pre-built library that helps you easily build Amazon Kinesis Applications for reading and processing data from an Amazon Kinesis stream.
- Kinesis Connector Library is a pre-built library (built on top of KCL) that integrates Kinesis with other AWS services and third-party tools e.g. DynamoDB, Redshift, S3, and Elasticsearch.
- Kinesis Storm Spout is a pre-built library that helps you easily integrate Amazon Kinesis Streams with Apache Storm.
- One record processor maps to one shard and processes records from that shard.
- An application calls KCL (with configuration such as the stream name and AWS credentials) to instantiate a worker. This call also passes a reference to an IRecordProcessorFactory implementation. KCL uses this factory to create new record processors as needed to process data from the stream. KCL communicates with these record processors using the IRecordProcessor interface.
- KCL automatically creates DynamoDB table for each Kinesis Application to track and maintain state information such as resharding events and sequence number checkpoints. The DynamoDB table shares the same name with the application so that you need to make sure your application name doesn’t conflict with any existing DynamoDB tables under the same account within the same region.


## RedShift

- Redshift data warehouse is a collection of computing resources called nodes, which are organized into a group called a cluster. Each cluster runs an Amazon Redshift engine and contains one or more databases.
- You can use most SQL client tools with Amazon Redshift JDBC or ODBC drivers to connect to an Redshift cluster.
- Single Node clusters consist of a single node which performs both leader and compute functions.
- Multi Node clusters consist of compute nodes that you select and a leader node that is added to your cluster, free of charge. The leader node is the access point for ODBC/JDBC and generates the query plans executed on the compute nodes.
- Redshift data cluster can contain from 1-128 compute nodes
- Redshift supports KMS and HSM for data encryption.
- You cannot access your nodes directly. 
- It is recommended to use the COPY command to load large datasets into Amazon Redshift from Amazon S3 or DynamoDB. The data can be located in an S3 bucket, an EMR cluster, or a remote host that is accessed using SSH.
- Data Pipeline provides a high performance, reliable, fault tolerant solution to load data from a variety of AWS data sources into Redshift.
- Redshift uses replication and continuous backups to enhance availability and improve data durability and can automatically recover from component and node failures.
- Single node clusters do not support data replication. In the event of a drive failure you will need to restore the cluster from snapshot on S3.
- Redshift always attempts to maintain at least three copies of your data (the original and replica on the compute nodes and a backup in Amazon S3).
- Redshift can also asynchronously replicate your snapshots to S3 in another region for disaster recovery. The replication to a remote region can be done manually or set to be done automatically with retention on remote copied snapshots.
- Redshift snapshot include the following details: No. of nodes, Type of nodes, Cluster configuration, data.	

- Redshift will remain available in the event of a drive failure however you may see a slight decline in performance for certain queries.
- In case on a node failure, the cluster will be unavailable for queries and updates until a replacement node is provisioned and added to the DB.
- In case the cluster's AZ goes down or becomes available, you will not be able to use your cluster until power and network access to the AZ are restored.
- You can create a cluster using either Dense Storage (DS) nodes or Dense Compute (DC) nodes.
- Redshift's MPP architecture means you can increase your performance by increasing the number of nodes in your data warehouse cluster.
- There is no Data Transfer charge for data transferred to or from Amazon Redshift outside of Amazon VPC. Data Transfer to or from Redshift in Amazon VPC accrues standard AWS data transfer charges.
- You can increase the number of nodes within your cluster. The cluster remains available for read operations while a new data warehouse cluster gets created during scaling operations.
- When shutting down a cluster allows a final creation of snapshot and the cluster is deleted. Deleting a cluster will not allow the creation of a final snapshot. Both options will delete all any automated snapshots associated.
- 

## TODO

- CloudTrail

- CloudWatch 

- Elastic MapReduce

- Data Pipeline

- Simple Email Service (SES)

- Simple Notifications Service (SNS)

- Lambda 

- Amazon Cognito
