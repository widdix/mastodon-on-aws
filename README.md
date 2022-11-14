# Mastodon on AWS

Want to host your own Mastodon instance on AWS? Here you go.

## Development

IaC based on [cfn-modules](https://github.com/cfn-modules/docs).

```
cd mastodon-on-aws
npm install
aws cloudformation package --template-file mastodon.yaml --s3-bucket <S3_BUCKET> --output-template-file packaged.yml
aws cloudformation deploy --template-file packaged.yml --stack-name mastodon-on-aws --capabilities CAPABILITY_IAM --parameter-overrides "DomainName=<DOMAIN_NAME>" "SecretKeyBase=<SECRET_KEY_BASE>" "OtpSecret=<OTP_SECRET>" "VapidPrivateKey=<VAPID_PRIVATE_KEY>" "VapidPublicKey=<VAPID_PUBLIC_KEY>"
```

## TODOs

-[ ] Use custom resource transforming outputs of `AWS::IAM::AccessKey` into SMTP credentials.
-[ ] Use `noeviction` maxmemory policy to avoid REdis evicts Sidekiq data.
-[ ] Optimize for costs.
-[ ] Enable auto-scaling for ECS services.

## Administration

```
aws ecs execute-command --cluster <CLUSTER_NAME> --container app --command /bin/bash --interactive --task <TASK_ID>
```
