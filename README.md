# Mastodon on AWS

Want to host your own Mastodon instance on AWS? Here you go.

The architecture consists of the following building blocks.

* Application Load Balancer (ALB)
* ECS and Fargate
* RDS Aurora Serverless
* ElastiCache (Redis)
* S3
* SES
* CloudWatch
* IAM
* KMS
* Route 53

## Prerequisites

First, you need an AWS account.

Second, a top-level or sub domain where you are able to configure a `NS` record to delegate to the Route 53 nameservers is required. For example, you could register a domain with Rout 53 or use an exsisting domain and add an `NS` record to the hosted zone.

## Costs for running Mastodon on AWS

Estimating costs for AWS is not trivial. My estimation assumes a small Mastodon instance for 1-50 users. The architecture's monthly charges are about $60 per month. The following table lists the details (us-east-1).

| Service | Configuration | Monthly Costs (USD) |
| ---------- | ------------- | ----------------------------: |
| ECS + Fargate | 3 Spot Tasks x (0.25 CPU + 0.5 GB) | $8.66 |
| RDS for Postgres | t4g.micro (Multi-AZ) | $23.61 |
| ElastiCache for Redis | t4g.micro (Single-AZ) | $11.52 |
| ALB | Load Balancer Hours | $16.20 |
| S3 | 25 GB + requests | $0.58 |
| Route 53 | Hosted Zone | $0.50 |
| **Total** | | $61.08 |

Please note that the cost estimation is not complete and costs differ per region. For example, the estimation does not include network traffic, CloudWatch, SES, and domain. [Monitor your costs](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-create.html)!

## Installation

[Click here to deploy Mastodon on AWS](https://console.aws.amazon.com/cloudformation/home?#/stacks/create/review?templateURL=https://s3.eu-central-1.amazonaws.com/mastodon-on-aws-cloudformation/v0.5.0/quickstart.yml&stackName=mastodon-on-aws) to your AWS account.

To generate the required secrets and keys use the following commands.

```
# Start Docker container locally
$ docker run -it tootsuite/mastodon:latest sh

# Generate SECRET_KEY_BASE
$ bundle exec rake secret
758a3b431265776b9ab55910890162bb84aec0617724ca611475c3a774965f2d0aca183091d3c1a84ff3640cf7cc438c559034a2735253ee895b7a2308ac450c

# Generate OTP_SECRET
$ bundle exec rake secret
c528b5cbb0236e4b0c2fe38a6d7ed1edc5fa12608c67a45690e225f005bad8bfbabfa99f7b83cb9c0981ba8fcc5fd76c68918d9bc854bd158c2c23fd6df89abc

# Generate VAPID_PRIVATE_KEY and VAPID_PUBLIC_KEY
$ bundle exec rake mastodon:webpush:generate_vapid_key
VAPID_PRIVATE_KEY=am3vlPBGQGv7Rl3xOKXSv7lRYyWfZITItb88FXX9IOs=
VAPID_PUBLIC_KEY=BMGkIr1PaK4v7Kut7q7eoHtWxu9gEBQ5BeV28xOIR9c9VIvDWvOViTn1SV5G2LIEFGWo0f1dQka-UynR58WMn2Y=
```

## Administration

```
aws ecs execute-command --cluster <CLUSTER_NAME> --container app --command /bin/bash --interactive --task <TASK_ID>
```


## Development

IaC based on [cfn-modules](https://github.com/cfn-modules/docs).

```
$ npm install
$ aws cloudformation package --template-file mastodon.yaml --s3-bucket <S3_BUCKET> --output-template-file packaged.yml
$ aws cloudformation deploy --template-file packaged.yml --stack-name mastodon-on-aws --capabilities CAPABILITY_IAM --parameter-overrides "DomainName=<DOMAIN_NAME>" "SecretKeyBase=<SECRET_KEY_BASE>" "OtpSecret=<OTP_SECRET>" "VapidPrivateKey=<VAPID_PRIVATE_KEY>" "VapidPublicKey=<VAPID_PUBLIC_KEY>"
```
