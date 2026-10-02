# 🚀 AWS Priority Queue Processing with SNS, SQS, Lambda & CloudFormation

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws)
![CloudFormation](https://img.shields.io/badge/AWS-CloudFormation-FF4F8B?logo=amazon-aws)
![SQS](https://img.shields.io/badge/Amazon-SQS-FF4F8B?logo=amazon-aws)
![SNS](https://img.shields.io/badge/Amazon-SNS-FF4F8B?logo=amazon-aws)
![Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900?logo=awslambda)
![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python)

## 📌 Project Overview

This project demonstrates a **priority-based message processing architecture on AWS** using:

* **Amazon SNS** for message publishing
* **Amazon SQS** for reliable message queuing
* **SNS Filter Policies** for priority-based message routing
* **AWS Lambda** for serverless message processing
* **AWS IAM** for access control
* **AWS CloudFormation** for Infrastructure as Code

The architecture separates incoming messages into **High Priority** and **Low Priority** queues automatically based on the `priority` message attribute.

This is a practical example of an **event-driven, serverless AWS architecture** that can be used in applications such as order processing, support ticket systems, payment processing, notifications, and background job processing.

---

## 🏗️ Architecture

```text
                         ┌──────────────────────┐
                         │      Producer        │
                         │   Application/API    │
                         └──────────┬───────────┘
                                    │
                                    │ Publish Message
                                    ▼
                         ┌──────────────────────┐
                         │      Amazon SNS      │
                         │ PriorityQueueTopic   │
                         └──────────┬───────────┘
                                    │
                   ┌────────────────┴────────────────┐
                   │                                 │
             priority=high                     priority=low
                   │                                 │
                   ▼                                 ▼
        ┌────────────────────┐             ┌────────────────────┐
        │   High Priority    │             │    Low Priority    │
        │       SQS          │             │        SQS         │
        │                    │             │                    │
        │ devops-High-       │             │ devops-Low-        │
        │ Priority-Queue     │             │ Priority-Queue    │
        └─────────┬──────────┘             └─────────┬──────────┘
                  │                                  │
                  └────────────────┬─────────────────┘
                                   │
                                   ▼
                         ┌──────────────────────┐
                         │    AWS Lambda        │
                         │ Priority Processing  │
                         └──────────────────────┘
```

---

## 🔄 How the Architecture Works

### 1️⃣ Message Producer

An application publishes a message to the SNS topic:

```text
devops-Priority-Queues-Topic
```

The message contains a `priority` attribute.

Example:

```text
priority = high
```

or

```text
priority = low
```

---

### 2️⃣ Amazon SNS

SNS acts as the **central message distribution layer**.

The topic receives incoming messages and evaluates the subscription filter policies.

```text
SNS Topic
    │
    ├── priority = high → High Priority SQS
    │
    └── priority = low  → Low Priority SQS
```

---

### 3️⃣ SNS Filter Policies

The project uses SNS subscription filtering to route messages.

### High Priority Subscription

```yaml
FilterPolicy:
  priority:
    - high
```

Messages with:

```text
priority = high
```

are delivered to:

```text
devops-High-Priority-Queue
```

### Low Priority Subscription

```yaml
FilterPolicy:
  priority:
    - low
```

Messages with:

```text
priority = low
```

are delivered to:

```text
devops-Low-Priority-Queue
```

This allows the application to separate workloads without requiring additional routing logic in the producer.

---

## 📦 AWS Services Used

| AWS Service            | Purpose                          |
| ---------------------- | -------------------------------- |
| **AWS CloudFormation** | Infrastructure as Code           |
| **Amazon SNS**         | Message publishing and fan-out   |
| **Amazon SQS**         | Reliable message queuing         |
| **AWS Lambda**         | Serverless message processing    |
| **AWS IAM**            | Permissions and access control   |
| **Amazon S3**          | Stores Lambda deployment package |

---

## 🛠️ Infrastructure Created

The CloudFormation stack creates the following resources:

### SQS Queues

```text
devops-High-Priority-Queue
devops-Low-Priority-Queue
```

Both queues have:

```yaml
VisibilityTimeout: 30
```

### SNS Topic

```text
devops-Priority-Queues-Topic
```

### SNS Subscriptions

```text
High Priority → High Priority SQS
Low Priority  → Low Priority SQS
```

### SQS Queue Policies

Queue policies allow the SNS topic to send messages to the corresponding SQS queues.

The source is restricted using:

```yaml
aws:SourceArn
```

to the project's SNS topic.

### IAM Role

Lambda uses:

```text
lambda_execution_role
```

with permissions for:

* AWS Lambda basic execution
* Amazon SQS
* Amazon SNS

### Lambda Function

```text
devops-priorities-queue-function
```

Runtime:

```text
Python 3.9
```

Handler:

```text
index.lambda_handler
```

---

# 📂 Project Structure

Recommended repository structure:

```text
priority-queue-processing/
│
├── cloudformation/
│   └── priority-queue-stack.yaml
│
├── lambda/
│   └── index.py
│
├── screenshots/
│   ├── cloudformation-stack.png
│   ├── sns-topic.png
│   ├── sqs-queues.png
│   └── lambda-function.png
│
└── README.md
```

---

# 🚀 Deployment

## Prerequisites

Make sure you have:

* AWS Account
* AWS CLI
* IAM permissions for CloudFormation, SNS, SQS, Lambda, IAM and S3
* An S3 bucket containing the Lambda deployment package
* `function-code.zip`

Configure AWS CLI:

```bash
aws configure
```

Verify credentials:

```bash
aws sts get-caller-identity
```

---

## 1️⃣ Upload Lambda Code to S3

Create the Lambda deployment package:

```bash
zip function-code.zip index.py
```

Upload it to S3:

```bash
aws s3 cp function-code.zip s3://YOUR-BUCKET/
```

Update the CloudFormation template with your bucket and object:

```yaml
Code:
  S3Bucket: YOUR-BUCKET
  S3Key: function-code.zip
```

---

## 2️⃣ Deploy CloudFormation Stack

Run:

```bash
aws cloudformation create-stack \
  --stack-name priority-queue-processing \
  --template-body file://priority-queue-stack.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

Check stack status:

```bash
aws cloudformation describe-stacks \
  --stack-name priority-queue-processing
```

---

## 3️⃣ Verify Resources

List SQS queues:

```bash
aws sqs list-queues
```

List SNS topics:

```bash
aws sns list-topics
```

List Lambda functions:

```bash
aws lambda list-functions
```

---

# 🧪 Testing the Priority Routing

After deployment, publish messages to the SNS topic using message attributes.

## High Priority Message

```bash
aws sns publish \
  --topic-arn YOUR_SNS_TOPIC_ARN \
  --message "Critical production issue" \
  --message-attributes '{"priority":{"DataType":"String","StringValue":"high"}}'
```

Expected flow:

```text
Producer
   ↓
SNS
   ↓
priority=high
   ↓
High Priority SQS
```

---

## Low Priority Message

```bash
aws sns publish \
  --topic-arn YOUR_SNS_TOPIC_ARN \
  --message "General information" \
  --message-attributes '{"priority":{"DataType":"String","StringValue":"low"}}'
```

Expected flow:

```text
Producer
   ↓
SNS
   ↓
priority=low
   ↓
Low Priority SQS
```

---

# 🔍 Verify Messages

Retrieve messages from the High Priority queue:

```bash
aws sqs receive-message \
  --queue-url HIGH_PRIORITY_QUEUE_URL
```

Retrieve messages from the Low Priority queue:

```bash
aws sqs receive-message \
  --queue-url LOW_PRIORITY_QUEUE_URL
```

This allows you to verify that SNS filtering is routing messages correctly.

---

# 🔐 Security Considerations

The project demonstrates controlled communication between SNS and SQS using queue policies.

The queue policy uses:

```yaml
Condition:
  ArnEquals:
    aws:SourceArn: !Ref PriorityQueueTopic
```

This ensures that the message source is restricted to the configured SNS topic.

### Production Improvements

For a production environment, consider:

* Using least-privilege IAM policies
* Avoiding `AmazonSQSFullAccess`
* Avoiding `AmazonSNSFullAccess`
* Using customer-managed IAM policies
* Encrypting SQS queues with AWS KMS
* Enabling SNS encryption
* Using CloudWatch monitoring and alarms
* Adding Dead Letter Queues
* Configuring Lambda event source mappings
* Using environment-specific parameters
* Avoiding hard-coded resource names where possible

---

# 💡 Real-World Use Cases

This architecture can be used for:

### 🛒 E-Commerce

```text
Order Received
      ↓
     SNS
      ↓
 ┌────┴─────┐
 │          │
High       Low
Priority   Priority
Orders     Orders
```

### 🎫 Support Ticket Processing

```text
Critical Ticket → High Priority Queue
Normal Ticket   → Low Priority Queue
```

### 💳 Payment Processing

```text
Payment Failure → High Priority Queue
Normal Payment  → Processing Queue
```

### 🔔 Notification Systems

```text
Critical Alert → High Priority
Marketing      → Low Priority
```

---

# 🎯 Key DevOps Concepts Demonstrated

This project demonstrates practical knowledge of:

* Infrastructure as Code
* AWS CloudFormation
* Event-driven architecture
* Serverless architecture
* Message queues
* Pub/Sub architecture
* SNS → SQS integration
* SNS subscription filtering
* IAM roles and policies
* S3-based Lambda deployment
* AWS CLI
* Cloud automation
* Asynchronous processing
* Decoupled architecture

---

# 🧠 Interview Explanation

You can explain the project like this:

> "I built a priority-based message processing architecture using AWS CloudFormation. An application publishes messages to an SNS topic with a priority attribute. SNS uses subscription filter policies to route high-priority messages to one SQS queue and low-priority messages to another. This decouples the producer from the consumers and allows different workloads to be processed independently. I also provisioned the complete infrastructure using CloudFormation and configured IAM permissions for the Lambda processing layer."

---

# 📊 Architecture Benefits

| Feature         | Benefit                                       |
| --------------- | --------------------------------------------- |
| SNS             | Decouples producers from consumers            |
| SQS             | Provides reliable asynchronous processing     |
| Filter Policies | Routes messages based on priority             |
| Lambda          | Enables serverless processing                 |
| CloudFormation  | Provides repeatable infrastructure deployment |
| IAM             | Controls access to AWS resources              |
| S3              | Stores Lambda deployment artifacts            |

---

# 🧹 Cleanup

To remove the CloudFormation stack:

```bash
aws cloudformation delete-stack \
  --stack-name priority-queue-processing
```

Verify deletion:

```bash
aws cloudformation describe-stacks \
  --stack-name priority-queue-processing
```

---

# 📚 Learning Outcomes

After completing this project, you should understand:

```text
SNS
 │
 ├── Subscription Filter Policy
 │
 ├── High Priority → SQS
 │
 └── Low Priority  → SQS
                         │
                         ▼
                       Lambda
```

You also gain practical experience with **AWS Infrastructure as Code, serverless architecture, asynchronous processing, IAM, AWS CLI, and event-driven system design**.

---

## 👨‍💻 Author

**Subodh Kumar**

☁️ AWS Cloud Support Engineer | AWS Certified

🔗 **LinkedIn:**
https://www.linkedin.com/in/subodh-kumar-aws-certified/

🐙 **GitHub:**
https://github.com/SubodhK143

---

## ⭐ If you found this project useful

Give this repository a ⭐ and feel free to explore the implementation.

**Built with AWS ☁️ | CloudFormation 🚀 | SNS 📢 | SQS 📬 | Lambda ⚡**
