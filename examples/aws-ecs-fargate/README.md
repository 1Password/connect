# AWS ECS with Fargate

The CloudFormation document in this folder will create the following resources as well as the required rules, groups, and policies to make them work.

The result is a publicly accessible hostname that will route requests to a 1Password Connect instance running in AWS Fargate.

If you are looking to deploy into existing an AWS VPC or use an existing ECS cluster please consider this CloudFormation as a starting point and you can modify it using a text editor or using Amazon's CloudFormation Designer tool.

## Networking Resources

- A VPC
- 2 Public subnets
- An Internet Gateway
- An ALB

## ECS Resources

- An ECS Cluster
- Task Definition
  - `1password/connect-api` container
  - `1password/connect-sync` container

## Getting Started

When importing this CloudFormation template you will be prompted to provide a base64url encoded version of the `1password-credentials.json`. This encoded value can be generated using this standard unix shell command

```
cat 1password-credentials.json | base64 | tr '/+' '_-' | tr -d '=' | tr -d '\n'
```
