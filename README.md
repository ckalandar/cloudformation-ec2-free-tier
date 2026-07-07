# cloudformation-ec2-free-tier
# Repo Structure
cloudformation-ec2-free-tier/
├── .github/
│   └── workflows/
│       └── validate.yml
│
├── templates/
│   └── ec2.yaml
│
├── guard/
│   └── ec2.guard
│
├── taskcat.yml
├── .cfnlintrc
├── Gemfile
├── requirements.txt
├── README.md
└── .gitignore

# CloudFormation EC2 Free Tier
####################CloudFormation

Deploys:

- Amazon Linux 2023 EC2
- SSM Session Manager Access
- IAM Role
- Security Group

## Install

```bash
python3 -m venv .venv

source .venv/bin/activate

pip install -r requirements.txt
```

## Lint

```bash
cfn-lint templates/ec2.yaml
```

## Guard

```bash
cfn-guard validate \
  --rules guard \
  --data templates/ec2.yaml
```

## cfn-nag

```bash
gem install cfn-nag

cfn_nag_scan \
  --input-path templates/ec2.yaml
```

## TaskCat

```bash
taskcat test run
```

## Deploy

```bash
aws cloudformation deploy \
  --template-file templates/ec2.yaml \
  --stack-name ec2-free-tier \
  --capabilities CAPABILITY_NAMED_IAM
```

## Connect

AWS Console

Systems Manager

Session Manager

Start Session

## Initial Setup

What You'll Build
GitHub Repository
        │
        ▼
CodePipeline
        │
        ▼
CodeBuild
        │
        ├── cfn-lint
        ├── cfn-guard
        ├── cfn-nag
        ├── taskcat
        ▼
CloudFormation Deploy
        ▼
EC2 Instance

This is much closer to what many AWS-centric enterprises use.

Step 1: Create an S3 Bucket

CodePipeline stores artifacts here.

Example:

cfn-ec2-artifacts-<account-id>

AWS Console:

S3
 → Create bucket
Step 2: Create a CodeBuild Service Role

IAM → Create Role

Trusted Entity:

AWS Service
CodeBuild

Attach:

AdministratorAccess

For learning.

Later you'll replace with least privilege.

Role name:

CodeBuild-CFN-Role
Step 3: Create buildspec.yml

Add this to your repo root:

version: 0.2

phases:

  install:
    runtime-versions:
      python: 3.11

    commands:

      - pip install --upgrade pip
      - pip install "setuptools<81"
      - pip install -r requirements.txt

      - wget https://github.com/aws-cloudformation/cloudformation-guard/releases/download/3.2.0/cfn-guard-v3-x86_64-linux-latest.tar.gz

      - tar -xzf cfn-guard-v3-x86_64-linux-latest.tar.gz

      - chmod +x cfn-guard-v3-x86_64-linux-latest/cfn-guard

      - mv cfn-guard-v3-x86_64-linux-latest/cfn-guard /usr/local/bin/

      - gem install cfn-nag

  build:
    commands:

      - echo "Running cfn-lint"
      - cfn-lint templates/ec2.yaml

      - echo "Running cfn-guard"
      - cfn-guard validate --rules guard --data templates/ec2.yaml

      - echo "Running cfn-nag"
      - cfn_nag_scan --input-path templates/ec2.yaml

artifacts:
  files:
    - '**/*'

This replaces your validate.yml.

Step 4: Create CodeBuild Project

AWS Console:

CodeBuild
 → Create Project

Name:

ec2-free-tier-build

Source:

GitHub

Repository:

Your repo.

Environment:

Managed Image
Amazon Linux
Standard

Service Role:

CodeBuild-CFN-Role

Buildspec:

Use buildspec.yml
Step 5: Test Build

Run:

Start Build

Verify:

cfn-lint PASS
cfn-guard PASS
cfn-nag PASS
Step 6: Create CloudFormation Deploy Stage

You have two options.

Option A (Recommended)

Add deployment inside buildspec.

After validation:

VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=is-default,Values=true \
  --query 'Vpcs[0].VpcId' \
  --output text)

SUBNET_ID=$(aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'Subnets[0].SubnetId' \
  --output text)

aws cloudformation deploy \
  --stack-name ec2-free-tier \
  --template-file templates/ec2.yaml \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides \
  VpcId=$VPC_ID \
  SubnetId=$SUBNET_ID
Option B

Use a dedicated CloudFormation Deploy stage in CodePipeline.

This is how larger enterprises usually separate validation and deployment.

Step 7: Create CodePipeline

AWS Console:

CodePipeline
 → Create Pipeline

Pipeline name:

ec2-free-tier-pipeline

Source Stage:

GitHub

Connect GitHub account.

Repository:

cloudformation-ec2-free-tier

Branch:

main

Build Stage:

CodeBuild

Select:

ec2-free-tier-build

Deploy Stage:

Either:

CodeBuild deployment

or

CloudFormation Deploy Action
Step 8: Add TaskCat

Your GitHub pipeline already has TaskCat.

For AWS-native pipeline:

Add to buildspec:

taskcat test run

after generating .taskcat.yml.

This gives:

CodeBuild
 ├─ cfn-lint
 ├─ cfn-guard
 ├─ cfn-nag
 ├─ taskcat
 └─ deploy
Step 9: Add Manual Approval (Enterprise Pattern)

Between Build and Deploy:

CodePipeline
   ↓
Validate
   ↓
Manual Approval
   ↓
Deploy

This is extremely common in production.

Final Architecture
GitHub
   │
   ▼
CodePipeline
   │
   ▼
CodeBuild
   │
   ├── cfn-lint
   ├── cfn-guard
   ├── cfn-nag
   ├── taskcat
   ▼
Manual Approval
   ▼
CloudFormation Deploy
   ▼
EC2
