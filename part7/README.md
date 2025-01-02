# AWS CloudFront DNS Configuration for Static Website

This project provides CloudFormation templates to configure DNS records for a static website hosted on AWS CloudFront with an S3 bucket origin.

The templates create Route 53 DNS records that point your domain and www subdomain to a CloudFront distribution, enabling your static website to be served via CloudFront's global content delivery network.

## Repository Structure

- `part7-parameters.json`: Contains configuration parameters for the CloudFormation stack.
- `part7-static-website.yaml`: CloudFormation template that creates the necessary DNS records.

## Usage Instructions

### Prerequisites

- An AWS account with appropriate permissions
- AWS CLI installed and configured
- An existing S3 bucket with your static website content
- A CloudFront distribution set up to serve your S3 bucket
- A Route 53 hosted zone for your domain

### Deployment Steps

1. Clone this repository to your local machine.

2. Navigate to the project directory:

   ```bash
   cd path/to/project/part7
   ```

3. Open `part7-parameters.json` and replace the placeholder values:

   ```json
   [
     { 
       "ParameterKey": "HostedZoneId", 
       "ParameterValue": "YOUR_HOSTED_ZONE_ID" 
     },
     { 
       "ParameterKey": "DomainName", 
       "ParameterValue": "YOUR_DOMAIN_NAME" 
     },
     { 
       "ParameterKey": "CloudFrontDomainName", 
       "ParameterValue": "YOUR_CLOUDFRONT_DOMAIN_NAME" 
     }
   ]
   ```

4. Deploy the CloudFormation stack:

   ```bash
   aws cloudformation create-stack \
     --stack-name static-website-dns \
     --template-body file://part7-static-website.yaml \
     --parameters file://part7-parameters.json
   ```

5. Monitor the stack creation:

   ```bash
   aws cloudformation describe-stacks \
     --stack-name static-website-dns \
     --query 'Stacks[0].StackStatus'
   ```

   Wait until the status is `CREATE_COMPLETE`.

### Verification

After successful deployment, verify that the DNS records have been created:

1. Check the A records in your Route 53 hosted zone:

   ```bash
   aws route53 list-resource-record-sets \
     --hosted-zone-id YOUR_HOSTED_ZONE_ID \
     --query "ResourceRecordSets[?Type == 'A']"
   ```

2. Ensure that both your apex domain and www subdomain resolve to the CloudFront distribution:

   ```bash
   dig +short YOUR_DOMAIN_NAME
   dig +short www.YOUR_DOMAIN_NAME
   ```

   Both should return the CloudFront distribution's IP addresses.

### Troubleshooting

- If the stack creation fails, check the CloudFormation events:

  ```bash
  aws cloudformation describe-stack-events \
    --stack-name static-website-dns
  ```

- Ensure that the HostedZoneId in your parameters file matches your Route 53 hosted zone.
- Verify that the CloudFrontDomainName parameter is correct and that the distribution is deployed.
- If DNS resolution is not working, allow up to 48 hours for DNS propagation.

## Data Flow

1. User requests your website (e.g., example.com or www.example.com).
2. DNS request is routed to Route 53.
3. Route 53 returns the CloudFront distribution's domain name.
4. CloudFront serves the content from the nearest edge location.
5. If the content is not in the edge location, CloudFront retrieves it from the S3 bucket origin.

```
[User] -> [Route 53] -> [CloudFront] -> [S3 Bucket]
```

## Infrastructure

The CloudFormation template `part7-static-website.yaml` defines the following resources:

- AWS::Route53::RecordSet (ApexAliasRecord):
  - Creates an A record for the apex domain
  - Points to the CloudFront distribution

- AWS::Route53::RecordSet (WwwAliasRecord):
  - Creates an A record for the www subdomain
  - Points to the same CloudFront distribution

Both records use alias targets, which allow them to point directly to AWS resources like CloudFront distributions.