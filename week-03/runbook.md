# Week 3 — AWS EC2 Compute & EBS

## Overview

This week focused on deploying and managing an application on Amazon EC2, configuring network security, troubleshooting Linux server initialization, and managing EBS storage and snapshots.

I deployed the PropertyLite Flask REST API on an Amazon Linux 2023 EC2 instance, verified its endpoints from both the instance and my local machine, investigated a failed user data bootstrap process, and created an EBS snapshot before cleaning up the resources.

## 1. Architecture

**Architecture:**

Mac (Local Machine) → Internet → EC2 Security Group → Amazon EC2 (Amazon Linux 2023) → Flask API (Port 8080) → Local CSV Dataset

**AWS Configuration:**

| Component             | Configuration                             |
| --------------------- | ----------------------------------------- |
| Region                | us-east-1 (N. Virginia)                   |
| EC2 instance          | property-api-01                           |
| Instance type         | t3.micro                                  |
| AMI                   | Amazon Linux 2023                         |
| Root storage          | 8 GiB gp3 EBS                             |
| SSH                   | TCP 22, restricted to my public IP        |
| Application           | TCP 8080, accessible publicly for testing |
| Authentication        | RSA SSH key pair                          |
| Application framework | Python Flask 3.0.3                        |

## 2. EC2 Instance Deployment

I launched an EC2 instance using Amazon Linux 2023 and the t3.micro instance type.

I created an SSH key pair named `training-key` and configured its file permissions:

```bash
chmod 400 ~/.ssh/training-key.pem
```

I created a security group named `property-api-sg` with two inbound rules:

| Type       | Port | Source    | Purpose                      |
| ---------- | ---- | --------- | ---------------------------- |
| SSH        | 22   | My IP /32 | Secure remote administration |
| Custom TCP | 8080 | 0.0.0.0/0 | Public API testing           |

The SSH rule followed the principle of least privilege by restricting administrative access to my public IP address.

Port 8080 was temporarily exposed to the internet for this training lab. In a production environment, I would restrict access and place the application behind an appropriate load balancer or reverse proxy with HTTPS.

## 3. User Data Bootstrapping

I configured EC2 user data to automate application setup during the instance's first boot.

The bootstrap process was designed to:

1. Install Python and pip.
2. Create the application directory at `/opt/property-api`.
3. Create a Python virtual environment.
4. Install Flask 3.0.3.
5. Generate the Flask application and sample CSV dataset.
6. Start the application on port 8080.

The script used Amazon Linux 2023's `dnf` package manager and a Python virtual environment to isolate application dependencies.

### Bootstrap Failure and Troubleshooting

After launching the instance, my initial API health check failed:

```bash
curl http://localhost:8080/health
```

The command returned a connection error, and `/opt/property-api` did not exist.

I investigated the EC2 initialization logs:

```bash
sudo tail -80 /var/log/cloud-init-output.log
```

The logs showed that the `scripts-user` module had failed.

I then inspected the user data script:

```bash
sudo head -n 3 /var/lib/cloud/instance/scripts/part-001 | cat -A
```

The output revealed an unexpected blank line before the `#!/bin/bash` shebang.

This prevented the script from being recognized correctly as a shell script during initialization.

I corrected the script formatting and completed the application setup. After restarting the application, the health endpoint responded successfully.

**Lesson learned:** EC2 user data scripts must be formatted correctly, and cloud-init logs are essential for troubleshooting failed instance initialization.

## 4. PropertyLite API Verification

After fixing the bootstrap issue, I verified that the Flask application was running on port 8080.

### Health check

```bash
curl http://localhost:8080/health
```

Response:

```json
{ "status": "ok" }
```

### List properties

```bash
curl http://localhost:8080/properties
```

The endpoint returned the sample property dataset containing three listings.

### Retrieve a specific property

```bash
curl http://localhost:8080/properties/R100234
```

The response contained the Sacramento property listing with ID `R100234`.

### External API testing

I also tested the application from my Mac using the EC2 instance's public IP address:

```bash
curl http://<EC2-PUBLIC-IP>:8080/health
curl http://<EC2-PUBLIC-IP>:8080/properties/R100234
```

Both requests succeeded, confirming that the application was accessible through the EC2 security group.

## 5. EBS Snapshot Practice

The EC2 instance used an 8 GiB gp3 EBS root volume.

I navigated to:

**EC2 → Volumes → Select root volume → Actions → Create snapshot**

I created a snapshot with the description:

`Week 3 - PropertyLite EC2 root volume backup`

I verified that the snapshot reached:

- Progress: 100%
- Snapshot status: Completed

### What is an EBS snapshot?

An EBS snapshot is a point-in-time backup of an EBS volume.

A new EBS volume can be created from the snapshot to restore the saved block-level data.

Snapshots are incremental after the initial snapshot, helping reduce storage requirements for subsequent backups.

## 6. SSH Connectivity Troubleshooting

I connected to the EC2 instance using:

```bash
ssh -o ConnectTimeout=5 \
  -i ~/.ssh/training-key.pem \
  ec2-user@<EC2-PUBLIC-IP>
```

I verified that my local public IP matched the SSH security group rule.

```bash
curl https://checkip.amazonaws.com
```

The security group allowed SSH access only from my public IP using a `/32` CIDR block.

I confirmed that the SSH connection worked successfully.

### Troubleshooting an SSH Timeout

If an SSH connection times out, I would check:

1. Whether the EC2 instance is running.
2. Whether the public IP address is correct.
3. Whether the security group allows TCP port 22 from my current IP.
4. Whether the subnet route table provides a route to an internet gateway.
5. Whether network ACLs permit the required inbound and outbound traffic.

**Lesson learned:** The security group's "My IP" setting captures the public IP at configuration time. It does not automatically update when the client's IP changes.

## 7. Security Groups vs. Network ACLs

| Feature        | Security Groups                               | Network ACLs                 |
| -------------- | --------------------------------------------- | ---------------------------- |
| Scope          | Network interface / resource                  | Subnet                       |
| State          | Stateful                                      | Stateless                    |
| Rules          | Allow only                                    | Allow and deny               |
| Return traffic | Automatically allowed for tracked connections | Must be explicitly permitted |
| Evaluation     | All applicable rules                          | Ordered rule numbers         |

Security groups act as virtual firewalls for EC2 instances, while NACLs provide an additional network filtering layer at the subnet boundary.

## 8. Resource Cleanup

After completing the labs, I terminated the EC2 instance `property-api-01`.

I verified that:

- The EC2 instance entered the Terminated state.
- The associated EBS root volume was deleted.
- The EBS snapshot was manually deleted.
- No EBS volumes or owned snapshots remained in the selected region.

This cleanup prevents unnecessary ongoing charges from the lab's EC2 and EBS resources.

## 9. Key Takeaways

Through this lab, I gained practical experience with:

- **Amazon EC2:** Launching, configuring, accessing, and terminating Linux virtual machines.
- **AMIs and instance types:** Selecting an operating system image and appropriate compute capacity.
- **Security groups:** Applying least-privilege network access rules.
- **Linux administration:** Using SSH, file permissions, logs, and terminal commands.
- **EC2 user data and cloud-init:** Automating initialization and troubleshooting bootstrap failures.
- **EBS:** Managing persistent block storage and point-in-time snapshots.
- **Network troubleshooting:** Verifying public IP addresses, ports, and SSH connectivity.
- **Cost management:** Cleaning up compute and storage resources after testing.

## 10. Conclusion

Week 3 strengthened my understanding of AWS compute infrastructure and practical Linux server management.

The most valuable part was troubleshooting the failed bootstrap process. Instead of simply relaunching the instance, I investigated cloud-init logs, identified a script formatting issue, and verified the application after fixing it.

This exercise reinforced the importance of systematic troubleshooting, secure network configuration, infrastructure automation, and resource lifecycle management.
