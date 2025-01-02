# AWS Route 53 Public Hosted Zone Creator

This project provides a CloudFormation template for creating a public hosted zone in AWS Route 53 for a specified domain name.

The template allows you to easily set up DNS management for your domain using AWS Route 53. It creates a public hosted zone, which is a container for DNS records that define how to route traffic for a specific domain and its subdomains.

By using this template, you can automate the process of creating a hosted zone and obtain the necessary information to configure your domain registrar. This is particularly useful for setting up DNS management for websites, applications, or services that you want to make accessible via a custom domain name.

## Repository Structure

- `part4-parameters.json`: Contains the parameter configuration for the CloudFormation stack.
- `part4-static-website.yaml`: The main CloudFormation template that defines the Route 53 public hosted zone.

## Usage Instructions

### Prerequisites

- An AWS account with permissions to create Route 53 resources
- AWS CLI installed and configured with appropriate credentials
- A registered domain name

### Deployment

1. Clone this repository to your local machine.

2. Navigate to the `part4` directory:

   ```bash
   cd part4
   ```

3. Review and modify the `part4-parameters.json` file if needed. By default, it sets the domain name to "foobar.com":

   ```json
   [
     {
       "ParameterKey": "DomainName",
       "ParameterValue": "foobar.com"
     }
   ]
   ```

   Replace "foobar.com" with your actual domain name.

4. Deploy the CloudFormation stack using the AWS CLI:

   ```bash
   aws cloudformation create-stack \
     --stack-name my-route53-hosted-zone \
     --template-body file://part4-static-website.yaml \
     --parameters file://part4-parameters.json
   ```

5. Wait for the stack creation to complete. You can check the status using:

   ```bash
   aws cloudformation describe-stacks --stack-name my-route53-hosted-zone
   ```

6. Once the stack is created, retrieve the outputs:

   ```bash
   aws cloudformation describe-stacks --stack-name my-route53-hosted-zone --query 'Stacks[0].Outputs'
   ```

   This will provide you with the `HostedZoneId` and `NameServers` for your new hosted zone.

7. Update your domain registrar with the name servers provided in the output. This step varies depending on your registrar, but typically involves setting custom name servers for your domain.

### Configuration

The CloudFormation template (`part4-static-website.yaml`) accepts one parameter:

- `DomainName`: The domain name for which you want to create a hosted zone (e.g., example.com)

You can modify this parameter in the `part4-parameters.json` file or provide it directly when creating the stack.

### Troubleshooting

1. If the stack creation fails, check the CloudFormation events for error messages:

   ```bash
   aws cloudformation describe-stack-events --stack-name my-route53-hosted-zone
   ```

2. Common issues and solutions:
   - **Error: The domain name is already in use**: Ensure that you don't already have a hosted zone for the same domain in your AWS account.
   - **Error: Invalid domain name**: Verify that the domain name in `part4-parameters.json` is correctly formatted and is a valid domain name.

3. If you encounter permission-related errors, ensure that your AWS CLI is configured with credentials that have sufficient permissions to create Route 53 resources.

## Data Flow

The data flow for this project is straightforward:

1. User provides the domain name through the parameter file or directly to the CloudFormation stack.
2. CloudFormation template reads the domain name parameter.
3. CloudFormation creates a Route 53 public hosted zone using the provided domain name.
4. Route 53 generates name servers for the hosted zone.
5. CloudFormation outputs the Hosted Zone ID and the list of name servers.

```
[User Input] -> [CloudFormation] -> [Route 53] -> [Hosted Zone Created]
                      |                                  |
                      |                                  |
                      v                                  v
               [Output Hosted Zone ID]        [Output Name Servers]
```

Note: After the hosted zone is created, you need to update your domain registrar with the provided name servers to delegate DNS management to Route 53.

## Infrastructure

The infrastructure for this project is defined in the CloudFormation template `part4-static-website.yaml`. It creates the following AWS resource:

- Route 53 Hosted Zone (AWS::Route53::HostedZone)
  - Type: AWS::Route53::HostedZone
  - Purpose: Creates a public hosted zone for the specified domain name

The template also defines two outputs:
- HostedZoneId: The ID of the created hosted zone
- NameServers: A comma-separated list of name servers for the hosted zone

These outputs are crucial for configuring your domain registrar and for future DNS management tasks.