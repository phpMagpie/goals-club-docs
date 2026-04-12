# Goals Club - SSL Certificate Setup

## Overview

The SSL certificate for `thegoalsclub.co.uk` and `*.thegoalsclub.co.uk` is managed **outside of Pulumi** as a shared resource used by all Goals Club applications.

## Certificate Details

| Property | Value |
|----------|-------|
| **Certificate ARN** | `arn:aws:acm:us-east-1:724772097333:certificate/f15dab15-8c7c-460f-beae-b30fa3845971` |
| **Region** | us-east-1 (required for CloudFront) |
| **Domains** | `thegoalsclub.co.uk`, `*.thegoalsclub.co.uk` |
| **Validation** | DNS (via Cloudflare) |

## DNS Validation Record (Cloudflare)

Add this CNAME record to validate the certificate:

| Type | Name | Target |
|------|------|--------|
| CNAME | `_af1e1f43da5b3cdb5c1e3eb760b8ec5c` | `_b41682900327a92afdbc8972257b9649.jkddzztszm.acm-validations.aws` |

## Check Certificate Status

```bash
AWS_PROFILE=goalsclub aws acm describe-certificate \
  --certificate-arn 'arn:aws:acm:us-east-1:724772097333:certificate/f15dab15-8c7c-460f-beae-b30fa3845971' \
  --region us-east-1 \
  --query 'Certificate.Status'
```

## Usage in Pulumi

The certificate ARN is stored in the `.secrets` file:
```
ACM_CERTIFICATE_ARN_US_EAST_1=arn:aws:acm:us-east-1:724772097333:certificate/f15dab15-8c7c-460f-beae-b30fa3845971
```

And referenced in Pulumi stacks to configure CloudFront custom domains.

## Custom Domain CNAME Records (Cloudflare)

Once the certificate is validated, add these CNAMEs to point to CloudFront:

| Name | Target | Purpose |
|------|--------|---------|
| `dev-admin` | `d2msc9majzh3y7.cloudfront.net` | Admin Console |
| `dev-app` | `dei3pnat6dbzt.cloudfront.net` | Public Web App |

For production (when ready):
| Name | Target | Purpose |
|------|--------|---------|
| `admin` | (CloudFront domain) | Admin Console |
| `app` or `www` | (CloudFront domain) | Public Web App |

