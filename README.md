# CI/CD Pipeline for Deploying Application on AWS ECS using GitHub Actions

## Overview
This project leverages a **CI/CD pipeline** with **GitHub Actions** to deploy a Dockerized application on **AWS ECS (Elastic Container Service)**. The pipeline automates building, testing, and deploying the application to AWS.

## Features
- **Automated Builds**: GitHub Actions triggers a build process on every push.
- **Containerization**: Uses Docker to package the application.
- **Deployment to AWS ECS**: Deploys the Docker image to an AWS ECS service using Fargate or EC2.
- **AWS ECR Integration**: Pushes the built Docker image to Amazon Elastic Container Registry (ECR).
- **Infrastructure as Code**: Uses Terraform for infrastructure management.
- **AWS Secret Manager**: Securely manages and retrieves secrets for the application.

## Prerequisites
Ensure you have the following:
- An **AWS account** with permissions to use ECS, ECR, and IAM.
- **GitHub repository** for hosting your application and workflow files.
- **Docker** installed on your local machine.
- **AWS CLI** configured with your credentials.
- **GitHub Secrets** set up for AWS and RDS authentication (e.g., `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `PERSONAL_ACCESS_TOKEN`, `RDS_DB_NAME`, `RDS_DB_PASSWORD`,`RDS_DB_USERNAME`, `ECR_REGISTRY`).

## GitHub Actions Workflow
The CI/CD pipeline is defined in `.github/workflows/deploy_pipeline.yml`. It performs the following steps:
1. **Configure AWS Credentials**: Configures authentication with AWS.
2. **Build AWS infrastructure**: Deploy aws infrastructure.
3. **Create AWS ECR**: Create AWS ECR repository to store docker images.
4. **Start Self-hosted EC2 Runner**: Start self-hosted EC2 runner to build and push docker images and migrate data to RDS database.
5. **Build and Tag Docker and Push Image**: Builds the container image and tags it properly.
6. **Create Environment File and Export to S3**: Create environment file and export to S3 for ECS to use.
7. **Migrate Data into RDS Database with Flyway**: Migrate data into RDS database with Flyway.
8. **Stop the Self-hosted EC2 Runner**: Stop self-hosted EC2 runner.
9. **Create TD Revision**: Create new task definition revision for ECS service to use.
10. **Restart ECS Service**: Restart ECS Fargate service.

## Important Notes
1. **Handling Special Characters in YAML**: If your workflow YAML file contains special characters, it's recommended to enclose values in double quotes (`" "`) to avoid syntax errors.
2. **Docker Image Tags are Case-Sensitive**: Ensure consistency in naming. For example, `LATEST` is different from `latest`. Always specify the correct tag when pulling or deploying an image.

## Example GitHub Actions Workflow
Below is an example workflow file (`.github/workflows/deploy_pipeline.yml`):
```yaml
name: Deploy Pipeline

on:
  push:
    branches: [main]
    paths-ignore: [README.md]

env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  AWS_REGION: us-east-1
  TERRAFORM_ACTION: apply
  GITHUB_USERNAME: li-zhang1
  REPOSITORY_NAME: application-codes
  WEB_FILE_ZIP: rentzone.zip
  WEB_FILE_UNZIP: rentzone
  FLYWAY_VERSION: 11.3.0



jobs:
  # Configure AWS credentials 
  configure_aws_credentials:
    name: Configure AWS credentials
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1-node16
        with:
          aws-access-key-id: ${{ env.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ env.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

```

## Deployment Verification
After deployment, verify the application is running:
```sh
aws ecs list-tasks --cluster rentzone-dev-cluster
```
Check the logs:
```sh
aws logs describe-log-groups
aws logs tail /ecs/rentzone-dev-service
```

## Apply or Destroy
In order to run terraform apply, set TERRAFORM_ACTION to apply
```yaml
TERRAFORM_ACTION: apply
```
In order to run terraform destroy, set TERRAFORM_ACTION to destroy
```yaml
TERRAFORM_ACTION: destroy
```


## Conclusion
This CI/CD pipeline streamlines the deployment of a Dockerized application to AWS ECS, ensuring an efficient and automated deployment process. Modify the workflow as needed to fit your infrastructure and scaling requirements.

