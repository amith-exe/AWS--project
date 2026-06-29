# AWS Lambda Notes

# What is AWS Lambda?

AWS Lambda is a **serverless compute service** that lets you run code without managing servers.

Instead of creating EC2 instances or ECS containers, you simply upload your code, define a trigger, and AWS automatically runs it whenever needed.

AWS manages:

* Server provisioning
* Scaling
* Operating system updates
* Runtime management
* Monitoring
* High availability

You only focus on writing your application code.

---

# Why Do We Need Lambda?

Traditional applications require a server that runs **24/7**.

Example:

```text
User Request
      ↓
EC2 Server (Always Running)
      ↓
Application
```

Even if nobody uses your application, you still pay for the server.

Suppose your API receives only **100 requests per day**.

Your EC2 instance still remains running all day waiting for requests.

This wastes resources and increases cost.

---

## Lambda's Solution

Lambda follows an **event-driven** execution model.

Your code only runs when an event occurs.

```text
Event
   ↓
Lambda Starts
   ↓
Runs Code
   ↓
Returns Response
   ↓
Stops
```

No requests?

No compute cost.

---

# Serverless Computing

"Serverless" **does not mean there are no servers.**

AWS still uses servers internally.

The difference is:

* You don't create servers.
* You don't manage servers.
* You don't patch operating systems.
* You don't scale infrastructure.

AWS handles everything.

You only deploy your code.

---

# Lambda vs EC2 vs ECS

| Feature           | Lambda                   | EC2              | ECS        |
| ----------------- | ------------------------ | ---------------- | ---------- |
| Manage Servers    | ❌ No                     | ✅ Yes            | Partially  |
| Auto Scaling      | ✅ Automatic              | Manual/ASG       | Automatic  |
| Pay When Idle     | ❌ No                     | ✅ Yes            | ✅ Yes      |
| Event Driven      | ✅ Yes                    | ❌ No             | ❌ No       |
| Long Running Apps | ❌ No                     | ✅ Yes            | ✅ Yes      |
| Best For          | APIs, Events, Automation | Traditional Apps | Containers |

---

# How Lambda Works

When an event occurs:

```text
API Request
S3 Upload
SNS Message
CloudWatch Event
      ↓
Lambda Function
      ↓
Execute Code
      ↓
Return Response
```

Once execution finishes:

```text
Container Stops
```

You are charged only for the execution time.

---

# Lambda Execution Lifecycle

## 1. Cold Start

When the Lambda function is invoked for the first time (or after being idle), AWS must:

* Allocate compute resources
* Create a container
* Load the runtime
* Load your code

```text
Request
    ↓
Create Container
    ↓
Load Runtime
    ↓
Load Code
    ↓
Execute
```

This introduces additional latency.

Typical delay:

* Python: a few hundred milliseconds
* Java/.NET: usually longer

---

## 2. Warm Start

If another request arrives shortly afterward, AWS reuses the existing execution environment.

```text
Request
     ↓
Existing Container
     ↓
Execute Immediately
```

Warm starts are much faster than cold starts.

---

## 3. Automatic Scaling

Suppose:

```text
1 Request
```

AWS starts:

```text
1 Lambda Container
```

Now suppose:

```text
10,000 Requests
```

AWS automatically launches many Lambda execution environments simultaneously.

```text
Incoming Requests
        ↓
Lambda
 ├── Container 1
 ├── Container 2
 ├── Container 3
 ├── Container 4
 └── ...
```

Scaling is automatic.

No manual intervention is required.

---

# Hands-On: Create and Deploy patientping-ip-function

This section walks you through creating a Lambda function that returns the caller's IP address.

---

## Cost Check

* Lambda functions **do not cost anything when not running**
* Storage for the deployment package costs approximately:

```text
$0.0000000034 per GB per hour
```

For a small function like this, the cost is effectively negligible.

---

## Step 1: Create IAM Role

In the AWS Console:

1. Navigate to **IAM → Roles**
2. Click **Create role**
3. Configure:

* Trusted entity type: **AWS service**
* Use case: **Lambda**

4. Attach permissions policy:

* **AWSLambdaBasicExecutionRole**

5. Set role name:

```text
patientping-lambda-role
```

6. Click **Create role**

---

## Step 2: Create Lambda Function

In the AWS Console:

1. Navigate to **Lambda**
2. Click **Create function**
3. Choose:

* **Author from scratch**

4. Configure:

* Function name:

```text
patientping-ip-function
```

* Runtime:

```text
Python 3.x (latest available, e.g., Python 3.14)
```

5. Under **Permissions**:

* Select **Use an existing role**
* Choose:

```text
patientping-lambda-role
```

(You may need to expand **More settings** or select **Custom execution role**)

6. Leave all other settings as default
7. Click **Create function**

---

## Step 3: Add Function Code

In the **Code** tab:

1. Open `lambda_function.py`
2. Replace the default code with:

```python
def lambda_handler(event, context):
    request_context = event.get("requestContext", {})
    identity = request_context.get("identity", {})
    ip_address = identity.get("sourceIp")

    if not ip_address:
        headers = event.get("headers", {})
        ip_address = headers.get("X-Forwarded-For") or headers.get("x-forwarded-for")

    if not ip_address:
        ip_address = "unknown"

    print(f"Received IP: {ip_address}")

    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "text/plain",
        },
        "body": f"Your IP address is: {ip_address}",
    }
```

3. Ensure the handler remains:

```text
lambda_function.lambda_handler
```

4. Click **Deploy**

---

# When Should You Use Lambda?

Lambda is an excellent choice for **event-driven** applications.

Examples include:

### File Processing

```text
User Uploads Image
        ↓
S3 Bucket
        ↓
Lambda
        ↓
Resize Image
```

---

### REST APIs

```text
API Gateway
      ↓
Lambda
      ↓
Database
```

---

### Scheduled Jobs

Instead of keeping a server running:

```text
EC2
24 Hours
```

Run a Lambda function every hour:

```text
CloudWatch Schedule
        ↓
Lambda
```

---

### Background Tasks

Examples:

* Send Emails
* Generate PDFs
* Resize Images
* Data Validation
* Notifications

---

### Prototypes & MVPs

For small applications:

* Fast deployment
* Minimal infrastructure
* Very low cost

---

# When Should You NOT Use Lambda?

Lambda is **not** suitable for every workload.

Avoid Lambda for:

### Long Running Applications

Maximum execution time:

```text
15 Minutes
```

If your application runs for hours, use EC2 or ECS.

---

### Stateful Applications

Each Lambda execution is independent.

You cannot rely on memory between invocations.

Store state in:

* DynamoDB
* RDS
* Redis
* S3

---

### Large Applications

Deployment package limit:

```text
250 MB (Unzipped)
```

Large machine learning applications or heavy software stacks may exceed this limit.

---

### Constant High Traffic

If your application runs continuously 24/7:

```text
Millions of Requests
```

Lambda can become more expensive than EC2 or ECS.

---

### Custom Operating System Requirements

Lambda provides managed runtime environments.

If your application requires:

* Custom kernel modules
* Custom OS packages
* Full operating system access

Use EC2 or ECS instead.

---

# Advantages of Lambda

* No server management
* Automatic scaling
* Pay only for execution
* Built-in high availability
* Easy AWS integration
* Fast deployment
* Excellent for event-driven workloads

---

# Limitations

* Maximum execution time: **15 minutes**
* Cold start latency
* Stateless execution
* Deployment package size limits
* Limited runtime customization

---

# Real-World Examples

## Example 1: Image Processing

```text
User Uploads Image
        ↓
Amazon S3
        ↓
Lambda
        ↓
Resize Image
        ↓
Store Processed Image
```

---

## Example 2: API Backend

```text
Client
    ↓
API Gateway
    ↓
Lambda
    ↓
Amazon RDS
```

---

## Example 3: Email Notifications

```text
User Registers
        ↓
SNS
        ↓
Lambda
        ↓
Send Welcome Email
```

---

## Example 4: Scheduled Cleanup

```text
EventBridge Schedule
        ↓
Every Midnight
        ↓
Lambda
        ↓
Delete Old Files
```

---

# Cost Model

Lambda pricing is based on:

* Number of requests
* Execution duration
* Allocated memory

Unlike EC2 or ECS:

```text
Idle Time = $0 Compute Cost
```

You only pay while your function executes.

---

# Key Takeaways

### AWS Lambda

Runs code without managing servers.

### Serverless

AWS manages infrastructure while you focus on code.

### Event Driven

Functions execute only when triggered.

### Cold Start

First invocation creates a new execution environment.

### Warm Start

Subsequent invocations reuse an existing environment.

### Auto Scaling

AWS automatically scales to handle concurrent requests.

### Best Use Cases

* APIs
* File processing
* Scheduled jobs
* Background processing
* Event-driven applications

### Avoid Lambda For

* Long-running tasks
* Stateful applications
* Large software packages
* Always-running services

---

# Lambda Architecture

```text
            User Request
                  │
                  ▼
          API Gateway / S3 / SNS
                  │
                  ▼
            AWS Lambda Function
                  │
        ┌─────────┴─────────┐
        │                   │
     Read Data          Process Logic
        │                   │
        └─────────┬─────────┘
                  ▼
        RDS / DynamoDB / S3
                  │
                  ▼
            Return Response
```
---

# Testing Lambda Functions

After deploying a Lambda function, you should test it to verify that your code behaves as expected before exposing it to real users.

Lambda allows you to invoke functions directly from the **AWS Console**, **AWS CLI**, or through services like **API Gateway**, **S3**, **SNS**, etc.

For development, testing directly from the console is the easiest option.

---

# How Lambda Testing Works

Unlike calling a REST API, testing a Lambda function directly **does not send an actual HTTP request**.

Instead, AWS executes your function using a **mock event** (JSON payload) that you provide.

```text
Test Event (JSON)
        │
        ▼
AWS Lambda
        │
        ▼
Your Function
        │
        ▼
Response
```

Think of the test event as **fake input** that simulates what another AWS service (such as API Gateway) would normally send.

For example:

* API Gateway sends HTTP request information.
* S3 sends file upload information.
* SNS sends notification details.
* EventBridge sends scheduled event information.

During testing, you manually provide this information.

---

# Cold Start vs Warm Start During Testing

When testing Lambda, you'll notice different execution times.

## Cold Start

The first invocation (or after the function has been idle for several minutes) is called a **Cold Start**.

AWS must:

1. Allocate compute resources
2. Start a container
3. Load the runtime
4. Load your function code
5. Execute the function

```text
Invoke Function
      ↓
Create Container
      ↓
Load Runtime
      ↓
Load Function
      ↓
Execute
```

Typical latency:

* Python: **200–500 ms**
* Java/.NET: Usually longer

CloudWatch Logs will include:

```text
INIT_START
Init Duration
```

These indicate the cold start overhead.

---

## Warm Start

If another request arrives shortly after the first one, AWS reuses the existing execution environment.

```text
Invoke Function
      ↓
Existing Container
      ↓
Execute Immediately
```

Warm starts are much faster, often completing in only a few milliseconds.

CloudWatch Logs generally won't show `INIT_START` or `Init Duration` for warm executions.

---

# Why Cold Starts Matter

Cold starts can slightly increase response time.

For most applications, this delay is perfectly acceptable.

Examples:

* Scheduled jobs
* Background processing
* File processing
* Internal APIs

However, for latency-sensitive applications such as real-time APIs or gaming backends, developers often reduce cold starts using:

* Provisioned Concurrency
* Scheduled "keep warm" invocations

---

# Assignment: Testing patientping-ip-function

## Cost

Lambda includes a generous Free Tier.

* 1 Million requests per month
* Testing consumes only a few requests

---

## Step 1: Open the Function

1. Open **AWS Console**
2. Navigate to **Lambda**
3. Select:

```text
patientping-ip-function
```

4. Open the **Test** tab.

---

## Step 2: Create the First Test Event

Click **Create new event**.

Name:

```text
test-ip
```

Payload:

```json
{
  "requestContext": {
    "identity": {
      "sourceIp": "8.8.8.8"
    }
  },
  "headers": {}
}
```

Click **Save**.

---

## Step 3: Run the Test

Click **Test**.

Expected response:

```json
{
  "statusCode": 200,
  "body": "Your IP address is: 8.8.8.8"
}
```

This verifies that the function correctly reads the client's IP address from:

```text
requestContext.identity.sourceIp
```

---

## Step 4: Create a Second Test Event

Sometimes the client IP is provided through the HTTP header:

```text
X-Forwarded-For
```

Create another test event.

Name:

```text
test-xff
```

Payload:

```json
{
  "requestContext": {
    "identity": {}
  },
  "headers": {
    "X-Forwarded-For": "198.51.100.99"
  }
}
```

Save the event.

---

## Step 5: Run the Second Test

Click **Test** again.

Expected response:

```json
{
  "statusCode": 200,
  "body": "Your IP address is: 198.51.100.99"
}
```

This confirms that the function correctly falls back to reading the IP address from the **X-Forwarded-For** header when `sourceIp` is unavailable.

---

# Understanding the Test Events

## Test Event 1

```text
requestContext.identity.sourceIp
        │
        ▼
8.8.8.8
```

The function reads the IP directly from the request context.

---

## Test Event 2

```text
requestContext.identity
        │
        ▼
Empty
        │
        ▼
headers
        │
        ▼
X-Forwarded-For
        │
        ▼
198.51.100.99
```

The function uses the forwarded header as a fallback.

---

# Testing Workflow

```text
Create Test Event
        │
        ▼
Run Lambda
        │
        ▼
Inspect Response
        │
        ▼
Verify Output
        │
        ▼
Repeat with Different Inputs
```

---

# Best Practices

* Test with multiple input scenarios.
* Verify both successful and error responses.
* Monitor execution time to identify cold starts.
* Check CloudWatch Logs for debugging.
* Use realistic event payloads that match the service triggering your Lambda.

---

# Key Takeaways

* Lambda testing executes your function using **mock JSON events**.
* No real HTTP request is sent during console testing.
* Cold starts occur on the first execution after idle time.
* Warm starts reuse existing execution environments and are much faster.
* Testing with different payloads ensures your function handles multiple real-world scenarios correctly.
<img width="742" height="262" alt="image" src="https://github.com/user-attachments/assets/d531aa1b-174f-4d58-9530-214804ff9768" />
<img width="705" height="600" alt="image" src="https://github.com/user-attachments/assets/d99a3523-744e-4513-b733-05b84bf908c4" />
<img width="747" height="341" alt="image" src="https://github.com/user-attachments/assets/e8c0d9e1-2c46-48bd-a944-b86b067085f3" />
---

# API Gateway with AWS Lambda

Until now, we've been testing our Lambda function directly from the AWS Console using mock events.

While this is useful for development, real users cannot invoke Lambda directly.

To expose a Lambda function over the internet, AWS provides **API Gateway**.

API Gateway acts as the public entry point that receives HTTP requests and forwards them to your Lambda function.

---

# What is API Gateway?

Amazon API Gateway is a fully managed service that allows you to create, publish, secure, and manage REST or HTTP APIs.

It acts as a bridge between clients and backend services such as:

* AWS Lambda
* EC2
* ECS
* Other HTTP services

Instead of users calling Lambda directly, they communicate with API Gateway.

---

# Why Do We Need API Gateway?

Without API Gateway:

```text
Client
   │
   ▼
Lambda Function

❌ Not publicly accessible
```

A Lambda function has no public URL by default.

To make it accessible over HTTP, API Gateway sits in front of it.

```text
Client
      │
      ▼
API Gateway
      │
      ▼
Lambda Function
```

API Gateway receives the HTTP request, invokes the Lambda function, and returns the response to the client.

---

# API Gateway Request Flow

```text
Browser / Client
        │
HTTP GET /
        │
        ▼
API Gateway
        │
Invoke Lambda
        ▼
Lambda Function
        │
Execute Code
        ▼
Return Response
        │
        ▼
API Gateway
        │
        ▼
Client
```

---

# HTTP API vs REST API

AWS offers two API Gateway types.

| Feature     | HTTP API         | REST API        |
| ----------- | ---------------- | --------------- |
| Performance | Faster           | Slightly slower |
| Cost        | Lower            | Higher          |
| Features    | Basic            | Advanced        |
| Best For    | Most modern APIs | Enterprise APIs |

For simple Lambda applications, **HTTP API** is recommended because it is faster and cheaper.

---

# Cost

API Gateway pricing is based on usage.

Approximate pricing:

```text
HTTP API

≈ $1 per 1 Million Requests
```

For learning and small projects, the cost is usually only a few cents.

---

# Assignment: Create an HTTP API

We will expose our Lambda function over HTTP.

API Name:

```text
patientping-ip-api
```

---

# Step 1: Open API Gateway

1. Open the **AWS Console**
2. Search for **API Gateway**
3. Click **Create API**

---

# Step 2: Choose HTTP API

Under **HTTP API**, click:

```text
Build
```

HTTP API is the recommended option for Lambda-based APIs.

---

# Step 3: Configure the API

Enter:

```text
API Name

patientping-ip-api
```

---

# Step 4: Add Lambda Integration

Under **Integrations**:

1. Click **Add**
2. Integration Type:

```text
Lambda
```

3. Select:

```text
patientping-ip-function
```

Click **Next**.

---

# Step 5: Configure Routes

Create a route.

Method:

```text
GET
```

Resource Path:

```text
/
```

Integration:

```text
patientping-ip-function
```

This means:

```text
GET /
      │
      ▼
Lambda Function
```

Click **Next**.

---

# Step 6: Configure Stage

Leave the default settings.

Stage Name:

```text
$default
```

Auto Deploy:

```text
Enabled
```

Why?

Whenever you update your API, AWS automatically deploys the latest version without requiring manual deployment.

Click **Next**.

---

# Step 7: Create the API

Review all settings.

Click:

```text
Create
```

AWS creates:

* HTTP API
* Default Stage
* Public Endpoint
* Lambda Integration

---

# Step 8: Find the Invoke URL

Navigate to:

```text
API Gateway

↓

Deploy

↓

Stages

↓

$default
```

You'll see something similar to:

```text
https://abc123xyz.execute-api.ap-south-1.amazonaws.com
```

This is your **Invoke URL**.

---

# Step 9: Test the API

Open the Invoke URL in your browser.

Example:

```text
https://abc123xyz.execute-api.ap-south-1.amazonaws.com
```

Expected response:

```text
Your IP address is: 203.xxx.xxx.xxx
```

Unlike the earlier console tests, this response contains your **real public IP address**, because the request is coming from your browser through API Gateway.

---

# Complete Request Flow

```text
Browser
     │
HTTP GET
     │
     ▼
API Gateway
     │
Invoke
     ▼
Lambda
     │
Read Client IP
     │
Generate Response
     ▼
API Gateway
     │
Return Response
     ▼
Browser
```

---

# Console Navigation Summary

```text
AWS Console

↓

API Gateway

↓

Create API

↓

HTTP API

↓

Add Lambda Integration

↓

Configure Route (GET /)

↓

Create API

↓

Deploy

↓

Stages

↓

$default

↓

Copy Invoke URL

↓

Open in Browser
```

---

# Advantages of API Gateway

* Creates public HTTP endpoints for Lambda.
* Automatically scales with traffic.
* Supports authentication and authorization.
* Integrates with many AWS services.
* Supports HTTPS by default.
* Works seamlessly with Lambda.

---

# Real-World Examples

API Gateway is commonly used for:

* REST APIs
* Mobile applications
* Serverless web applications
* Backend APIs
* Microservices
* Webhooks
* IoT devices

---

# Key Takeaways

* Lambda cannot be accessed directly over HTTP.
* API Gateway provides a public HTTP endpoint for Lambda.
* HTTP API is faster and cheaper than REST API for most use cases.
* API Gateway receives HTTP requests, invokes Lambda, and returns the response.
* The **Invoke URL** is the public endpoint users access.
* The **$default** stage automatically deploys API updates when Auto Deploy is enabled.
* Testing the Invoke URL in a browser executes your Lambda with a real HTTP request.
# CloudWatch Logs for AWS Lambda

## What is CloudWatch Logs?

When a Lambda function executes, AWS automatically captures everything your function writes to **stdout** and **stderr** (such as Python's `print()` statements) and stores it in **Amazon CloudWatch Logs**.

CloudWatch Logs is AWS's centralized logging service used to monitor, debug, and troubleshoot applications.

Instead of connecting to a running server to view logs, you simply open CloudWatch and inspect the logs generated by your Lambda function.

---

# Why Do We Need CloudWatch Logs?

Unlike EC2, Lambda functions are **serverless**.

There is:

* No terminal
* No SSH access
* No running server to inspect

If something goes wrong, the only way to understand what happened is through logs.

CloudWatch Logs helps you:

* Debug application errors
* View `print()` output
* Measure execution time
* Monitor memory usage
* Identify failed invocations
* Analyze application behavior

---

# How Lambda Logging Works

Whenever a Lambda function executes:

```text
Client Request
      │
      ▼
 Lambda Function
      │
      ├── print()
      ├── Exceptions
      ├── Warnings
      └── Runtime Information
              │
              ▼
      CloudWatch Logs
```

AWS automatically creates a log group for every Lambda function.

Each execution creates a **Log Stream**, and every message inside that execution becomes a **Log Event**.

---

# CloudWatch Log Structure

```text
CloudWatch Logs
      │
      ▼
Log Group
      │
      ▼
Log Stream
      │
      ▼
Log Events
```

### Log Group

A Log Group represents logs for an application or AWS resource.

Example:

```text
/aws/lambda/patientping-ip-function
```

---

### Log Stream

Each Lambda execution environment creates its own Log Stream.

Example:

```text
2026/07/05/[$LATEST]abc123xyz...
```

---

### Log Events

Individual log messages.

Examples:

```text
START

Received IP: 8.8.8.8

END

REPORT
```

---

# What Lambda Automatically Logs

Every invocation automatically generates three system log entries.

## START

Marks the beginning of execution.

Contains:

* Request ID
* Function Version

Example:

```text
START RequestId: xxxxx Version: $LATEST
```

---

## END

Marks the completion of execution.

Example:

```text
END RequestId: xxxxx
```

---

## REPORT

Shows execution statistics.

Example:

```text
REPORT RequestId: xxxxx

Duration: 42 ms

Billed Duration: 100 ms

Memory Size: 128 MB

Max Memory Used: 58 MB
```

This information is extremely useful for performance tuning.

---

# Understanding the REPORT Line

The REPORT line tells you:

### Duration

Actual execution time.

Example:

```text
42 ms
```

---

### Billed Duration

AWS rounds execution time for billing purposes.

---

### Memory Size

The memory allocated to the Lambda function.

Example:

```text
128 MB
```

---

### Max Memory Used

Peak memory consumed during execution.

If this value approaches the configured memory limit, you should increase the function's memory allocation.

---

# Viewing Lambda Logs

## Step 1

Open:

```text
AWS Console

↓

CloudWatch
```

---

## Step 2

Navigate to:

```text
Logs

↓

Log Management
```

---

## Step 3

Open the Lambda Log Group.

Example:

```text
/aws/lambda/patientping-ip-function
```

---

## Step 4

Open the latest Log Stream.

You'll see entries similar to:

```text
START RequestId...

Received IP: 8.8.8.8

END RequestId...

REPORT Duration...
```

---

# Assignment: View Lambda Logs

## Step 1

Open:

```text
AWS Console

↓

CloudWatch

↓

Log Management
```

---

## Step 2

Open the log group:

```text
/aws/lambda/patientping-ip-function
```

---

## Step 3

Open the most recent Log Stream.

---

## Step 4

Verify the logs contain:

```text
START

Received IP: ...

END

REPORT
```

---

# Common Lambda Use Cases

Lambda is far more powerful than simply serving HTTP APIs.

It can automatically respond to events from many AWS services.

---

# 1. S3 File Processing

Whenever a file is uploaded to S3, Lambda can process it automatically.

Example:

```text
User Uploads Image
        │
        ▼
Amazon S3
        │
S3 Event
        ▼
Lambda
        │
Resize Image
        │
        ▼
Thumbnail Bucket
```

Typical tasks:

* Image resizing
* Video transcoding
* PDF generation
* Virus scanning
* Data validation

---

# 2. Scheduled Tasks

Lambda can run automatically using **Amazon EventBridge** (formerly CloudWatch Events).

Example:

```text
EventBridge Schedule
        │
Every Night at 2 AM
        ▼
Lambda
        │
Backup Database
        ▼
Amazon S3
```

Typical scheduled jobs:

* Database backups
* Data cleanup
* Report generation
* Sending reminder emails
* Scheduled notifications

---

# 3. Webhooks

Lambda can respond to external services.

Example:

```text
GitHub Actions
        │
Webhook
        ▼
API Gateway
        │
        ▼
Lambda
        │
Send Discord Message
        ▼
Discord API
```

Common integrations:

* Discord Bots
* Slack Bots
* Microsoft Teams
* GitHub Actions
* Stripe Webhooks

---

# When Should You Use Lambda?

Lambda is an excellent choice when:

* Workloads are event-driven.
* Traffic is unpredictable.
* Execution completes in under 15 minutes.
* Each request is independent (stateless).
* You want to avoid paying for idle servers.

Typical applications:

* REST APIs
* File processing
* Background jobs
* Automation scripts
* Scheduled tasks
* Notifications

---

# When Should You Avoid Lambda?

Lambda is not suitable when:

* Applications run continuously.
* Tasks take longer than 15 minutes.
* Persistent state is required.
* Cold start latency is unacceptable.
* Large software dependencies exceed deployment limits.
* You need full operating system customization.

In these cases, consider:

* Amazon EC2
* Amazon ECS
* Amazon EKS

---

# Best Practices

* Use `print()` (or proper logging libraries) for debugging.
* Always review CloudWatch Logs after deployment.
* Monitor the REPORT line for execution time and memory usage.
* Keep Lambda functions stateless.
* Store persistent data in DynamoDB, RDS, or S3.
* Use EventBridge for scheduled jobs instead of dedicated servers.
* Monitor logs regularly to detect performance issues early.

---

# Key Takeaways

* CloudWatch Logs automatically stores Lambda output.
* Every Lambda function gets its own Log Group.
* Each execution creates a Log Stream.
* Lambda automatically logs **START**, **END**, and **REPORT** information.
* The REPORT line helps analyze performance and memory usage.
* CloudWatch Logs is the primary tool for debugging Lambda functions.
* Lambda integrates seamlessly with many AWS services such as S3, EventBridge, API Gateway, SNS, and webhooks.
* Lambda is best suited for event-driven, short-running, and stateless applications.
