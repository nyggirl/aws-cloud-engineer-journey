# Week 7 — Serverless & Event-Driven Architecture on AWS

## Overview

Week 7 focuses on migrating PropertyLite's property lookup functionality from an EC2-hosted Flask application to a serverless architecture using Amazon API Gateway, AWS Lambda, and Amazon DynamoDB.

The lab also introduces asynchronous messaging using Amazon Simple Queue Service (SQS), dead-letter queues (DLQs), and event-driven application design.

## Architecture

```text
                 Client
                   |
             HTTP GET Request
                   |
             API Gateway
        GET /properties/{id}
                   |
                Lambda
             get-property
                   |
               DynamoDB
            Properties Table
                   |
             JSON Response
                   |
                 Client
```

Additional messaging architecture:

```text
             Message Producer
                    |
                    v
             Amazon SQS
             property-queue
                    |
          Processing / Retry
                    |
             Failed Messages
                    |
                    v
             Dead-Letter Queue
               property-dlq
```

## AWS Services

| Service | Purpose |
|---|---|
| AWS Lambda | Executes property lookup logic without managing EC2 servers |
| Amazon API Gateway | Exposes the property lookup API |
| Amazon DynamoDB | Stores property records |
| Amazon SQS | Supports asynchronous message processing |
| SQS Dead-Letter Queue | Stores messages that cannot be processed successfully |
| AWS IAM | Restricts Lambda access to the required DynamoDB table |
| Amazon CloudWatch Logs | Captures Lambda execution logs and errors |

## Lab 7.1 — Serverless Property Lookup

### 1. DynamoDB Table

Reuse the `Properties` table from Week 4, or recreate it using cleaned property records.

The table uses `L_ListingID` as the partition key.

Example property record:

```json
{
  "L_ListingID": "R100234",
  "L_City": "Sacramento",
  "L_SystemPrice": "450000",
  "L_Status": "Active"
}
```

### 2. AWS Lambda Function

Create a Lambda function named `get-property` using Python 3.12.

The function receives the listing ID from API Gateway, queries DynamoDB, and returns the property record as JSON.

```python
import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Properties')

def lambda_handler(event, context):
    listing_id = event['pathParameters']['id']
    resp = table.get_item(
        Key={'L_ListingID': listing_id}
    )

    return {
        'statusCode': 200,
        'body': json.dumps(resp.get('Item', {}))
    }
```

### 3. IAM Permissions

Configure a least-privilege IAM execution role allowing the Lambda function to read from the `Properties` DynamoDB table.

The policy should allow only the required DynamoDB operations and be scoped to the specific table ARN.

The function also requires appropriate CloudWatch Logs permissions for execution logging.

### 4. API Gateway

Create an HTTP API with a Lambda integration.

Configure the following route:

```text
GET /properties/{id}
```

Deploy the API and test it with:

```bash
curl https://<api-id>.execute-api.us-east-1.amazonaws.com/properties/R100234
```

Expected result: a JSON response containing the requested property record.

## Lab 7.2 — Amazon SQS and Dead-Letter Queue

Create two SQS queues:

- `property-queue` — Main message queue
- `property-dlq` — Dead-letter queue

Configure a redrive policy with a maximum receive count of `3`.

Example message:

```json
{
  "PropertyId": "P1002"
}
```

Send a test message:

```bash
aws sqs send-message \
  --queue-url <property-queue-url> \
  --message-body '{"PropertyId":"P1002"}'
```

Receive messages:

```bash
aws sqs receive-message \
  --queue-url <property-queue-url>
```

The goal is to understand asynchronous communication, message retries, and how dead-letter queues isolate repeatedly failed messages.

Note: Receiving a message alone does not demonstrate DLQ redrive. That requires repeated unsuccessful processing or receives without deletion, subject to the configured visibility timeout.

## Lab 7.3 — API Gateway Error Troubleshooting

Investigate an API Gateway endpoint returning:

```json
{
  "message": "Internal Server Error"
}
```

Use CloudWatch Logs to identify the underlying Lambda execution error.

Common causes include:

| Error | Possible cause |
|---|---|
| AccessDeniedException | Lambda lacks DynamoDB permissions |
| KeyError | Incorrect event path parameter |
| ResourceNotFoundException | DynamoDB table name or region is incorrect |

Troubleshooting workflow:

1. Open AWS Lambda.
2. Select the `get-property` function.
3. Open Monitor → View CloudWatch logs.
4. Inspect the most recent invocation.
5. Identify the exception and root cause.
6. Correct the configuration or code.
7. Redeploy and retest the API.

## Key Learning Objectives

- Understand Lambda's event-driven execution model and cold starts
- Migrate application functionality from EC2 to Lambda
- Build REST-style HTTP endpoints using API Gateway
- Read DynamoDB records using boto3
- Apply least-privilege IAM policies
- Understand SQS messaging and dead-letter queues
- Troubleshoot Lambda failures using CloudWatch Logs

## Validation Checklist

- [ ] DynamoDB `Properties` table contains sample records
- [ ] Lambda function successfully retrieves a property
- [ ] API Gateway returns the expected JSON response
- [ ] Lambda IAM policy is restricted to the required table
- [ ] SQS main queue and DLQ are configured
- [ ] Test message can be sent and received
- [ ] CloudWatch Logs can be used to diagnose Lambda errors

## Deliverables

The `week-07` directory should contain:

- `README.md` — Architecture and implementation documentation
- `lambda_function.py` — Property lookup Lambda source code
- `screenshots/` — AWS configuration and validation evidence
- `api-tests/` — curl or Postman test examples

## Conclusion

This lab explores how an existing EC2-hosted application can be redesigned using managed serverless services. It demonstrates API-based data access, least-privilege permissions, asynchronous messaging, and cloud-native troubleshooting.

Implementation and end-to-end validation should be completed before marking the lab as finished.