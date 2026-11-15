# AWS Labs

Hands-on labs on Amazon Web Services.

## Identity and Access

| # | Lab | Topics |
|---|---|---|
| 01 | [Introduction to AWS IAM](lab-01-intro-to-iam/) | IAM users, user groups, managed vs inline policies, least privilege, IAM sign-in URL |
| 06 | [Access Control with IAM: Permissions Boundaries, Groups, and Tags](lab-06-iam-access-control-abac/) | Permissions boundaries, implicit deny, group policies, tag-based access control (ABAC), cloud cost and cleanup |

## Networking

| # | Lab | Topics |
|---|---|---|
| 02 | [Build a VPC and Launch a Web Server](lab-02-build-vpc-launch-web-server/) | VPC, public and private subnets, internet gateway, NAT gateway, route tables, security groups, EC2 user data |

## Compute

| # | Lab | Topics |
|---|---|---|
| 03 | [Introduction to Amazon EC2](lab-03-intro-to-ec2/) | EC2 launch, user data, monitoring, security group rules, resizing an instance and EBS volume, service quotas, termination and stop protection |
| 04 | [AWS Lambda: Stopping an EC2 Instance on a Schedule](lab-04-aws-lambda/) | Lambda functions, execution roles, EventBridge schedules, Python with boto3, CloudWatch metrics |
| 05 | [AWS Elastic Beanstalk: Deploying a Web App](lab-05-elastic-beanstalk/) | Elastic Beanstalk environments, application deployment, load balancer, Auto Scaling group, security groups |
| 12 | [Scale and Load Balance a Web App: ALB and EC2 Auto Scaling](lab-12-scale-load-balance/) | AMIs, launch templates, Application Load Balancer, target groups and health checks, Auto Scaling groups, target tracking, CloudWatch alarms, load testing |

## Storage

| # | Lab | Topics |
|---|---|---|
| 07 | [Working with Amazon EBS: Volumes and Snapshots](lab-07-working-with-ebs/) | EBS volumes, attaching and mounting, ext3 and /etc/fstab, snapshots, restoring a volume from a snapshot |
| 08 | [Block and Object Storage: Amazon EBS and Amazon S3](lab-08-block-and-object-storage/) | EBS volumes, parted and GPT, ext4, mounting by UUID in /etc/fstab, moving a volume between instances, S3 buckets, Block Public Access, S3 REST API with Postman |

## Databases

| # | Lab | Topics |
|---|---|---|
| 09 | [Build a Database Server with Amazon RDS and Connect a Web App](lab-09-rds-database-web-app/) | Amazon RDS MySQL, Multi-AZ deployment, DB subnet groups, security group to security group rules, private subnets, connecting a web app |

## Security and Monitoring

| # | Lab | Topics |
|---|---|---|
| 10 | [Triggering a GuardDuty Finding: S3 Block Public Access Disabled](lab-10-guardduty-s3-block-public-access/) | Amazon GuardDuty, S3 Protection, CloudTrail events, account and bucket Block Public Access, Policy:S3 findings, incident response |
| 11 | [Securing SSH Access to an EC2 Instance](lab-11-secure-ssh-ec2/) | EC2 key pairs, security group inbound rules, SSH from a single /32 IP, removing 0.0.0.0/0, GuardDuty Runtime Monitoring |

> See also: EC2 launch, security groups and SSH key pairs in [Linux Lab 01](https://github.com/willkhot/Linux-Labs/tree/main/lab-01-deploy-linux-servers) in my Linux Labs.

[← Back to Cloud Labs](../)
