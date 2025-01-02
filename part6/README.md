# CloudFront Distribution with S3 Origin for Static Website Hosting

This project sets up a CloudFront distribution with an existing S3 bucket as the origin for hosting a static website. It provides a secure and scalable solution for serving web content using AWS services.

The CloudFormation template in this project creates a CloudFront distribution that uses an existing S3 bucket as its origin. It configures the necessary components to serve a static website, including setting up HTTPS support, custom domain names, and proper access controls between CloudFront and S3.

Key features of this setup include:
- CloudFront distribution with custom domain support
- SSL/TLS encryption using an ACM certificate
- Origin Access Control for secure S3 bucket access
- Optimized caching and CORS policies
- Automatic redirection to HTTPS

## Repository Structure

The repository contains two main files:

- `part6-parameters.json`: Contains the parameter values for the CloudFormation stack.
- `part6-static-website.yaml`: The CloudFormation template that defines the infrastructure.

## Usage Instructions

### Prerequisites

- An AWS account with appropriate permissions
- AWS CLI installed and configured
- An existing S3 bucket containing your static website files
- A registered domain name
- An SSL/TLS certificate for your domain (must be in the us-east-1 region)

### Deployment

1. Clone this repository to your local machine.

2. Update the `part6-parameters.json` file with your specific values:

```json
[
    { 
        "ParameterKey": "ExistingBucketName", 
        "ParameterValue": "your-bucket-name" 
    },
    { 
        "ParameterKey": "DomainName", 
        "ParameterValue": "example.com" 
    },
    { 
        "ParameterKey": "AcmCertificateArn", 
        "ParameterValue": "ACM_CERTIFICATE_ARN" 
    },
    { 
        "ParameterKey": "TagValue", 
        "ParameterValue": "example.com" 
    }
]
```

3. Deploy the CloudFormation stack using the AWS CLI:

```bash
aws cloudformation create-stack \
  --stack-name my-static-website \
  --template-body file://part6-static-website.yaml \
  --parameters file://part6-parameters.json \
  --capabilities CAPABILITY_IAM
```

4. Wait for the stack creation to complete. You can monitor the progress in the AWS CloudFormation console or using the AWS CLI:

```bash
aws cloudformation wait stack-create-complete --stack-name my-static-website
```

5. Once the stack is created, retrieve the CloudFront distribution domain name:

```bash
aws cloudformation describe-stacks \
  --stack-name my-static-website \
  --query "Stacks[0].Outputs[?OutputKey=='DistributionDomainName'].OutputValue" \
  --output text
```

6. Update your DNS settings to point your custom domain to the CloudFront distribution domain name using a CNAME record.

### Configuration

The CloudFront distribution is configured with the following settings:

- Price Class: PriceClass_100 (US, Canada, Europe excluding Middle East)
- Minimum TLS version: TLSv1.2_2021
- Viewer Protocol Policy: redirect-to-https
- Default Root Object: index.html

You can modify these settings in the `part6-static-website.yaml` file if needed.

### Troubleshooting

1. Issue: CloudFront can't access the S3 bucket

   - Check the S3 bucket policy to ensure it allows access from the CloudFront distribution.
   - Verify that the Origin Access Control is correctly set up.

2. Issue: SSL/TLS certificate errors

   - Ensure the ACM certificate is in the us-east-1 region.
   - Verify that the certificate covers both the apex domain and the www subdomain.

3. Issue: Custom domain not working

   - Check your DNS settings to ensure the CNAME record points to the correct CloudFront distribution domain.
   - Allow sufficient time for DNS propagation (up to 48 hours in some cases).

For more detailed troubleshooting, enable CloudFront logging and check the CloudWatch logs.

## Data Flow

The data flow for serving content through this setup is as follows:

1. User requests content from your custom domain (e.g., example.com).
2. DNS routes the request to the CloudFront distribution.
3. CloudFront checks its cache for the requested content.
4. If not in cache, CloudFront requests the content from the S3 origin.
5. S3 returns the content to CloudFront.
6. CloudFront caches the content and returns it to the user.
7. Subsequent requests for the same content are served from CloudFront's cache.

```
[User] -> [DNS] -> [CloudFront] -> [S3 Bucket]
           ^          |   ^          |
           |          |   |          |
           |          v   |          v
        [Custom    [CloudFront]    [Origin
         Domain]     Cache]        Access
                                  Control]
```

Note: The Origin Access Control ensures that the S3 bucket content can only be accessed through CloudFront, enhancing security.

## Infrastructure

The infrastructure is defined in the `part6-static-website.yaml` CloudFormation template. Key resources include:

- CloudFront::OriginAccessControl
  - Purpose: Manages secure access between CloudFront and the S3 bucket

- CloudFront::Distribution
  - Purpose: Serves the static website content with global edge locations
  - Configuration:
    - Uses the specified S3 bucket as the origin
    - Applies custom domain names and SSL/TLS certificate
    - Configures caching and CORS policies

- S3::BucketPolicy
  - Purpose: Grants the CloudFront distribution read access to the S3 bucket contents

These resources work together to create a secure and efficient static website hosting solution using CloudFront and S3.