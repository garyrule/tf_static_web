# tf_static_web
A Terraform Module to configure a static website using Amazon AWS Services

Support for using Gandi LiveDNS instead of Route53

<!-- TOC -->
* [tf_static_web](#tfstaticweb)
  * [Prerequisites](#prerequisites)
  * [Cost Example](#cost-example)
      * [Assumptions](#assumptions)
    * [AWS](#aws)
    * [Gandi](#gandi)
  * [Notes](#notes)
  * [Examples](#examples)
  * [Requirements](#requirements)
  * [Providers](#providers)
  * [Resources](#resources)
  * [Inputs](#inputs)
  * [Outputs](#outputs)
<!-- TOC -->
| Function            | Service                  |
|---------------------|--------------------------|
| Static File hosting | S3                       |
| Certificate         | ACM                      |
| DNS                 | Route53 or Gandi LiveDNS |
| CDN                 | Cloudfront               |


## Prerequisites
* AWS Account
* Route53 Zone - *If using Route53*
* Gandi Account - *If using Gandi LiveDNS*


![tf_static_web](./img/tf_static_web.jpeg)


## Cost Example
You should use the [AWS Pricing Calculator](https://calculator.aws/) to best determine your costs.
The estimates are an example only and do not take into account any other infrastructure you may run.

The cost is lower if your deployment is covered under the [AWS Free Tier](https://aws.amazon.com/free/)

#### Assumptions
* 1 S3 bucket with 500 MB of data
  * 250 PUT, COPY, POST, LIST requests / mo
  * 10,000 SELECT, GET requests / mo
  * Outbound data to Cloudfront is free
* 1 x Cloudfront Distribution
  * 10 GB /mo of outbound data transfer to the internet
  * 25,000 HTTPS requests /mo
* 1 x either AWS Route 53 Hosted Zone or a Gandi Domain w/ LiveDNS
  * w/ Route 53, 25,000 standard queries / mo
  * w/ Gandi, unlimited /mo


### AWS
| Service      | Cost / Mo | Covered Under Free Tier / mo |
|--------------|-----------|------------------------------|
| Route 53     | $0.51     | $0.51                        |
| S3           | $0.02     | $0.02                        |
| CloudFront   | $0.88     | $0.03                        |
| **Total**    | **$1.41** | **$0.56**                    |


### Gandi

| Service       | Cost / Mo | Covered Under Free Tier / mo |
|---------------|-----------|------------------------------|
| Gandi LiveDNS | $0.00     | $0.00                        |
| S3            | $0.02     | $0.02                        |
| CloudFront    | $0.88     | $0.03                        |
| **Total**     | **$0.90** | **$0.05**                    |


## Notes
When you use the Amazon S3 static website endpoint, connections between CloudFront and Amazon S3 are available only over HTTP.

For this configuration, the S3 bucket's block public access settings must be turned off.

The end result is that the S3 bucket is world readable over HTTP even though Cloudfront delivers the content over SSL.

To provide a small barrier to the public from reading the contents fo the S3 bucket we will pass a shared secret in a header from Cloudfront to S3. The S3 policy will require the header be present and have the correct value.

This module will generate a header value automatically or you can provide your own with the variable `referer_header`

## Examples
* [AWS Only Example](./examples/aws-only)
* [Gandi LiveDNS Example](./examples/gandi-livedns)

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.4.6, < 2.0.0 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | ~> 5.5.0 |
| <a name="requirement_gandi"></a> [gandi](#requirement\_gandi) | = 2.2.3 |
| <a name="requirement_random"></a> [random](#requirement\_random) | 3.5.1 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | 5.5.0 |
| <a name="provider_aws.use1"></a> [aws.use1](#provider\_aws.use1) | 5.5.0 |
| <a name="provider_gandi"></a> [gandi](#provider\_gandi) | 2.2.3 |
| <a name="provider_random"></a> [random](#provider\_random) | 3.5.1 |

## Resources

| Name | Type |
|------|------|
| [aws_acm_certificate.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate) | resource |
| [aws_acm_certificate_validation.site-aws](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate_validation) | resource |
| [aws_acm_certificate_validation.site-gandi](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate_validation) | resource |
| [aws_cloudfront_distribution.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudfront_distribution) | resource |
| [aws_route53_record.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record) | resource |
| [aws_route53_record.site-validation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record) | resource |
| [aws_s3_bucket.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket) | resource |
| [aws_s3_bucket_acl.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_acl) | resource |
| [aws_s3_bucket_cors_configuration.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_cors_configuration) | resource |
| [aws_s3_bucket_ownership_controls.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_ownership_controls) | resource |
| [aws_s3_bucket_policy.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_policy) | resource |
| [aws_s3_bucket_public_access_block.site-grant](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_public_access_block) | resource |
| [aws_s3_bucket_versioning.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_versioning) | resource |
| [aws_s3_bucket_website_configuration.site](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket_website_configuration) | resource |
| [gandi_livedns_record.site](https://registry.terraform.io/providers/go-gandi/gandi/2.2.3/docs/resources/livedns_record) | resource |
| [gandi_livedns_record.site-validation](https://registry.terraform.io/providers/go-gandi/gandi/2.2.3/docs/resources/livedns_record) | resource |
| [random_string.referer](https://registry.terraform.io/providers/hashicorp/random/3.5.1/docs/resources/string) | resource |
| [aws_iam_policy_document.get-all-with-header](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_website_hostname"></a> [website\_hostname](#input\_website\_hostname) | Fully qualified domain name for website - www.yourdomain.com | `string` | n/a | yes |
| <a name="input_bucket_versioning"></a> [bucket\_versioning](#input\_bucket\_versioning) | S3 Bucket Versioning? | `bool` | `false` | no |
| <a name="input_cloudfront_price_class"></a> [cloudfront\_price\_class](#input\_cloudfront\_price\_class) | CloudFront edge locations are grouped into geographic regions, and we’ve grouped regions into price classes | `string` | `"PriceClass_100"` | no |
| <a name="input_cloudfront_viewer_security_policy"></a> [cloudfront\_viewer\_security\_policy](#input\_cloudfront\_viewer\_security\_policy) | Which Security Policy to use. Consult this table: https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/secure-connections-supported-viewer-protocols-ciphers.html | `string` | `"TLSv1.2_2021"` | no |
| <a name="input_dns_type"></a> [dns\_type](#input\_dns\_type) | Either "aws" or "gandi" depending on your DNS set up | `string` | `"aws"` | no |
| <a name="input_force_destroy_bucket"></a> [force\_destroy\_bucket](#input\_force\_destroy\_bucket) | Force Destroy S3 Bucket? | `bool` | `false` | no |
| <a name="input_gandi_key"></a> [gandi\_key](#input\_gandi\_key) | Gandi API Key | `string` | `""` | no |
| <a name="input_gandi_sharing_id"></a> [gandi\_sharing\_id](#input\_gandi\_sharing\_id) | Gandi API Sharing ID | `string` | `""` | no |
| <a name="input_referer_header"></a> [referer\_header](#input\_referer\_header) | Shared secret that Cloudfront will pass to S3 in order to read the bucket contents | `string` | `""` | no |
| <a name="input_region"></a> [region](#input\_region) | AWS Region | `string` | `"us-west-2"` | no |
| <a name="input_route53_zone_id"></a> [route53\_zone\_id](#input\_route53\_zone\_id) | Route 53 Zone ID for website TLD | `string` | `""` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_bucket-endpoint"></a> [bucket-endpoint](#output\_bucket-endpoint) | Static file S3 Bucket Website Endpoint |
| <a name="output_bucket-hosted-zone-id"></a> [bucket-hosted-zone-id](#output\_bucket-hosted-zone-id) | Static file Bucket Zone ID |
| <a name="output_bucket-id"></a> [bucket-id](#output\_bucket-id) | Static file S3 Bucket ID |
| <a name="output_bucket-region"></a> [bucket-region](#output\_bucket-region) | Static file S3 Bucket Region |
| <a name="output_bucket-versioning"></a> [bucket-versioning](#output\_bucket-versioning) | Static file Bucket Versioning |
| <a name="output_certificate-validation-domain-name"></a> [certificate-validation-domain-name](#output\_certificate-validation-domain-name) | Certificate Validation, validation record FQDNs |
| <a name="output_cloudfront-distribution-domain-name"></a> [cloudfront-distribution-domain-name](#output\_cloudfront-distribution-domain-name) | Cloudfront Distribution Domain Name |
| <a name="output_cloudfront-distribution-http-last-modified-time"></a> [cloudfront-distribution-http-last-modified-time](#output\_cloudfront-distribution-http-last-modified-time) | Cloudfront Distribution Last Modified |
| <a name="output_cloudfront-distribution-http-version"></a> [cloudfront-distribution-http-version](#output\_cloudfront-distribution-http-version) | Cloudfront Distribution HTTP Version |
| <a name="output_cloudfront-distribution-id"></a> [cloudfront-distribution-id](#output\_cloudfront-distribution-id) | Cloudfront Distribution ID |
| <a name="output_cloudfront-distribution-status"></a> [cloudfront-distribution-status](#output\_cloudfront-distribution-status) | Cloudfront Distribution Status |
| <a name="output_cloudfront-distribution-zone-id"></a> [cloudfront-distribution-zone-id](#output\_cloudfront-distribution-zone-id) | Cloudfront Distribution Zone ID |
| <a name="output_dns-site-alias"></a> [dns-site-alias](#output\_dns-site-alias) | DNS Site Alias |
| <a name="output_dns-site-id"></a> [dns-site-id](#output\_dns-site-id) | DNS Site ID |
| <a name="output_dns-site-name"></a> [dns-site-name](#output\_dns-site-name) | DNS Site Name |
| <a name="output_gandi-domain"></a> [gandi-domain](#output\_gandi-domain) | Are we a Gandi Domain Boolean |
| <a name="output_referer-header-value"></a> [referer-header-value](#output\_referer-header-value) | Referer header value Cloudfront passes to the S3 bucket |
| <a name="output_site-certificate-arn"></a> [site-certificate-arn](#output\_site-certificate-arn) | Site Certificate ARN |
| <a name="output_site-certificate-domain-name"></a> [site-certificate-domain-name](#output\_site-certificate-domain-name) | Site Certificate domain name |
| <a name="output_site-certificate-domain-validation-options"></a> [site-certificate-domain-validation-options](#output\_site-certificate-domain-validation-options) | Site Certificate Domain Validation Options |
| <a name="output_site-certificate-expiration"></a> [site-certificate-expiration](#output\_site-certificate-expiration) | Site Certificate Expiration |
| <a name="output_site-certificate-issued"></a> [site-certificate-issued](#output\_site-certificate-issued) | Site Certificate Issued |
| <a name="output_site-certificate-status"></a> [site-certificate-status](#output\_site-certificate-status) | Site Certificate provisioning status |
<!-- END_TF_DOCS -->