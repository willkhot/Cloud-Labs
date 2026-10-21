# ☁️ Cloud Labs

Hands-on labs on public cloud platforms, organized by provider. Each lab has a write-up with objectives, the steps I followed, screenshots, and key takeaways.

## [AWS](aws/)

| # | Lab | Topics |
|---|---|---|
| 01 | [Introduction to AWS IAM](aws/lab-01-intro-to-iam/) | IAM users, user groups, managed vs inline policies, least privilege, IAM sign-in URL |
| 02 | [Build a VPC and Launch a Web Server](aws/lab-02-build-vpc-launch-web-server/) | VPC, public and private subnets, internet gateway, NAT gateway, route tables, security groups, EC2 user data |
| 03 | [Introduction to Amazon EC2](aws/lab-03-intro-to-ec2/) | EC2 launch, user data, monitoring, security group rules, resizing an instance and EBS volume, service quotas, termination and stop protection |
| 04 | [AWS Lambda: Stopping an EC2 Instance on a Schedule](aws/lab-04-aws-lambda/) | Lambda functions, execution roles, EventBridge schedules, Python with boto3, CloudWatch metrics |
| 05 | [AWS Elastic Beanstalk: Deploying a Web App](aws/lab-05-elastic-beanstalk/) | Elastic Beanstalk environments, application deployment, load balancer, Auto Scaling group, security groups |
| 06 | [Access Control with IAM: Permissions Boundaries, Groups, and Tags](aws/lab-06-iam-access-control-abac/) | Permissions boundaries, implicit deny, group policies, tag-based access control (ABAC), cloud cost and cleanup |
| 07 | [Working with Amazon EBS: Volumes and Snapshots](aws/lab-07-working-with-ebs/) | EBS volumes, attaching and mounting, ext3 and /etc/fstab, snapshots, restoring a volume from a snapshot |

> An earlier AWS EC2 deployment is part of [Linux Lab 01](https://github.com/willkhot/Linux-Labs/tree/main/lab-01-deploy-linux-servers) in my Linux Labs.

## What I've Been Learning

**Cloud (AWS)**
- Launching a server on EC2, connecting to it, and shutting it down when I'm done
- Using IAM groups and policies so each user only gets the access their job needs
- Building a VPC with public and private subnets and launching a web server inside it
- Monitoring and resizing an EC2 instance, and opening only the port it needs in its security group
- Running code on a schedule with Lambda and EventBridge, with an IAM role deciding what the code is allowed to do
- Deploying a web app with Elastic Beanstalk and finding the load balancer, Auto Scaling group, and instances it runs on
- Controlling access with permissions boundaries, group policies, and tags, and why limiting permissions also keeps cloud costs down
- Adding EBS storage to an instance, backing it up with a snapshot, and restoring the data to a new volume

## Author

**Will Khotsyphom**

[![GitHub](https://img.shields.io/badge/GitHub-willkhot-181717?logo=github)](https://github.com/willkhot)

## Security Note

Screenshots and commands in this repository are sanitized. Account IDs, public IP addresses, hostnames, emails, usernames and keys are redacted or replaced with placeholders. All cloud resources shown were terminated after each lab.
