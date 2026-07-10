# RepliMap IAM Policy

RepliMap requires **read-only** access to scan your AWS resources. We never create, modify, or delete anything.

---

## Recommended Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RepliMapReadOnly",
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "rds:Describe*",
                "rds:ListTagsForResource",
                "s3:GetBucket*",
                "s3:GetLifecycleConfiguration",
                "s3:ListAllMyBuckets",
                "elasticache:Describe*",
                "elasticache:ListTagsForResource",
                "elasticloadbalancing:Describe*",
                "autoscaling:Describe*",
                "iam:ListRoles",
                "iam:ListPolicies",
                "iam:ListInstanceProfiles",
                "iam:ListRoleTags",
                "iam:GetRole",
                "iam:GetPolicy",
                "sqs:GetQueueAttributes",
                "sqs:ListQueues",
                "sqs:ListQueueTags",
                "sns:GetTopicAttributes",
                "sns:ListTopics",
                "sns:ListTagsForResource",
                "logs:DescribeLogGroups",
                "logs:ListTagsLogGroup",
                "cloudwatch:DescribeAlarms",
                "tag:GetResources",
                "sts:GetCallerIdentity"
            ],
            "Resource": "*"
        }
    ]
}
```

---

## Quick Setup

### Option 1: Create Dedicated IAM User

```bash
# Create user
aws iam create-user --user-name replimap-scanner

# Attach inline policy
aws iam put-user-policy \
    --user-name replimap-scanner \
    --policy-name RepliMapReadOnly \
    --policy-document file://replimap-policy.json

# Create access keys
aws iam create-access-key --user-name replimap-scanner

# Configure AWS CLI profile
aws configure --profile replimap
```

### Option 2: Use Existing Profile

If you have an existing profile with sufficient read permissions:

```bash
replimap scan --profile your-existing-profile --region us-east-1
```

### Option 3: IAM Role (EC2/ECS)

For running RepliMap on EC2 or ECS, create a role with the policy above and attach it to your instance/task.

---

## Verify Permissions

```bash
# Check identity
aws sts get-caller-identity --profile replimap

# Dry run scan (checks permissions without full scan)
replimap doctor --profile replimap
```

---

## Security Guarantees

| ✅ RepliMap Does | ❌ RepliMap Does NOT |
|------------------|----------------------|
| Read resource metadata | Create any resources |
| Read tags and configurations | Modify any resources |
| Read network topology | Delete any resources |
| Process data locally | Access S3 object contents |
| Generate Terraform code | Read database contents |
| | Store your credentials |
| | Upload data to external services |

---

## Minimal Policy (VPC Only)

If you only need to scan networking resources:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RepliMapVPCOnly",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSecurityGroupRules",
                "ec2:DescribeRouteTables",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeNatGateways",
                "ec2:DescribeNetworkAcls",
                "ec2:DescribeVpcEndpoints",
                "ec2:DescribeTags",
                "sts:GetCallerIdentity"
            ],
            "Resource": "*"
        }
    ]
}
```

---

## Troubleshooting

### "Access Denied" Errors

1. Verify your profile: `aws sts get-caller-identity --profile YOUR_PROFILE`
2. Check the specific action: The error message will show which permission is missing
3. Add the missing `Describe*` permission to your policy

### "ExpiredToken" Errors

If using temporary credentials (SSO, assumed role):

```bash
# Refresh SSO session
aws sso login --profile your-sso-profile

# Or re-assume role
aws sts assume-role --role-arn arn:aws:iam::ACCOUNT:role/ROLE --role-session-name replimap
```

---

## Questions?

- Open an [issue](https://github.com/RepliMap/replimap-community/issues)
- Email: support@replimap.com
