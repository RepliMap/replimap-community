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
                "s3:GetEncryptionConfiguration",
                "s3:GetReplicationConfiguration",
                "s3:ListAllMyBuckets",
                "elasticache:Describe*",
                "elasticache:ListTagsForResource",
                "elasticloadbalancing:Describe*",
                "autoscaling:Describe*",
                "route53:ListHostedZones",
                "route53:GetHostedZone",
                "route53:ListResourceRecordSets",
                "route53:ListTagsForResource",
                "acm:ListCertificates",
                "acm:DescribeCertificate",
                "acm:ListTagsForCertificate",
                "cloudfront:ListDistributions",
                "cloudfront:GetDistributionConfig",
                "cloudfront:ListTagsForResource",
                "ecs:ListClusters",
                "ecs:DescribeClusters",
                "ecs:ListServices",
                "ecs:DescribeServices",
                "ecs:ListTaskDefinitionFamilies",
                "ecs:ListTaskDefinitions",
                "ecs:DescribeTaskDefinition",
                "ecs:ListTagsForResource",
                "ecr:DescribeRepositories",
                "ecr:ListTagsForResource",
                "iam:ListRoles",
                "iam:ListPolicies",
                "iam:ListInstanceProfiles",
                "iam:ListInstanceProfilesForRole",
                "iam:ListAttachedRolePolicies",
                "iam:ListRolePolicies",
                "iam:ListRoleTags",
                "iam:ListInstanceProfileTags",
                "iam:ListAccessKeys",
                "iam:GetRole",
                "iam:GetPolicy",
                "iam:GetInstanceProfile",
                "lambda:ListFunctions",
                "lambda:GetFunctionConfiguration",
                "lambda:ListEventSourceMappings",
                "lambda:ListTags",
                "lambda:GetFunctionUrlConfig",
                "lambda:GetPolicy",
                "dynamodb:ListTables",
                "dynamodb:DescribeTable",
                "dynamodb:ListTagsOfResource",
                "dynamodb:DescribeContinuousBackups",
                "dynamodb:DescribeTimeToLive",
                "sqs:GetQueueAttributes",
                "sqs:ListQueues",
                "sqs:ListQueueTags",
                "sns:GetTopicAttributes",
                "sns:ListTopics",
                "sns:ListTagsForResource",
                "logs:DescribeLogGroups",
                "logs:ListTagsLogGroup",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:ListTagsForResource",
                "secretsmanager:ListSecrets",
                "secretsmanager:DescribeSecret",
                "ssm:DescribeParameters",
                "ssm:ListTagsForResource",
                "kms:ListKeys",
                "kms:DescribeKey",
                "kms:ListAliases",
                "kms:ListResourceTags",
                "kms:GetKeyRotationStatus",
                "kms:GetKeyPolicy",
                "tag:GetResources",
                "sts:GetCallerIdentity",
                "sts:GetSessionToken"
            ],
            "Resource": "*"
        },
        {
            "Sid": "RepliMapApiGatewayRead",
            "Effect": "Allow",
            "Action": "apigateway:GET",
            "Resource": [
                "arn:aws:apigateway:*::/restapis",
                "arn:aws:apigateway:*::/restapis/*/resources",
                "arn:aws:apigateway:*::/restapis/*/resources/*/methods/*/integration",
                "arn:aws:apigateway:*::/restapis/*/stages",
                "arn:aws:apigateway:*::/restapis/*/authorizers",
                "arn:aws:apigateway:*::/domainnames",
                "arn:aws:apigateway:*::/domainnames/*/basepathmappings",
                "arn:aws:apigateway:*::/domainnames/*/apimappings",
                "arn:aws:apigateway:*::/vpclinks",
                "arn:aws:apigateway:*::/apis",
                "arn:aws:apigateway:*::/apis/*/integrations",
                "arn:aws:apigateway:*::/apis/*/routes",
                "arn:aws:apigateway:*::/apis/*/stages",
                "arn:aws:apigateway:*::/apis/*/authorizers"
            ]
        }
    ]
}
```

> **API Gateway** (graph only): `apigateway:GET` is scoped to the specific
> list endpoints RepliMap calls (REST APIs, resources/integrations, stages,
> authorizers, custom domains and their mappings, VPC links, and the v2
> equivalents). It deliberately excludes API keys, usage plans, exports and
> SDK generation, so API key values can never be read.

> **Lambda and DynamoDB**: RepliMap scans Lambda functions and event source
> mappings, and DynamoDB table definitions, so serverless chains
> (API Gateway -> Lambda -> DynamoDB/SQS) appear in the dependency graph.
> Lambda functions are graph-only (no Terraform is generated for them, because
> that would need the code package); event source mappings and DynamoDB tables
> are generated. Lambda environment variable *values* are never stored: only
> the variable names are kept, redacted at ingestion.
>
> **RepliMap never calls `lambda:GetFunction`** (it returns a presigned code
> download URL), **`lambda:InvokeFunction`**, `lambda:GetLayerVersion`, or any
> DynamoDB data-plane action (`Scan`, `Query`, `GetItem`, `BatchGetItem`,
> `ExportTableToPointInTime`). Table items and function code are never read,
> so none of those permissions are needed.
>
> `replimap deps` (Pro+) also uses the IAM-policy actions above to resolve
> *cross-service* dependencies (e.g. which policies are attached to an IAM
> role), read-only. `sts:GetSessionToken` is used only by the optional MFA
> session-refresh path.

> **DNS / certificate / CDN actions** (`route53:*`, `acm:*`, `cloudfront:*`)
> are all `List*`/`Get*`/`Describe*`. `route53:GetHostedZone` is only called
> for *private* hosted zones, to read their VPC associations. RepliMap never
> calls `acm:ExportCertificate` (it returns private keys) or
> `acm:GetCertificate`. CloudFront certificates live in `us-east-1`, so when
> you scan another region the ACM calls are also made against `us-east-1`
> (restricted to certificates a CloudFront distribution uses); if an SCP
> blocks that region the scan continues without them.
> **ECS / ECR**: the `ecs:*` / `ecr:*` actions above are metadata reads only.
> RepliMap never calls `ecr:GetAuthorizationToken`, `ecr:BatchGetImage`,
> `ecr:GetDownloadUrlForLayer` or `ecs:ExecuteCommand`, so image contents and
> container shells are unreachable. Task definition environment variable
> **values** are redacted before storage; only the variable names are kept.
> `ecs:ListTagsForResource` backs the `include=TAGS` option on the ECS
> `Describe*` calls.

### EC2 user data

Launch template `UserData` is returned by `ec2:DescribeLaunchTemplateVersions`
(covered by `ec2:Describe*`). RepliMap reads it in memory and redacts it before
anything is stored. RepliMap does **not** call `ec2:DescribeInstanceAttribute`,
so instance user data is never read. `codify` does not emit `user_data` for
instances or launch templates (it adds `ignore_changes`); see the generated
comment for how to read it yourself.

### Secrets Manager, SSM Parameter Store and KMS: metadata only

RepliMap maps *where credentials and config live and which key encrypts
them*, never the credentials themselves. The `secretsmanager:*`, `ssm:*` and
`kms:*` actions above return names, descriptions, key ids, rotation settings,
tags and key policy documents only.

RepliMap **never needs** (and the code never calls) any of these, so do not
grant them:

- `secretsmanager:GetSecretValue`
- `ssm:GetParameter`, `ssm:GetParameters`, `ssm:GetParametersByPath`
  (these return values even for plain `String` parameters)
- `kms:Decrypt` (and `kms:Encrypt`, `kms:GenerateDataKey*`)

`kms:GetKeyPolicy` reads the key policy document (never key material) so
`codify` can emit it and the first plan shows no policy change; a key whose
policy cannot be read is still scanned, without a `policy`.
Keys in other accounts, or keys you cannot describe, are skipped without
failing the scan.

### Optional: remote Terraform state for `drift --state-bucket`

The policy above grants **no** access to S3 object contents. Only if you run
`replimap drift --state-bucket`, add a separate statement scoped to the single
state object RepliMap should read (replace the bucket and key):

```json
{
    "Sid": "RepliMapReadTerraformState",
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::YOUR-STATE-BUCKET/path/to/terraform.tfstate"
}
```

If your state bucket uses SSE-KMS, the principal also needs `kms:Decrypt` on
that bucket's key.

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
| Process data locally | Access S3 object contents\* |
| Generate Terraform code | Read database contents |
| | Store your credentials |
| | Upload data to external services |

\* One exception: `replimap drift --state-bucket` reads the single Terraform
state object you point it at (to compare against live AWS) — never any other
S3 object. Everyday scanning (`scan`, `codify`, `audit`, local `drift`) never
calls `s3:GetObject`.

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
