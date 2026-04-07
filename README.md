# ECS Multiservice Infrastructure

A Terraform project for deploying a multi-service architecture on AWS ECS Fargate with containerized applications, ECR repositories, and message queue integration.

## Overview

This infrastructure deploys:
- **AWS ECS Fargate Cluster**: Container orchestration platform for running microservices
- **S3 Service**: Containerized service for S3 bucket operations (listening on port 5001)
- **SQS Service**: Containerized service for message queue operations
- **ECR Repositories**: Private container registries for both services
- **IAM Roles**: Task execution roles with appropriate permissions
- **Networking**: Security groups and VPC configuration
- **Storage**: S3 bucket and SQS queue for service integration

## Prerequisites

Before deploying this infrastructure, ensure you have:

- **AWS Account**: With appropriate permissions to create ECS, ECR, IAM, S3, and SQS resources
- **Terraform**: Version 1.0 or later
- **AWS CLI**: Configured with credentials
- **Docker**: For building and pushing container images to ECR
- **Git**: Version control (repository already initialized)

## Project Structure

| File | Purpose |
|------|---------|
| `main.tf` | ECS cluster and service definitions |
| `ecr.tf` | ECR repository configurations |
| `iam.tf` | IAM roles and policies for task execution |
| `sg.tf` | Security group definitions |
| `data.tf` | Data sources for AWS account and region information |
| `provider.tf` | AWS provider configuration |
| `backend.tf` | Terraform state backend configuration |

## Quick Start

### 1. Initialize Terraform

```bash
terraform init
```

### 2. Review and Plan Changes

```bash
terraform plan
```

### 3. Apply Configuration

```bash
terraform apply
```

## Configuration

### AWS Region

The default region is set to `ap-southeast-1`. You can change this in the Terraform variables or environment.

### Service Configuration

Services are configured with the following resources:
- **CPU**: 512 units
- **Memory**: 1024 MB
- **Container Port**: 5001 (S3 Service), configurable per service
- **Capacity Provider**: Fargate (on-demand)
- **Minimum Healthy Percent**: 100% (for zero-downtime deployments)

### Environment Variables

Services receive the following environment variables:
- `AWS_REGION`: The AWS region (ap-southeast-1)
- `BUCKET_NAME`: S3 bucket name (jaz-s3-service-bkt)

## IAM Permissions

The following IAM roles are created:
- **s3-service-task-role**: Full S3 access for S3 Service tasks (testing purposes only)
- **sqs-service-task-role**: SQS access for SQS Service tasks

> ⚠️ **Note**: S3 full access is configured for testing. In production, restrict permissions to specific buckets and actions.

## Deploying Container Images

1. Build your Docker images:
   ```bash
   docker build -t jaz-s3-service:latest ./path/to/s3-service
   docker build -t jaz-sqs-service:latest ./path/to/sqs-service
   ```

2. Push to ECR:
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com
   
   docker tag jaz-s3-service:latest <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com/jaz-s3-service-ecr:latest
   docker tag jaz-sqs-service:latest <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com/jaz-sqs-service-ecr:latest
   
   docker push <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com/jaz-s3-service-ecr:latest
   docker push <ACCOUNT_ID>.dkr.ecr.ap-southeast-1.amazonaws.com/jaz-sqs-service-ecr:latest
   ```

3. Update ECS services to use new images (Terraform will handle this with the `latest` tag)

## Cleanup

To destroy all infrastructure and resources:

```bash
terraform destroy
```

## Troubleshooting

### Service Fails to Start
- Check CloudWatch logs: `aws logs tail /ecs/jaz-s3-service --follow`
- Verify Docker image exists in ECR
- Check security group rules allow required traffic

### IAM Permission Errors
- Verify ECS task role has required permissions
- Check task role trust relationships include `ecs-tasks.amazonaws.com`

### Container Image Not Found
- Ensure Docker image is pushed to ECR with correct tag
- Verify ECR repository exists and is in the same region

## Security Considerations

- This infrastructure uses `force_delete = true` for ECR repositories (dangerous in production)
- S3 full access is enabled for demonstration purposes only
- Review and restrict IAM permissions before production deployment
- Enable encryption for S3 and SQS
- Use security group rules to restrict traffic appropriately

## Additional Resources

- [AWS ECS Terraform Module](https://registry.terraform.io/modules/terraform-aws-modules/ecs/aws)
- [AWS Fargate Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/what-is-fargate.html)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)