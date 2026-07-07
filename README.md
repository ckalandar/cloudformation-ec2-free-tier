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
Step 1 - Create OIDC Provider

AWS Console

IAM
  └── Identity Providers
         └── Add Provider

Provider Type

OpenID Connect

Provider URL

https://token.actions.githubusercontent.com

Audience

sts.amazonaws.com

After creation you'll have:

arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com
Step 2 - Create IAM Role

Name:

GitHubActionsCloudFormationRole

Trust Policy:

Replace <ACCOUNT_ID> with yours.

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:ckalandar/cloudformation-ec2-free-tier:*"
        }
      }
    }
  ]
}
Step 3 - IAM Permissions

For learning:

Attach:

AdministratorAccess

I wouldn't do this in production, but for a personal lab it's the fastest way to get moving.

Step 4 - GitHub Secret

Repository:

Settings → Secrets and Variables → Actions

Create:

AWS_ROLE_ARN

Value:

arn:aws:iam::<ACCOUNT_ID>:role/GitHubActionsCloudFormationRole
