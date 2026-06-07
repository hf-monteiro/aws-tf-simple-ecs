# ECS/RDS Environment Lab

A Terraform lab environment showing ECS Fargate with multiple services, Aurora PostgreSQL, ECR, S3, SQS, VPC routing, security groups, and Secrets Manager integration.

## Architecture

```mermaid
flowchart TD
    Internet["Internet"] --> ALB1["ALB\nService 01"] & ALB2["ALB\nService 02"] & ALB3["ALB\nService 03"]

    subgraph VPC["VPC 172.42.0.0/16"]
        subgraph Public["Public Subnets (1a · 1b · 1c)"]
            ALB1
            ALB2
            ALB3
            NAT["NAT Gateway"]
        end

        subgraph Private["Private Subnets (1a · 1b · 1c)"]
            SVC1["ECS Fargate\nService 01\n(reads S3)"]
            SVC2["ECS Fargate\nService 02\n(reads SQS)"]
            SVC3["ECS Fargate\nService 03\n(reads S3)"]

            RDS1["Aurora PostgreSQL\nService 01 DB"]
            RDS2["Aurora PostgreSQL\nService 02 DB"]
            RDS3["Aurora PostgreSQL\nService 03 DB"]
        end
    end

    ALB1 --> SVC1
    ALB2 --> SVC2
    ALB3 --> SVC3

    SVC1 --> RDS1
    SVC2 --> RDS2
    SVC3 --> RDS3

    SVC1 -- "GetObject" --> S3_1["S3 Bucket\nService 01"]
    SVC3 -- "GetObject" --> S3_3["S3 Bucket\nService 03"]
    SVC2 -- "ReceiveMessage" --> SQS["SQS Queue\nService 02"]

    SVC1 & SVC2 & SVC3 -- "pull image" --> ECR["ECR\nContainer Registry"]
    SVC1 & SVC2 & SVC3 -- "get credentials" --> SM["Secrets Manager"]

    Private --> NAT --> Internet
```

## Repository Structure

```
envs/dev/
├── vpc.tf               # VPC, subnets, IGW, NAT gateway, route tables, security groups
├── ecs.tf               # ECS cluster, task definitions, services, ALBs per service
├── rds.tf               # Aurora PostgreSQL clusters (one per service)
├── ecr.tf               # ECR repositories
├── s3.tf                # S3 buckets (per service)
├── sqs.tf               # SQS queue
├── ec2.tf               # Bastion/utility EC2 (optional)
├── vars.tf              # Input variables
├── container-defs/      # ECS container definition JSON files
└── policy-docs/         # IAM task policies per service
```

## Prerequisites

- [Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli) installed
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) configured
- S3 bucket for Terraform remote state
- Secrets Manager secret with database credentials

## Usage

```shell
terraform init
terraform plan
terraform apply
```

To tear down:
```shell
terraform destroy
```
