# AWS Static Website Hosting Infrastructure

This project provides a CloudFormation template for deploying a static website hosting infrastructure on AWS using S3, CloudFront, Route 53, and ACM.

The infrastructure created by this template includes an S3 bucket for hosting static website content, a CloudFront distribution for content delivery, a Route 53 hosted zone for DNS management, and an ACM certificate for HTTPS support. This setup ensures a secure, scalable, and high-performance hosting environment for static websites.

## Repository Structure

```
.
└── combined_cfn
    ├── static-website-params.json
    └── static-website.yaml
```

### Key Files

- `combined_cfn/static-website.yaml`: The main CloudFormation template that defines the entire infrastructure.
- `combined_cfn/static-website-params.json`: A JSON file containing parameters for the CloudFormation stack.

## Usage Instructions

### Prerequisites

- An AWS account with appropriate permissions to create the required resources.
- AWS CLI installed and configured with your AWS credentials.
- A registered domain name (if you plan to use a custom domain).

### Deployment

1. Clone this repository to your local machine.

2. Navigate to the `combined_cfn` directory:

   ```
   cd combined_cfn
   ```

3. Open the `static-website-params.json` file and replace the placeholder values:

   ```json
   [
     {
       "ParameterKey": "DomainName",
       "ParameterValue": "your-domain.com"
     },
     {
       "ParameterKey": "BucketName",
       "ParameterValue": "your-bucket-name"
     },
     {
       "ParameterKey": "TagValue",
       "ParameterValue": "your-tag-value"
     }
   ]
   ```

4. Deploy the CloudFormation stack using the AWS CLI:

   ```
   aws cloudformation create-stack \
     --stack-name your-stack-name \
     --template-body file://static-website.yaml \
     --parameters file://static-website-params.json \
     --capabilities CAPABILITY_IAM
   ```

5. Wait for the stack creation to complete. You can monitor the progress in the AWS CloudFormation console or using the AWS CLI:

   ```
   aws cloudformation wait stack-create-complete --stack-name your-stack-name
   ```

6. Once the stack is created, you can retrieve the outputs using:

   ```
   aws cloudformation describe-stacks --stack-name your-stack-name --query 'Stacks[0].Outputs'
   ```

### Configuration

After deployment, you'll need to configure your domain registrar to use the name servers provided in the stack outputs. This step is crucial for routing traffic to your newly created hosted zone.

### Uploading Content

To upload your static website content:

1. Use the AWS CLI or S3 console to upload your files to the S3 bucket created by the stack.
2. Ensure your main page is named `index.html` and your error page is named `error.html`.

### Testing

After uploading your content and configuring your domain, you can test your website by navigating to your domain in a web browser. Both `http://your-domain.com` and `http://www.your-domain.com` should redirect to their HTTPS counterparts.

### Troubleshooting

- If your website is not accessible, check the CloudFront distribution status in the AWS console.
- Verify that your domain's DNS settings are correctly pointing to the provided name servers.
- Check the S3 bucket contents to ensure your files are uploaded correctly.
- Review the CloudFormation stack events for any deployment errors.

## Data Flow

The static website hosting infrastructure follows this data flow:

1. User requests the website (e.g., https://example.com).
2. DNS request is routed to Route 53.
3. Route 53 directs the request to the CloudFront distribution.
4. CloudFront checks its cache for the requested content.
5. If not in cache, CloudFront requests the content from the S3 bucket.
6. S3 returns the content to CloudFront.
7. CloudFront caches the content and serves it to the user.
8. The user receives the website content over HTTPS.

```
[User] -> [Route 53] -> [CloudFront] -> [S3 Bucket]
   ^            |             |
   |            |             |
   +------------+-------------+
```

## Infrastructure

The CloudFormation template (`static-website.yaml`) defines the following key resources:

### S3 Buckets
- `LoggingBucket`: Stores access logs for the static website bucket.
- `StaticWebsiteBucket`: Hosts the static website content.

### IAM
- `BucketPolicy`: Allows CloudFront to access the static website bucket.

### Route 53
- `PublicHostedZone`: Manages DNS records for the specified domain.

### ACM (AWS Certificate Manager)
- `Certificate`: Provides SSL/TLS certificate for the domain and www subdomain.

### CloudFront
- `CloudFrontOriginAccessControl`: Configures origin access control for S3.
- `CloudFrontDistribution`: Distributes the static website content globally.

### Route 53 Records
- `ApexAliasRecord`: DNS record for the apex domain.
- `WwwAliasRecord`: DNS record for the www subdomain.

These resources work together to create a secure, scalable, and high-performance static website hosting infrastructure.