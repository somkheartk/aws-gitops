# Serverless GitOps with AWS Lambda

## Overview

This example demonstrates GitOps principles applied to serverless applications using AWS Lambda, API Gateway, and infrastructure as code.

**ภาษาไทย:** ตัวอย่างนี้แสดงการใช้หลักการ GitOps กับ serverless applications โดยใช้ AWS Lambda, API Gateway และ Infrastructure as Code

## Architecture

```
┌─────────────────────────────────────────────────┐
│            Git Repository                        │
│     (Lambda Functions + IaC)                     │
└───────────────┬─────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────┐
│         GitHub Actions / CodePipeline            │
│              (CI/CD Pipeline)                    │
└───────────────┬─────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────┐
│         Terraform / SAM / CDK                    │
│        (Infrastructure Deployment)               │
└───────────────┬─────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────┐
│            AWS Services                          │
│  Lambda │ API Gateway │ DynamoDB │ S3           │
└─────────────────────────────────────────────────┘
```

## What You'll Learn

- Deploy Lambda functions using GitOps
- Infrastructure as Code with Terraform
- Serverless CI/CD pipelines
- Environment management for serverless apps

## Prerequisites

- AWS Account
- AWS CLI configured
- Terraform or AWS SAM CLI
- Node.js or Python (for Lambda functions)

## Project Structure

```
serverless-gitops/
├── functions/              # Lambda function code
│   ├── hello-world/
│   │   ├── index.js
│   │   └── package.json
│   └── api-handler/
│       ├── handler.py
│       └── requirements.txt
├── infrastructure/         # Infrastructure as Code
│   ├── main.tf
│   ├── variables.tf
│   ├── api-gateway.tf
│   └── lambda.tf
├── tests/                 # Tests
│   └── integration/
└── .github/
    └── workflows/
        └── deploy.yml
```

## Quick Start

### Option 1: Using Terraform

#### Step 1: Create Lambda Function

```javascript
// functions/hello-world/index.js
exports.handler = async (event) => {
  console.log('Event:', JSON.stringify(event, null, 2));
  
  return {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      message: 'Hello from GitOps Lambda!',
      timestamp: new Date().toISOString(),
      environment: process.env.ENVIRONMENT || 'dev'
    }),
  };
};
```

#### Step 2: Define Infrastructure

```hcl
# infrastructure/lambda.tf
resource "aws_lambda_function" "hello_world" {
  filename         = "functions/hello-world.zip"
  function_name    = "${var.environment}-hello-world"
  role            = aws_iam_role.lambda_role.arn
  handler         = "index.handler"
  runtime         = "nodejs18.x"
  source_code_hash = filebase64sha256("functions/hello-world.zip")

  environment {
    variables = {
      ENVIRONMENT = var.environment
    }
  }

  tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

resource "aws_iam_role" "lambda_role" {
  name = "${var.environment}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_basic" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}
```

#### Step 3: Deploy

```bash
cd infrastructure

# Initialize Terraform
terraform init

# Plan deployment
terraform plan -var="environment=dev"

# Apply
terraform apply -var="environment=dev"
```

### Option 2: Using AWS SAM

#### Step 1: Create SAM Template

```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: GitOps Serverless Application

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - staging
      - production

Globals:
  Function:
    Timeout: 30
    MemorySize: 256
    Runtime: nodejs18.x
    Environment:
      Variables:
        ENVIRONMENT: !Ref Environment

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub '${Environment}-hello-world'
      CodeUri: functions/hello-world/
      Handler: index.handler
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get

  ApiHandlerFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub '${Environment}-api-handler'
      CodeUri: functions/api-handler/
      Handler: handler.lambda_handler
      Runtime: python3.11
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /api/{proxy+}
            Method: any

Outputs:
  HelloWorldApi:
    Description: API Gateway endpoint URL
    Value: !Sub 'https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/'
```

#### Step 2: Deploy with SAM

```bash
# Build
sam build

# Deploy
sam deploy \
  --stack-name gitops-serverless-dev \
  --parameter-overrides Environment=dev \
  --capabilities CAPABILITY_IAM \
  --resolve-s3
```

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy-serverless.yml
name: Deploy Serverless Application

on:
  push:
    branches:
      - main
      - develop
    paths:
      - 'functions/**'
      - 'infrastructure/**'

env:
  AWS_REGION: us-west-2

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: |
          cd functions/hello-world
          npm install
      
      - name: Run tests
        run: |
          cd functions/hello-world
          npm test

  deploy-dev:
    needs: test
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: development
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
      
      - name: Terraform Init
        run: |
          cd infrastructure
          terraform init
      
      - name: Terraform Apply
        run: |
          cd infrastructure
          terraform apply -auto-approve -var="environment=dev"

  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
      
      - name: Terraform Init
        run: |
          cd infrastructure
          terraform init
      
      - name: Terraform Apply
        run: |
          cd infrastructure
          terraform apply -auto-approve -var="environment=production"
```

## Environment Management

### Development
- Auto-deploy on push to `develop` branch
- Lower memory/timeout limits
- Verbose logging enabled

### Staging
- Auto-deploy on push to `staging` branch
- Production-like configuration
- Integration tests run automatically

### Production
- Manual approval required
- Higher memory/timeout limits
- Minimal logging
- Alarms and monitoring

## Testing

### Unit Tests

```javascript
// functions/hello-world/test/handler.test.js
const { handler } = require('../index');

describe('Hello World Handler', () => {
  it('should return 200 status code', async () => {
    const event = {};
    const response = await handler(event);
    
    expect(response.statusCode).toBe(200);
  });

  it('should return message', async () => {
    const event = {};
    const response = await handler(event);
    const body = JSON.parse(response.body);
    
    expect(body.message).toBe('Hello from GitOps Lambda!');
  });
});
```

### Integration Tests

```bash
# Test deployed Lambda function
aws lambda invoke \
  --function-name dev-hello-world \
  --payload '{}' \
  response.json

cat response.json
```

## Monitoring and Logging

### CloudWatch Logs

```bash
# View logs
aws logs tail /aws/lambda/dev-hello-world --follow

# Filter errors
aws logs filter-pattern '{$.level = "ERROR"}' \
  --log-group-name /aws/lambda/dev-hello-world
```

### CloudWatch Alarms

```hcl
resource "aws_cloudwatch_metric_alarm" "lambda_errors" {
  alarm_name          = "${var.environment}-lambda-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "1"
  metric_name         = "Errors"
  namespace           = "AWS/Lambda"
  period              = "300"
  statistic           = "Sum"
  threshold           = "5"
  alarm_description   = "Lambda function error rate"
  
  dimensions = {
    FunctionName = aws_lambda_function.hello_world.function_name
  }
}
```

## Cost Optimization

1. **Right-size memory**
   - Start with 256MB
   - Monitor and adjust based on usage

2. **Use provisioned concurrency wisely**
   - Only for production functions with high traffic
   - Consider cold start vs cost trade-off

3. **Set appropriate timeouts**
   - Avoid unnecessary long timeouts
   - Monitor actual execution time

4. **Clean up old versions**
```bash
# Delete old Lambda versions
aws lambda list-versions-by-function \
  --function-name hello-world \
  --query 'Versions[?Version!=`$LATEST`].Version' \
  --output text | xargs -n1 -I {} \
  aws lambda delete-function --function-name hello-world:{}
```

## Rollback Strategy

### Quick Rollback

```bash
# Publish version
VERSION=$(aws lambda publish-version \
  --function-name hello-world \
  --query 'Version' --output text)

# Create alias pointing to version
aws lambda create-alias \
  --function-name hello-world \
  --name prod \
  --function-version $VERSION

# Rollback by updating alias
aws lambda update-alias \
  --function-name hello-world \
  --name prod \
  --function-version <previous-version>
```

### GitOps Rollback

```bash
# Revert commit
git revert <commit-hash>
git push origin main

# Or rollback to specific commit
git checkout <previous-commit> -- functions/ infrastructure/
git commit -m "Rollback to previous version"
git push origin main
```

## Best Practices

1. **Version Control**
   - Tag releases
   - Use semantic versioning
   - Maintain changelog

2. **Security**
   - Use IAM roles with least privilege
   - Encrypt environment variables
   - Enable AWS WAF for API Gateway

3. **Monitoring**
   - Set up CloudWatch alarms
   - Use X-Ray for tracing
   - Implement structured logging

4. **Testing**
   - Write unit tests
   - Run integration tests
   - Perform load testing

## Cleanup

```bash
# Using Terraform
cd infrastructure
terraform destroy -var="environment=dev"

# Using SAM
sam delete --stack-name gitops-serverless-dev
```

## Resources

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---

**ภาษาไทย:** ตัวอย่างนี้แสดงให้เห็นว่า GitOps ไม่ได้จำกัดแค่ Kubernetes แต่สามารถใช้กับ serverless applications ได้เช่นกัน!
