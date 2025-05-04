# AWS ECS with Fargate

The CloudFormation document in this folder will create the following resources, along with the necessary rules, groups, and policies to make them work.

The result is a publicly accessible hostname (via HTTPS) that routes requests to a 1Password Connect instance running in AWS Fargate. You can use your own VPC and subnets, or optionally set up DNS records automatically with Route 53. 

To customize further, like deploying into an existing ECS cluster, use this CloudFormation as a starting point and tweak it with a text editor or Amazon's CloudFormation Designer tool.

## Networking Resources

- A VPC (optional, if you don’t provide your own)
- 2 Public subnets (optional, if you don’t provide your own)
- An Internet Gateway (optional, if creating a VPC)
- An ALB (Application Load Balancer) with HTTPS
- An ACM certificate for HTTPS
- Route 53 DNS records (optional, if you provide a hosted zone ID)

## ECS Resources

- An ECS Cluster
- Task Definition
  - `1password/connect-api` container
  - `1password/connect-sync` container
- Service Discovery for internal routing

## Getting Started

When [importing this CloudFormation template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-template.html), you’ll be prompted to provide the following:

- **CredentialsJson**: The plain JSON contents of your `1password-credentials.json` file.  The template will encode this to base64 and store it securely in AWS Secrets Manager.
- **DomainName**: A fully qualified domain name for the ALB (e.g., `connect.example.com`). This is required for HTTPS.
- **Route53HostedZoneID** (optional): If you’re using Route 53, provide the hosted zone ID to automatically create DNS records. Leave blank to set up DNS manually with your registrar.
- **VPCID**, **PublicSubnets** (optional): Provide these if you want to use an existing VPC and subnets. Leave as default to create a new VPC. 
- **VPCCIDR** (optional): Sets the CIDR for a new VPC if you’re creating one; ignored if using an existing VPC.

### DNS Setup (If Not Using Route 53)
If you leave `Route53HostedZoneID` blank, the deployment will pause at the `HTTPSCertificate` creation step, waiting for DNS validation:
1. Check the ACM console for the certificate (status: `Pending validation`).
2. Add the provided CNAME record to your DNS registrar to validate the certificate.
3. Wait for the certificate to be issued (status: `Issued`), then the deployment will continue.
4. After deployment, use the `ExternalUrl` output (or `CNAME Name` and `CNAME Value`) to create an `A` (with alias) or `CNAME` record in your registrar to point to the ALB.

For more details, see the deployment note in the template description.