# Week 6 — High Availability & Auto Scaling on AWS

## Overview

In Week 6, I built a high-availability infrastructure environment for the PropertyLite Flask API using Amazon EC2, Application Load Balancer (ALB), and EC2 Auto Scaling.

The goal was to deploy the application across multiple Availability Zones, distribute incoming HTTP requests, and automatically adjust EC2 capacity based on CPU utilization.

## Architecture

```text
                  Internet
                     |
          Application Load Balancer
              property-api-alb
                     |
               Target Group
              property-api-tg
                HTTP :8080
                     |
          EC2 Auto Scaling Group
              property-api-asg
                 /       \
                /         \
          EC2 Instance   EC2 Instance
          us-east-1a     us-east-1b
                \         /
                 Private Subnets
                       |
                 training-vpc
```

## AWS Services Used

| Service | Purpose |
|---|---|
| Amazon EC2 | Hosts the Flask API application |
| Application Load Balancer | Distributes HTTP requests across EC2 instances |
| EC2 Auto Scaling | Manages instance capacity and scaling |
| Amazon VPC | Provides isolated networking |
| Security Groups | Controls traffic between ALB and EC2 |
| Amazon CloudWatch | Provides CPU metrics for scaling decisions |
| AWS IAM | Manages EC2 permissions |
| AWS Systems Manager | Supports instance administration and troubleshooting |

## Implementation

### 1. EC2 Launch Template

Created `property-api-template` using Amazon Linux 2023 and `t3.micro` instances.

Configured EC2 user data to install Python and Flask, create the PropertyLite application and sample property dataset, and register a systemd service.

The application exposes three HTTP endpoints:

- `GET /health` — Application health check
- `GET /properties` — Returns property listings, optionally filtered by city
- `GET /properties/{listing_id}` — Returns a specific property listing

The application is configured to listen on port `8080`.

### 2. Application Load Balancer

Created an internet-facing Application Load Balancer named `property-api-alb` across two public subnets in `us-east-1a` and `us-east-1b`.

Created the target group `property-api-tg` with the following configuration:

| Setting | Value |
|---|---|
| Target type | Instance |
| Protocol | HTTP |
| Port | 8080 |
| Health check path | /health |
| Healthy threshold | 5 |
| Unhealthy threshold | 2 |
| Health check interval | 30 seconds |
| Timeout | 5 seconds |
| Success code | 200 |

Configured security groups to permit application traffic from the ALB security group to EC2 instances on port `8080`.

### 3. EC2 Auto Scaling Group

Created `property-api-asg` using the EC2 launch template.

Deployed the group across two private subnets in separate Availability Zones.

Configured capacity settings:

| Setting | Value |
|---|---|
| Desired capacity | 2 |
| Minimum capacity | 1 |
| Maximum capacity | 4 |
| Instance type | t3.micro |
| Availability Zones | us-east-1a, us-east-1b |

Attached the Auto Scaling group to `property-api-tg`.

### 4. Automatic Scaling Policy

Configured a target tracking scaling policy named `property-api-cpu-scaling`.

- Metric: Average CPU utilization
- Target utilization: 50%
- Instance warmup: 300 seconds
- Scale-out and scale-in enabled

The policy is designed to adjust capacity automatically in response to changing CPU utilization.

### 5. IAM and Troubleshooting

Reviewed the existing `training-ec2-ssm-role` IAM role with AWS Systems Manager and CloudWatch Agent permissions.

Investigated Systems Manager connectivity and ALB target health to troubleshoot the application deployment.

## Validation and Troubleshooting

The Auto Scaling group successfully launched two EC2 instances in separate Availability Zones. Both instances reached the `InService` lifecycle state and passed their EC2 status checks.

However, the ALB target group reported unhealthy targets due to health check request timeouts. Systems Manager connectivity was also unavailable during troubleshooting.

These results demonstrate successful EC2 provisioning and Auto Scaling group configuration, but do not yet confirm successful end-to-end API availability.

## Key Learnings

- How EC2 Auto Scaling maintains desired capacity across Availability Zones
- How ALBs distribute incoming requests through target groups
- How security groups restrict communication between infrastructure components
- How CloudWatch metrics support automatic scaling policies
- Why EC2 status checks and application-level health checks are different
- How IAM permissions and private subnet connectivity affect Systems Manager access

## Next Steps

- Resolve EC2 application startup and ALB health check issues
- Verify successful requests through the ALB DNS endpoint
- Test scaling behavior under simulated CPU load
- Capture screenshots of healthy targets and scaling activities

## Conclusion

This lab provided practical experience building a multi-AZ EC2 deployment with load balancing, automatic scaling, network isolation, and IAM-based administration. It also introduced real-world troubleshooting of application health and private network connectivity.