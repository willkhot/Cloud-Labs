# Lab 12 — Scale and Load Balance a Web App: ALB and EC2 Auto Scaling

## Overview

This lab turned a single web server into an app that spreads traffic across servers and grows on its own when it gets busy. I started with one server, **Web Server 1**, and saved its disk as an **AMI**. Then I put an **Application Load Balancer** in front of the app and built an **Auto Scaling group** that launches copies of that server in two Availability Zones.

To test it, I ran the app's built-in load test. CPU went to 100%, a **CloudWatch** alarm fired, and Auto Scaling went from 2 instances to 4. At the end I terminated the original server, since the Auto Scaling group didn't need it anymore.

## Objectives

- Create an AMI from a running instance
- Create a target group and an internet-facing Application Load Balancer
- Create a launch template and an Auto Scaling group with a target tracking policy
- Check that the load balancer sends traffic to healthy instances
- Push CPU up and watch CloudWatch trigger a scale-out
- Terminate the original server

## Tools & Technologies

| Category | Tools |
|---|---|
| Cloud platform | Amazon Web Services (AWS) |
| Services | Amazon EC2, Elastic Load Balancing (ALB), EC2 Auto Scaling, Amazon CloudWatch, Amazon VPC |
| Objects | AMI, target group, load balancer, launch template, Auto Scaling group, scaling policy, CloudWatch alarms |
| Region | US East (N. Virginia) `us-east-1`, AZs `us-east-1a` and `us-east-1b` |

## How the Pieces Fit

| Piece | Job in this lab |
|---|---|
| AMI (`WebServerAMI`) | A saved copy of Web Server 1's disk, so new servers start with the app already installed |
| Launch template (`LabConfig`) | The recipe for a new server: AMI, instance type, key pair, security group |
| Auto Scaling group | Keeps between 2 and 6 servers running, spread across two private subnets |
| Target group (`LabGroup`) | The list of servers the load balancer can send traffic to, plus their health checks |
| Load balancer (`LabELB`) | One public address that spreads requests across the healthy servers |
| Scaling policy + CloudWatch alarms | Watch average CPU and add or remove servers to stay near the target |

## Architecture

This is the architecture.

![ALB in the public subnets, Auto Scaling group in the private subnets, built from an AMI and launch template and scaled by CloudWatch](screenshots/00-architecture.png)

---

## Task 1 — Create an AMI from Web Server 1

The lab started with two running instances in Public Subnet 1: **Web Server 1**, which runs the app, and a **Bastion Host**.

![Starting instances](screenshots/01-starting-instances.png)

**Web Server 1 → Actions → Image and templates → Create image.**

| Setting | Value |
|---|---|
| Image name | `WebServerAMI` |
| Description | `Lab AMI for Web Server` |

![Create image settings](screenshots/02-create-image.png)

AWS started building the AMI from the instance's 8 GiB root volume.

![AMI being created](screenshots/03-ami-creating.png)

---

## Task 2 — Create the Load Balancer

### 2.1 Target group

**Target Groups → Create target group.** I chose **Instances** as the target type, named it `LabGroup`, and kept HTTP on port 80 in the Lab VPC. I didn't register any targets yet, because the Auto Scaling group would add its own instances later.

![Create target group](screenshots/04-create-target-group.png)

![LabGroup created with no targets](screenshots/05-target-group-created.png)

### 2.2 Application Load Balancer

**Load Balancers → Create load balancer → Application Load Balancer.**

| Setting | Value |
|---|---|
| Name | `LabELB` |
| Scheme | Internet-facing |
| Network mapping | Lab VPC, Public Subnet 1 (`us-east-1a`) and Public Subnet 2 (`us-east-1b`) |
| Security group | Web Security Group only (I removed `default`) |
| Listener | HTTP:80, forward to `LabGroup` |

An ALB works at layer 7 (HTTP), so it can route by the content of a request. It needs subnets in at least two AZs, and it puts a node in each one.

![LabELB name and internet-facing scheme](screenshots/06-alb-basic-config.png)

![Both public subnets selected](screenshots/07-alb-network-mapping.png)

![Web Security Group and listener forwarding to LabGroup](screenshots/08-alb-sg-listener.png)

---

## Task 3 — Launch Template and Auto Scaling Group

### 3.1 Launch template

**Launch Templates → Create launch template.**

| Setting | Value |
|---|---|
| Name | `LabConfig` |
| Auto Scaling guidance | Checked, so the form shows only the settings Auto Scaling uses |
| AMI | My AMIs → `WebServerAMI` |
| Instance type | `t2.micro` |
| Key pair | `vockey` |
| Security group | Web Security Group |
| Detailed CloudWatch monitoring | Enabled (1-minute metrics, so scaling reacts faster) |

![Launch template name and Auto Scaling guidance](screenshots/09-launch-template-name.png)

![WebServerAMI and t2.micro](screenshots/10-launch-template-ami-type.png)

![vockey and Web Security Group](screenshots/11-launch-template-key-sg.png)

![Detailed CloudWatch monitoring enabled](screenshots/12-launch-template-monitoring.png)

### 3.2 Auto Scaling group

From the template, **Actions → Create Auto Scaling group.**

| Step | Setting |
|---|---|
| Name and template | `Lab Auto Scaling Group`, using `LabConfig` |
| Network | Lab VPC, **Private Subnet 1** and **Private Subnet 2** |
| Load balancing | Attach to the existing target group `LabGroup` |
| Monitoring | Group metrics collection in CloudWatch enabled |
| Group size | Desired 2, minimum 2, maximum 6 |
| Scaling policy | Target tracking, `LabScalingPolicy`, average CPU utilization, target 60 |
| Tag | `Name = Lab Instance`, applied to new instances |

The app servers go in the **private** subnets. Users never reach them directly. Only the load balancer in the public subnets talks to them.

![ASG name and launch template](screenshots/13-asg-name-template.png)

![Private subnets in both AZs](screenshots/14-asg-private-subnets.png)

![Attached to LabGroup](screenshots/15-asg-attach-target-group.png)

![Group metrics collection](screenshots/16-asg-group-metrics.png)

![Min 2, max 6, target tracking on CPU at 60](screenshots/17-asg-size-scaling-policy.png)

![Name tag for new instances](screenshots/18-asg-tags.png)

---

## Task 4 — Check the Load Balancer

Right after the group was created, Auto Scaling launched two **Lab Instance** servers, one in each AZ.

![Two Lab Instances launching](screenshots/19-lab-instances-launched.png)

In `LabGroup → Targets`, both instances passed their health checks and showed as **Healthy**. The load balancer only sends traffic to healthy targets.

![Both targets healthy](screenshots/20-targets-healthy.png)

I copied the load balancer's DNS name from its details page and opened it in a browser.

![LabELB details](screenshots/21-load-balancer-dns.png)

The app loaded through the load balancer. The page shows which instance and AZ answered the request, and it's one of the new Lab Instances, not Web Server 1.

![App served through the ALB](screenshots/22-app-through-alb.png)

---

## Task 5 — Test Auto Scaling

### 5.1 The alarms

The target tracking policy created two CloudWatch alarms by itself:

| Alarm | Condition | What it does |
|---|---|---|
| AlarmHigh | CPU above the target for 3 minutes | Adds instances |
| AlarmLow | CPU well below the target for 15 minutes | Removes instances |

![Two alarms created by the policy](screenshots/23-cloudwatch-alarms.png)

I edited `LabScalingPolicy` and lowered the target from 60 to **50**. The alarms updated to match: AlarmHigh at CPU > 50, AlarmLow at CPU < 45.

![Target value changed to 50](screenshots/24-edit-target-50.png)

![Alarms updated for the new target](screenshots/25-alarms-target-50.png)

### 5.2 Generate load

I clicked **Load Test** in the app. The page refreshes itself, and each refresh can land on a different instance, so all of them get busy. CPU hit 100%.

![Load test running at 100% CPU](screenshots/26-load-test.png)

After a few minutes above the threshold, **AlarmHigh** changed to **In alarm**.

![AlarmHigh in alarm](screenshots/27-alarm-high-in-alarm.png)

Auto Scaling responded by launching more servers. There were now **4 Lab Instances**, two in each AZ, along with Web Server 1 and the Bastion Host.

![Scaled out to 4 Lab Instances](screenshots/28-scaled-out-instances.png)

---

## Task 6 — Terminate Web Server 1

Web Server 1 was only needed to make the AMI. The Auto Scaling group runs the app now, so I selected it and chose **Instance state → Terminate (delete) instance**.

![Terminating Web Server 1](screenshots/29-terminate-web-server-1.png)

---

## What I Learned

- An AMI captures a server's disk so I can launch identical copies. That's what makes scaling out possible.
- A launch template stores everything a new instance needs, and the Auto Scaling group uses it every time it adds a server
- The load balancer is the only public entry point. App servers stay in private subnets, and the ALB only sends traffic to targets that pass health checks.
- Spreading instances across two AZs means one AZ can fail and the app keeps running
- A target tracking policy is the simplest kind of scaling. I pick a metric and a target, and AWS creates the CloudWatch alarms and adjusts the instance count.
- Detailed monitoring and group metrics (1-minute data) let Auto Scaling react faster than the 5-minute default
- Once the group is running, the original server isn't needed anymore. Copies get replaced, so no single server matters.

---

> **Note:** Screenshots are from a temporary AWS lab account. The AWS account ID, my lab session username, instance, AMI, launch template, VPC, subnet, and security group IDs, load balancer and target group ARNs, the load balancer's DNS name, and public IP addresses have been redacted. The environment was shut down after I finished.

[← Back to AWS Labs](../)
