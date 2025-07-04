# Reusable AWS Lambda Terraform Module

## Why make a reusable Lambda module?
- DRY (Don’t Repeat Yourself): Avoid duplicating similar Terraform code for each Lambda.
- Consistency: Enforce consistent naming, configuration, IAM roles, environment variables, and deployment patterns.
- Maintainability: Fix bugs or add features once in the module, and it applies to all Lambdas using it.
- Parameterization: Customize each Lambda’s code path, memory, timeout, environment variables, and triggers via module inputs.
- Scalability: Easily add new Lambdas by reusing the module.

## What to include in a Lambda module?
- aws_lambda_function resource configuration
- IAM role and policies for the Lambda
- Optional CloudWatch log group
- Lambda environment variables
- Packaging and deployment instructions (e.g., S3 bucket for code)
- Optional event sources (API Gateway, S3 events, DynamoDB streams, etc.)


## Example module interface (variables)

```
variable "function_name" {}
variable "handler" {}
variable "runtime" {}
variable "role_arn" {}
variable "source_path" {}         # Path to zipped lambda code or S3 bucket/key
variable "environment_variables" {
  type = map(string)
  default = {}
}
variable "memory_size" {
  default = 128
}
variable "timeout" {
  default = 3
}
```

## Example usage

```
module "my_lambda" {
  source = "../modules/lambda"

  function_name         = "user-registration"
  handler               = "index.handler"
  runtime               = "nodejs18.x"
  role_arn              = aws_iam_role.lambda_exec.arn
  source_path           = "s3://my-lambda-bucket/user-registration.zip"
  environment_variables = {
    ENV = "prod"
  }
  memory_size           = 256
  timeout               = 10
}
```

## Bonus tips
- If Lambdas share the same IAM role and policies, you can create those separately and pass in the role ARN to the module.
- For packaging, consider automating build and zip in your CI/CD pipeline.
- You can also create a module to manage Lambda event sources (API Gateway, S3 triggers) to keep your Lambda module focused.