# AWS ECS (Elastic Container Service) Notes

## 1. Why ECS?

-   ECS is AWS's managed container orchestration service.
-   It runs, scales, and manages Docker containers.
-   Simpler than Kubernetes and tightly integrated with AWS.

### Docker vs ECS

-   Docker builds containers.
-   ECS runs and manages containers.

## ECS Workflow

``` text
Docker Image
    ↓
Amazon ECR
    ↓
ECS Cluster
    ↓
Task Definition
    ↓
ECS Service
    ↓
Target Group
    ↓
Application Load Balancer
    ↓
Users
```

## 2. Amazon ECR

Stores Docker images that ECS pulls when starting containers.

### Steps

1.  Open **ECR**.
2.  Create repository `patientping-ecs`.
3.  Use **View push commands**.
4.  Login, build and push the Docker image.
5.  Verify the image exists.

## 3. ECS Cluster

A logical environment where ECS runs tasks.

**Recommended:** Fargate (serverless).

### Steps

1.  ECS → Clusters.
2.  Create Cluster.
3.  Name: `patientping-ecs`.
4.  Choose **Fargate Only**.
5.  Create.

## 4. IAM Roles

### Execution Role

Used by ECS to: - Pull images from ECR - Send logs to CloudWatch

### Task Role

Used by the running application to access AWS services (SSM, S3, etc.).

Create: - `patientping-ecs-execution-role` - `patientping-ecs-task-role`

Trust entity: `ecs-tasks.amazonaws.com`

## 5. Task Definition

Blueprint for running containers.

Contains: - Image - CPU - Memory - Network - Ports - Logging - IAM Roles

### Steps

1.  ECS → Task Definitions.
2.  Create New.
3.  Configure image and roles.
4.  Register task definition.

## 6. Security Groups

**External SG** - Attached to ALB. - Allows HTTP (80) from Internet.

**Internal SG** - Attached to ECS Tasks. - Allows traffic only from
External SG.

## 7. Application Load Balancer

Receives user traffic and forwards it to ECS tasks.

### Steps

1.  EC2 → Load Balancers.
2.  Create Application Load Balancer.
3.  Internet-facing.
4.  Select public subnets.
5.  Attach External SG.
6.  Configure HTTP listener.
7.  Create.

## 8. Target Group

Defines where the ALB sends traffic.

For Fargate: - Target type = IP - Health check = `/`

Update the ALB listener to forward traffic to the target group.

## 9. CloudWatch Log Groups

Stores application logs.

Create:

`/ecs/patientping-ecs`

Retention: - 7 Days

## 10. ECS Service

Keeps containers running and registers them with the ALB.

### Steps

1.  ECS Cluster → Services.
2.  Create Service.
3.  Task Definition: `patientping-ecs`
4.  Service: `patientping-svc`
5.  Capacity Provider: FARGATE.
6.  Configure networking.
7.  Attach existing ALB and Target Group.
8.  Create.

Verify: - `1/1 Running Tasks`

## Architecture

``` text
Developer
    ↓
Docker Image
    ↓
Amazon ECR
    ↓
ECS Cluster
    ↓
Task Definition
    ↓
ECS Service
    ↓
Target Group
    ↓
Application Load Balancer
    ↓
Users
```
