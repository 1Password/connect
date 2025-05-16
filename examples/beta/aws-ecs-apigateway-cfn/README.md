# AWS ECS with Fargate and API Gateway

The CloudFormation document in this folder will create the following resources, along with the necessary rules, groups, and policies to make them work.

The result is a publicly accessible hostname (via HTTPS) that routes requests to a 1Password Connect instance running in AWS Fargate, using API Gateway for secure ingress. You can use your own VPC and subnets.

To customize further, like deploying into an existing ECS cluster, use this CloudFormation as a starting point and tweak it with a text editor or Amazon's CloudFormation Designer tool.

## Networking Resources

- A VPC Link for API Gateway to route to the Fargate service
- An API Gateway with HTTPS
- A VPC (optional, if you don’t provide your own)
- 2 Public subnets (optional, if you don’t provide your own)
- An Internet Gateway (optional, if creating a VPC)


## ECS Resources

- An ECS Cluster
- Task Definition
  - `1password/connect-api` container
  - `1password/connect-sync` container
- Service Discovery for internal routing

## Getting Started

When [importing this CloudFormation template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-template.html), you’ll be prompted to provide the following:

- **CredentialsJson**: The plain JSON contents of your `1password-credentials.json` file (e.g., `{"token":"your-token"}`). The template will encode this to base64 and store it securely in AWS Secrets Manager.
- **VPCID**, **PublicSubnets** (optional): Provide these if you want to use an existing VPC and subnets. Leave as default to create a new VPC.
- **VPCCIDR** (optional): Sets the CIDR for a new VPC if you’re creating one; ignored if using an existing VPC.

For more details, see the deployment note in the template description.