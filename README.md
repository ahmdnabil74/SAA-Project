# Serverless Task Management API on AWS

## Project Summary

This project implements a cloud-native REST API for managing application data using a fully serverless AWS architecture.

The application separates the frontend, authentication, API layer, business logic, and persistence layer. The main objective is to build an API that can scale automatically while maintaining secure access control, monitoring, and reliable data storage.

### Main AWS Services

* Amazon API Gateway
* AWS Lambda
* Amazon DynamoDB
* Amazon Cognito
* Amazon CloudFront
* Amazon S3
* AWS WAF
* Amazon CloudWatch
* AWS X-Ray
* Amazon SNS
* AWS IAM
* AWS KMS
* Amazon Route 53

---

## Architecture at a Glance

```text
                         Internet Users
                               |
                               v
                         Amazon Route 53
                               |
                               v
                         Amazon CloudFront
                         /              \
                        /                \
                       v                  v
                Amazon S3             AWS WAF
              Frontend Assets             |
                                          v
                                  Amazon API Gateway
                                          |
                                  Cognito Authorizer
                                          |
                                          v
                                    AWS Lambda
                                          |
                                          v
                                  Amazon DynamoDB
                                          |
                                  DynamoDB Streams
                                          |
                                          v
                                    Async Lambda
```

---

## 1. Application Layer

The frontend is delivered as static content through Amazon S3 and Amazon CloudFront.

CloudFront provides the public HTTPS endpoint and distributes static resources through AWS edge locations.

The API is exposed through API Gateway and uses Lambda for application logic, allowing the backend to operate without managing EC2 instances or other servers.

---

## 2. Authentication and Access Control

Amazon Cognito is responsible for application user authentication.

The authentication workflow is:

1. A user signs in through the Cognito user pool.
2. Cognito authenticates the user.
3. Cognito issues the required tokens.
4. The client sends the access token with API requests.
5. API Gateway validates the token before forwarding the request.
6. Only authenticated requests reach the Lambda functions.

This keeps authentication separate from the application logic.

---

## 3. API Design

Amazon API Gateway acts as the entry point for the backend.

The API exposes operations for managing application records:

| Operation        | HTTP Method | Backend |
| ---------------- | ----------- | ------- |
| Create record    | POST        | Lambda  |
| Retrieve records | GET         | Lambda  |
| Modify record    | PUT         | Lambda  |
| Remove record    | DELETE      | Lambda  |

Each operation can be handled independently, which keeps the Lambda functions small and easier to maintain.

---

## 4. Serverless Compute

AWS Lambda contains the application logic.

Instead of maintaining a permanent server, Lambda executes the required function when API Gateway receives a request.

The functions are designed to:

* Validate incoming requests
* Process application logic
* Read or update DynamoDB
* Return HTTP responses
* Generate logs for troubleshooting

IAM execution roles restrict each function to the AWS resources it actually needs.

---

## 5. Data Storage

Amazon DynamoDB is used as the application's primary database.

The table design is based on the application's access patterns rather than using a relational schema.

Additional indexes can be used when the application needs to query records using attributes other than the primary key.

The database configuration can include:

* On-demand capacity
* Point-in-time recovery
* Encryption at rest
* DynamoDB Streams

This allows the database layer to remain managed and automatically scalable.

---

## 6. Event-Driven Processing

DynamoDB Streams records changes made to database items.

For example:

```text
DynamoDB
   |
   | INSERT / MODIFY / REMOVE
   v
DynamoDB Streams
   |
   v
Lambda
   |
   +---- Audit processing
   +---- Notifications
   +---- Additional asynchronous tasks
```

This allows secondary processing to happen independently from the original API request.

---

## 7. Edge Security

AWS WAF is placed at the CloudFront layer to inspect incoming web traffic.

The Web ACL can include managed AWS rules and rate-based protection.

Examples of traffic that can be inspected include:

* SQL injection attempts
* Cross-site scripting patterns
* Excessive request rates
* Other suspicious HTTP requests

CloudFront also provides HTTPS delivery for users accessing the application.

---

## 8. Observability

The architecture uses several AWS monitoring services.

### CloudWatch

CloudWatch is used to monitor application and infrastructure metrics such as:

* Lambda errors
* Lambda duration
* Lambda invocations
* API Gateway errors
* API latency
* DynamoDB throttling

Alarms can be configured to notify administrators when important thresholds are exceeded.

### X-Ray

AWS X-Ray provides request tracing across the serverless components.

A typical request can be visualized as:

```text
Client
  |
  v
API Gateway
  |
  v
Lambda
  |
  v
DynamoDB
```

This helps identify where latency or failures occur during a request.

### SNS

Amazon SNS can be connected to CloudWatch alarms so that operational notifications are delivered to administrators.

---

## 9. Security Architecture

Security is applied at multiple layers instead of relying on a single control.

| Area                           | AWS Implementation       |
| ------------------------------ | ------------------------ |
| Transport                      | HTTPS through CloudFront |
| Authentication                 | Amazon Cognito           |
| API authorization              | API Gateway authorizer   |
| Network/application protection | AWS WAF                  |
| AWS permissions                | IAM roles                |
| Database encryption            | DynamoDB encryption      |
| Object storage                 | S3 encryption            |
| Monitoring                     | CloudWatch and X-Ray     |
| Auditability                   | AWS logging services     |

Lambda functions use dedicated IAM roles with permissions limited to the resources required by each function.

---

## 10. Request Lifecycle

A normal authenticated request follows this path:

```text
1. User sends HTTPS request
          |
2. Route 53 resolves the application domain
          |
3. CloudFront receives the request
          |
4. WAF evaluates the request
          |
5. API Gateway receives the API call
          |
6. Cognito authorizer validates authentication
          |
7. API Gateway invokes Lambda
          |
8. Lambda accesses DynamoDB
          |
9. DynamoDB returns the result
          |
10. Lambda generates the API response
          |
11. Response returns to the client
```

Monitoring and tracing operate alongside this flow.

---

## 11. Availability and Scalability

The architecture avoids traditional server management.

API Gateway provides a managed API endpoint, Lambda automatically handles concurrent executions, and DynamoDB can operate using on-demand capacity.

CloudFront distributes frontend content geographically, while S3 provides durable object storage for the application assets.

As a result, the application can handle variable traffic without maintaining a permanently running application server.

---

## 12. Data Protection

Several AWS capabilities are used to improve data protection:

* DynamoDB point-in-time recovery
* Encryption at rest
* IAM-based access control
* S3 encryption
* CloudWatch monitoring
* AWS audit logging

Where customer-managed encryption keys are required, AWS KMS can be used to control encryption keys and their lifecycle.

---

## 13. Deployment Components

The main infrastructure components required for the solution are:

```text
DNS
 └── Route 53

Delivery
 └── CloudFront

Frontend
 └── S3

Security
 ├── WAF
 ├── Cognito
 ├── IAM
 └── KMS

Backend
 ├── API Gateway
 └── Lambda

Database
 └── DynamoDB
      └── DynamoDB Streams

Operations
 ├── CloudWatch
 ├── X-Ray
 └── SNS
```

---

## 14. What This Project Demonstrates

This implementation brings together several important AWS architecture concepts:

* Building REST APIs without managing servers
* Implementing authentication with Amazon Cognito
* Using Lambda for event-driven application logic
* Designing DynamoDB access patterns
* Protecting public endpoints with AWS WAF
* Delivering frontend content through CloudFront
* Monitoring serverless workloads
* Tracing distributed requests with X-Ray
* Applying IAM least-privilege principles
* Using DynamoDB Streams for asynchronous processing
* Designing for automatic scaling and high availability

---

## AWS Services Used

| AWS Service      | Role in the Solution       |
| ---------------- | -------------------------- |
| Route 53         | DNS management             |
| CloudFront       | Global content delivery    |
| S3               | Frontend object storage    |
| API Gateway      | REST API entry point       |
| Cognito          | User authentication        |
| Lambda           | Application/business logic |
| DynamoDB         | Application database       |
| DynamoDB Streams | Database event source      |
| WAF              | Web traffic protection     |
| CloudWatch       | Metrics, logs and alarms   |
| X-Ray            | Request tracing            |
| SNS              | Operational notifications  |
| IAM              | Access control             |
| KMS              | Encryption key management  |

---

## Final Architecture Goal

The final design provides a managed AWS environment where the application can authenticate users, expose protected API endpoints, execute backend logic on demand, persist application data, and monitor requests without requiring traditional server infrastructure.

The architecture is designed around AWS managed services, security controls, observability, and automatic scaling.
