# AWS ACM Certificate Creation for Static Website

This project provides an AWS CloudFormation template for creating an AWS Certificate Manager (ACM) certificate with DNS validation for a specified domain name and its www subdomain. The certificate is designed to secure a static website hosted on AWS.

The CloudFormation template automates the process of requesting and validating an SSL/TLS certificate for your domain, which is a crucial step in setting up a secure static website. By using this template, you can ensure that your website is protected with HTTPS, enhancing security and user trust.

The template is configurable through parameters, allowing you to specify the domain name, hosted zone ID, and tag value. This flexibility makes it easy to use the same template for different domains or environments.

## Repository Structure

- `part5-parameters.json`: Contains the parameter values for the CloudFormation stack.
- `part5-static-website.yaml`: The main CloudFormation template that defines the ACM certificate resource.

## Usage Instructions

### Prerequisites

- An AWS account with appropriate permissions to create ACM certificates and access Route 53.
- A registered domain name.
- A Route 53 hosted zone for your domain.

### Installation

1. Clone this repository or download the `part5-static-website.yaml` and `part5-parameters.json` files.

2. Update the `part5-parameters.json` file with your specific values:

```json
[
    { 
        "ParameterKey": "DomainName", 
        "ParameterValue": "your-domain.com" 
    },
    {
        "ParameterKey": "HostedZoneId",
        "ParameterValue": "YOUR_HOSTED_ZONE_ID"
    },
    {
        "ParameterKey": "TagValue",
        "ParameterValue": "your-domain.com"
    }
]
```

3. Deploy the CloudFormation stack using the AWS CLI:

```bash
aws cloudformation create-stack \
  --stack-name your-stack-name \
  --template-body file://part5-static-website.yaml \
  --parameters file://part5-parameters.json
```

Replace `your-stack-name` with a name for your CloudFormation stack.

### Configuration

The template accepts the following parameters:

- `DomainName`: The domain name for the certificate (e.g., example.com)
- `HostedZoneId`: The Route 53 Hosted Zone ID
- `TagValue`: The tag value for the certificate

### Troubleshooting

1. Certificate Creation Failure

Problem: The certificate fails to create.

Solution:
- Verify that the domain name in the parameters matches your registered domain.
- Ensure the Hosted Zone ID is correct and corresponds to your domain.
- Check that you have the necessary permissions in your AWS account.

Debug steps:
a. Enable CloudFormation stack creation logs:
```bash
aws cloudformation update-stack --stack-name your-stack-name --template-body file://part5-static-website.yaml --parameters file://part5-parameters.json --enable-termination-protection
```
b. Review the stack events in the AWS CloudFormation console or use the AWS CLI:
```bash
aws cloudformation describe-stack-events --stack-name your-stack-name
```

2. DNS Validation Issues

Problem: The certificate is created but remains in the "Pending Validation" state.

Solution:
- Ensure that your domain's DNS is managed by Route 53 and the Hosted Zone ID is correct.
- Wait for up to 30 minutes for DNS propagation.

Debug steps:
a. Check the status of the certificate:
```bash
aws acm describe-certificate --certificate-arn <certificate-arn>
```
b. Verify DNS records in Route 53:
```bash
aws route53 list-resource-record-sets --hosted-zone-id <your-hosted-zone-id>
```

## Data Flow

The data flow for creating an ACM certificate using this CloudFormation template is as follows:

1. User provides parameter values (DomainName, HostedZoneId, TagValue).
2. CloudFormation stack is created using the template and parameters.
3. ACM receives the certificate request for the domain and www subdomain.
4. ACM generates DNS validation records.
5. CloudFormation adds the DNS validation records to the specified Route 53 hosted zone.
6. ACM verifies the DNS records and issues the certificate.
7. The certificate ARN is output by the CloudFormation stack.

```
[User Input] -> [CloudFormation] -> [ACM] -> [Route 53]
                      ^                |
                      |                v
                      +--- [Certificate Validation] ---+
                                      |
                                      v
                              [Certificate Issued]
```

Note: The DNS validation process may take up to 30 minutes to complete.

## Infrastructure

The CloudFormation template defines the following AWS resource:

- AWS::CertificateManager::Certificate
  - Type: ACM Certificate
  - Purpose: Provides SSL/TLS certificate for the specified domain and www subdomain
  - Configuration:
    - DomainName: Primary domain name
    - SubjectAlternativeNames: www subdomain
    - ValidationMethod: DNS
    - DomainValidationOptions: Uses provided HostedZoneId for both domain and www subdomain
    - Tags: Includes a 'workload' tag with the specified TagValue

The template also defines an output:

- CertificateArn: The ARN of the created ACM certificate

This infrastructure setup ensures that a properly configured SSL/TLS certificate is created and validated for use with your static website.