# Week 8 — Infrastructure as Code with Terraform

## Overview

This lab focuses on Infrastructure as Code (IaC) using Terraform to provision and manage AWS infrastructure.

Instead of manually creating networking resources through the AWS Management Console, Terraform is used to define and deploy the infrastructure through configuration files.

The goal is to reproduce the VPC architecture from Week 5 while learning Terraform state management, infrastructure drift detection, and resource lifecycle management.

## Technologies Used

- **AWS VPC** — Virtual networking environment
- **Terraform** — Infrastructure as Code provisioning
- **AWS NAT Gateway** — Outbound internet access for private subnets
- **AWS Internet Gateway** — Internet connectivity for public subnets
- **AWS Route Tables** — Network traffic routing
- **Git & GitHub** — Version control

## Architecture

The infrastructure consists of:

- **1 VPC:** `training-vpc-tf` (`10.1.0.0/16`)
- **2 Availability Zones:** `us-east-1a` and `us-east-1b`
- **2 Public Subnets:** `10.1.1.0/24` and `10.1.2.0/24`
- **2 Private Subnets:** `10.1.101.0/24` and `10.1.102.0/24`
- **1 Internet Gateway**
- **1 NAT Gateway** shared across private subnets
- **Route tables** for public and private network traffic

Public subnets route internet-bound traffic through the Internet Gateway. Private subnets use the NAT Gateway for outbound internet connectivity without allowing unsolicited inbound connections from the internet.

## Lab 8.1 — Deploy AWS VPC Using Terraform

### Step 1: Initialize Terraform

```bash
terraform init
```

Downloads the required AWS provider and VPC module and initializes the working directory.

### Step 2: Review the Execution Plan

```bash
terraform plan
```

Displays the infrastructure changes Terraform intends to make before applying them.

The plan allows engineers to review resources that will be created, updated, or destroyed, reducing the risk of unintended infrastructure changes.

### Step 3: Deploy Infrastructure

```bash
terraform apply
```

Applies the configuration after confirmation and provisions the AWS networking resources.

### Step 4: Verify Deployment

Verify the resources in the AWS VPC Console:

- VPC `training-vpc-tf` exists
- Public and private subnets are created
- Internet Gateway is attached
- NAT Gateway is provisioned
- Route tables contain the expected routes

## Lab 8.2 — Terraform State and Drift Detection

### Terraform State

Terraform maintains a state file (`terraform.tfstate`) that maps resources defined in configuration files to real infrastructure.

State allows Terraform to determine which resources it manages and what changes may be required.

```bash
terraform state list
```

Lists the resources currently tracked by Terraform.

### Infrastructure Drift

Infrastructure drift occurs when actual infrastructure differs from its intended configuration or previously recorded state.

For example, manually modifying or deleting a Terraform-managed resource in the AWS Console can introduce drift.

To investigate:

```bash
terraform plan
terraform state list
```

Compare Terraform's proposed changes and tracked resources against the actual AWS environment.

Possible recovery actions include:

- Restore the intended configuration through Terraform.
- Import an existing resource into Terraform state when appropriate.
- Remove a stale state entry only after confirming that Terraform should no longer manage the resource.

**Key lesson:** Infrastructure managed through Terraform should be modified through Terraform rather than manually through the AWS Console.

## Terraform Lifecycle Commands

| Command | Purpose |
|---|---|
| `terraform init` | Initializes the Terraform working directory |
| `terraform plan` | Previews infrastructure changes |
| `terraform apply` | Creates or updates infrastructure |
| `terraform state list` | Lists resources tracked in state |
| `terraform destroy` | Removes Terraform-managed infrastructure |

## Cleanup

```bash
terraform destroy
```

Review the resources scheduled for deletion and confirm the operation.

Verify that the VPC and associated resources have been removed from AWS to avoid unnecessary charges.

## Key Takeaways

1. **Declarative Infrastructure:** Terraform defines the desired infrastructure state rather than requiring engineers to manually execute each provisioning step.
2. **Repeatability:** Infrastructure can be recreated consistently using version-controlled configuration files.
3. **State Management:** Terraform state tracks managed infrastructure and supports change planning.
4. **Drift Detection:** Terraform plans help identify differences between configuration and deployed resources.
5. **Infrastructure Lifecycle:** Terraform supports provisioning, updating, and destroying infrastructure through a consistent workflow.
6. **Security:** Terraform state and provider credentials must be protected and should not be committed to public repositories.

## Conclusion

Week 8 introduces Infrastructure as Code as an alternative to manually provisioning AWS infrastructure.

Using Terraform to define a multi-AZ VPC demonstrates how cloud infrastructure can be managed consistently through reusable configuration, version control, and state management.

These practices are foundational for Cloud Engineering, DevOps, and Infrastructure Engineering roles.
