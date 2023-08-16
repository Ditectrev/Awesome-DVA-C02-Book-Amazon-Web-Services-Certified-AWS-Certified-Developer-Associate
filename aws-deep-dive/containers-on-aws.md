# Containers on AWS

## What is Docker?

- Docker is a software development platform to deploy apps
- Apps are packaged in **containers** that can be run on any OS
- Apps run the same, regardless of where they’re run
- Any machine
- No compatibility issues
- Predictable behavior
- Less work
- Easier to maintain and deploy
- Works with any language, any OS, any technology
- Use cases: microservices architecture, lift-and-shift apps from on-premises to the AWS cloud,...

### Where are Docker images stored?

- Docker images are stored in Docker Repositories
- **Docker Hub** (<https://hub.docker.com>)
  - Public repository
  - Find base images for many technologies or OS (e.g., Ubuntu, MySQL, ...)
- **Amazon ECR (Amazon Elastic Container Registry)**
  - Private repository
  - Public repository (Amazon ECR Public Gallery <https://gallery.ecr.aws>)

### Docker Containers Management on AWS

- **Amazon Elastic Container Service (Amazon ECS)**
  - Amazon’s own container platform
- **Amazon Elastic Kubernetes Service (Amazon EKS)**
  - Amazon’s managed Kubernetes (open source)
- **AWS Fargate**
  - Amazon’s own Serverless container platform
  - Works with ECS and with EKS
- **Amazon ECR**
  - Store container images

## Amazon ECS - EC2 Launch Type

- ECS = Elastic Container Service
- Launch Docker containers on AWS = Launch **ECS Task**s on **ECS Clusters**
- **EC2 Launch Type: you must provision & maintain the infrastructure (the EC2 instances)**
- Each EC2 Instance must run the ECS Agent to register in the ECS Cluster
- AWS takes care of starting / stopping containers

![EC2 Launch Type](./../images/ecs-ec2-launch-type.png)

## Amazon ECS – Fargate LaunchType

- Launch Docker containers on AWS
- **You do not provision the infrastructure (no EC2 instances to manage)**
- **It’s all Serverless!**
- You just create task definitions
- AWS just runs ECSTasks for you based on the CPU / RAM you need
- To scale, just increase the number of tasks. Simple - no more EC2 instances

![ECS – Fargate](./../images/ecs-fargate.png)

## Amazon ECS – IAM Roles for ECS

- EC2 Instance Profile (EC2 Launch Type only):
  - Used by the **ECS agent**
  - Makes API calls to ECS service
  - Send container logs to CloudWatch Logs
  - Pull Docker image from ECR
  - Reference sensitive data in Secrets Manager or SSM Parameter Store
- ECSTask Role:
  - Allows each task to have a specific role
  - Use different roles for the different ECS Services you run
  - Task Role is defined in the **task definition**

![IAM Roles](./../images/ecs-iam-roles.png)

## Amazon ECS – Load Balancer Integrations

- **Application Load Balancer** supported and works for most use cases
- **Network Load Balancer** recommended only for high throughput / high performance use cases, or to pair it with AWS Private Link
- **Classic Load Balancer** supported but not recommended (no advanced features – no Fargate)

## Amazon ECS – Data Volumes (EFS)

- Mount EFS file systems onto ECS tasks
- Works for both **EC2** and **Fargate** launch types
- Tasks running in any AZ will share the same data in the EFS file system
- **Fargate + EFS = Serverless**
- Use cases: persistent multi-AZ shared storage for your containers
- Note:
  - Amazon S3 cannot be mounted as a file system

## ECS Service Auto Scaling

- Automatically increase/decrease the desired number of ECS tasks
- Amazon ECS Auto Scaling uses **AWS Application Auto Scaling**
  - ECS Service Average CPU Utilization
  - ECS Service Average Memory Utilization - Scale on RAM
  - ALB Request Count Per Target – metric coming from the ALB
- **Target Tracking** – scale based on target value for a specific CloudWatch metric
- **Step Scaling** – scale based on a specified CloudWatch Alarm
- **Scheduled Scaling** – scale based on a specified date/time (predictable changes)
- ECS Service Auto Scaling (task level) ≠ EC2 Auto Scaling (EC2 instance level)
- Fargate Auto Scaling is much easier to setup (because **Serverless**)

### EC2 Launch Type – Auto Scaling EC2 Instances

- Accommodate ECS Service Scaling by adding underlying EC2 Instances
- **Auto Scaling Group Scaling**
  - Scale your ASG based on CPU Utilization
  - Add EC2 instances over time
- **ECS Cluster Capacity Provider**
  - Used to automatically provision and scale the infrastructure for your ECSTasks
  - Capacity Provider paired with an Auto Scaling Group
  - Add EC2 Instances when you’re missing capacity (CPU, RAM...)

### ECS Scaling – Service CPU Usage Example

![ECS Scaling](./../images/ecs-scaling.png)

## ECS Rolling Updates

- When updating from v1 to v2, we can control how many tasks can be started and stopped, and in which order
  - Minimum healthy percent: 100
  - Minimum percent: 200

![ECS Rolling Updates](./../images/ecs-rolling-updates.png)

### ECS Rolling Update – Min 50%, Max 100%

- Starting number of tasks: 4

![ECS Rolling Update](./../images/ecs-rolling-updates-1.png)

### ECS Rolling Update – Min 100%, Max 150%

- Starting number of tasks: 4

![Alt text](./../images/ecs-rolling-updates-2.png)
