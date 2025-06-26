#!/bin/bash
# setup-aws-infrastructure.sh

# Set variables
AWS_REGION="us-east-1"
CLUSTER_NAME_FRONTEND="frontend-cluster"
CLUSTER_NAME_BACKEND="backend-cluster"
SERVICE_NAME_FRONTEND="frontend-service"
SERVICE_NAME_BACKEND="backend-service"
ECR_REPO_FRONTEND="frontend-app"
ECR_REPO_BACKEND="backend-api"

echo "Setting up AWS infrastructure..."

# Create ECR repositories
echo "Creating ECR repositories..."
aws ecr create-repository --repository-name $ECR_REPO_FRONTEND --region $AWS_REGION || echo "Frontend ECR repository already exists"
aws ecr create-repository --repository-name $ECR_REPO_BACKEND --region $AWS_REGION || echo "Backend ECR repository already exists"

# Create ECS clusters
echo "Creating ECS clusters..."
aws ecs create-cluster --cluster-name $CLUSTER_NAME_FRONTEND --region $AWS_REGION || echo "Frontend cluster already exists"
aws ecs create-cluster --cluster-name $CLUSTER_NAME_BACKEND --region $AWS_REGION || echo "Backend cluster already exists"

# Create task definitions
echo "Creating task definitions..."

# Frontend task definition
cat > frontend-task-definition.json << EOF
{
  "family": "frontend-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::YOUR_ACCOUNT_ID:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "frontend-container",
      "image": "YOUR_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/frontend-app:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/frontend-task",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
EOF

# Backend task definition
cat > backend-task-definition.json << EOF
{
  "family": "backend-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::YOUR_ACCOUNT_ID:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "backend-container",
      "image": "YOUR_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/backend-api:latest",
      "portMappings": [
        {
          "containerPort": 3001,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/backend-task",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ]
    }
  ]
}
EOF

# Register task definitions
aws ecs register-task-definition --cli-input-json file://frontend-task-definition.json --region $AWS_REGION
aws ecs register-task-definition --cli-input-json file://backend-task-definition.json --region $AWS_REGION

# Create CloudWatch log groups
aws logs create-log-group --log-group-name /ecs/frontend-task --region $AWS_REGION || echo "Frontend log group already exists"
aws logs create-log-group --log-group-name /ecs/backend-task --region $AWS_REGION || echo "Backend log group already exists"

echo "AWS infrastructure setup complete!"
echo "Don't forget to:"
echo "1. Replace YOUR_ACCOUNT_ID with your actual AWS account ID"
echo "2. Set up your VPC, subnets, and security groups"
echo "3. Create ECS services"
echo "4. Configure Application Load Balancer if needed"
echo "5. Set up the required IAM roles and policies"

---

# Required GitHub Secrets Setup Instructions

# In your GitHub repository, go to Settings > Secrets and variables > Actions
# Add these repository secrets:

# AWS_ACCESS_KEY_ID - Your AWS access key ID
# AWS_SECRET_ACCESS_KEY - Your AWS secret access key
# AWS_ACCOUNT_ID - Your AWS account ID (12-digit number)

# Optional secrets for notifications:
# SLACK_WEBHOOK_URL - For Slack notifications
# DISCORD_WEBHOOK_URL - For Discord notifications

---

# Example IAM Policy for GitHub Actions
# Create this policy and attach it to the IAM user whose credentials you're using

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecr:PutImage"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecs:UpdateService",
        "ecs:DescribeServices",
        "ecs:DescribeTasks",
        "ecs:ListTasks"
      ],
      "Resource": "*"
    }
  ]
}
