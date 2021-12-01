# c2c-npm

Private NPM container that can backup to efs

## Running in AWS ECS Fargate

### Architecture:

```
Clients
 |
 | (HTTPS)
 v
Application Load Balancer
 |
 | (HTTP)
 v
ECS Fargate
Docker image (Verdaccio)
 |
 | (NFS)
 v
Elastic File System
```
Estimated monthly cost for a small installation (in us-east-1):
- ALB (1 LCU average): $22.265/mo
- ECS Fargate (0,25 vCPU and 0,5 GB memory): $0.437/mo
- EFS (5gb): $1.5/mo
- Data transfer: (10gb): $0.9/mo
- TOTAL: ~ $25/mo

### How to run:

Two AWS CloudFormation stacks are provided within the `aws-cfn` directory to run this npm registry within an AWS ECS Fargate cluster.

- `vpc.yml` Create a VPC to run the ECS cluster.
- `ecs.yml` The ECS cluster.

1. Add a Domain within Certificate
2. Create a VPC
    ```
    aws cloudformation deploy --template-file vpc.yml --stack-name c2c-npm-vpc \    
    --capabilities CAPABILITY_NAMED_IAM
    ```
3. Create the ECS cluster and run Fargate
    ```
    aws cloudformation deploy --template-file ecs.yml --stack-name c2c-npm-service \
    --parameter-overrides AppName=c2c-npm \
    VPC=vpc-0b86a98e6a162eae2 \
    SubnetOne=subnet-06f5679881e4753ed SubnetTwo=subnet-09656dd29f4f331cc  \
    CertificateArn=arn:aws:acm:ap-northeast-1:074635850563:certificate/8e2c0f9e-176a-424f-988a-c84581442e71 \
    HostedZoneId=Z046069732VOUBKR2L5UG \
    Domain=registry.c2c-platform.com \
    --capabilities CAPABILITY_NAMED_IAM
    ```

##### Notes:

- Make sure your Amazon EFS file system is successfully mounted to your Fargate container ([references])
- First need to supply a copy of [config.yaml] in `/verdaccio/conf` directory; the Fargate container will not start properly if this file is missing
- Need to run `sudo chown -R 10001:65533 /path/for/verdaccio` otherwise you will get permission errors at runtime


[config.yaml]: config.yaml "config.yaml"
[references]: https://aws.amazon.com/premiumsupport/knowledge-center/ecs-fargate-mount-efs-containers-tasks/