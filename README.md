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

## Installation

[Click here to deploy Mastodon on AWS] to your AWS account.

https://console.aws.amazon.com/cloudformation/home?#/stacks/create/review?templateURL=https://s3.eu-central-1.amazonaws.com/mastodon-on-aws-cloudformation/v0.2.0/quickstart.yml&stackName=mastodon-on-aws

```
$ docker run -it tootsuite/mastodon:latest sh

$ bundle exec rake secret
758a3b431265776b9ab55910890162bb84aec0617724ca611475c3a774965f2d0aca183091d3c1a84ff3640cf7cc438c559034a2735253ee895b7a2308ac450c

$ bundle exec rake secret
c528b5cbb0236e4b0c2fe38a6d7ed1edc5fa12608c67a45690e225f005bad8bfbabfa99f7b83cb9c0981ba8fcc5fd76c68918d9bc854bd158c2c23fd6df89abc

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
$ cd mastodon-on-aws
$ npm install
$ aws cloudformation package --template-file mastodon.yaml --s3-bucket <S3_BUCKET> --output-template-file packaged.yml
$ aws cloudformation deploy --template-file packaged.yml --stack-name mastodon-on-aws --capabilities CAPABILITY_IAM --parameter-overrides "DomainName=<DOMAIN_NAME>" "SecretKeyBase=<SECRET_KEY_BASE>" "OtpSecret=<OTP_SECRET>" "VapidPrivateKey=<VAPID_PRIVATE_KEY>" "VapidPublicKey=<VAPID_PUBLIC_KEY>"
```

## TODOs

-[ ] Use `noeviction` maxmemory policy to avoid REdis evicts Sidekiq data.
-[ ] Optimize for costs.
-[ ] Enable auto-scaling for ECS services.
