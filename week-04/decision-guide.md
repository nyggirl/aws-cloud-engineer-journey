# Week 4 — AWS Storage & Database Decision Guide

| AWS Service | Storage / Database Type | Use this when... |
|---|---|---|
| Amazon S3 | Object Storage | You need to store files, static website assets, backups, or archived data with high durability and scalability. |
| Amazon EBS | Block Storage | You need persistent disk storage for an EC2 instance, such as an operating system volume or application data. |
| Amazon EFS | Shared File Storage | Multiple EC2 instances need simultaneous access to the same shared files. |
| Amazon RDS | Relational Database | Your application needs structured relational data, SQL queries, and managed database operations. |
| Amazon DynamoDB | NoSQL Database | Your application needs fast, scalable key-value or document storage without managing database servers. |

## Week 4 Hands-On Experience

**Lab 4.1 — Amazon S3**

Worked with an S3 bucket and lifecycle management for transitioning objects to Glacier Flexible Retrieval after 30 days.

**Lab 4.2 — Amazon RDS MySQL**

Created a MySQL RDS instance, connected from EC2, created the `training_data` database, imported sample property data, verified its schema, used synthetic listing IDs, and performed SQL aggregate queries.

**Lab 4.3 — Amazon DynamoDB**

Created a `Properties` table with `L_ListingID` as the partition key and on-demand billing. Inserted a sanitized property record and successfully retrieved it using AWS CLI.