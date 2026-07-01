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
