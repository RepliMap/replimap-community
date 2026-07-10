<!--
<p align="center">
  <img src="assets/logo.png" alt="RepliMap Logo" width="120" />
</p>
-->

<h1 align="center">RepliMap</h1>

<p align="center">
  <strong>AWS Infrastructure Intelligence Engine</strong>
</p>

<p align="center">
  Walk into any brownfield AWS account. Scan it, map every dependency, and turn ClickOps into import-ready Terraform. 100% on your machine.
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> •
  <a href="#features">Features</a> •
  <a href="#-limitations--scope">Limitations</a> •
  <a href="#-frequently-asked-questions">FAQ</a> •
  <a href="#documentation">Docs</a>
</p>

<p align="center">
  <a href="https://pypi.org/project/replimap/">
    <img src="https://img.shields.io/pypi/v/replimap?color=blue&label=PyPI" alt="PyPI" />
  </a>
  <img src="https://img.shields.io/badge/python-3.10+-blue.svg" alt="Python 3.10+" />
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/license-BSL--1.1-green.svg" alt="License" />
  </a>
</p>

<p align="center">
  <a href="https://dev.to/davidlu1001/two-terraform-traps-that-burned-me-hidden-defaults-circular-dependencies-4n74">
    <img src="https://img.shields.io/badge/Dev.to-Technical%20Deep%20Dive-0A0A0A?style=flat-square&logo=dev.to&logoColor=white" alt="Dev.to" />
  </a>
  <a href="https://news.ycombinator.com/item?id=46720620">
    <img src="https://img.shields.io/badge/Hacker%20News-Join%20Discussion-FF6600?style=flat-square&logo=ycombinator&logoColor=white" alt="Hacker News" />
  </a>
  <a href="https://twitter.com/davidlu1001">
    <img src="https://img.shields.io/badge/Twitter-Follow%20Updates-1DA1F2?style=flat-square&logo=twitter&logoColor=white" alt="Twitter" />
  </a>
</p>

<p align="center">
  <img src="assets/demo-scan.gif" alt="RepliMap scan demo — masked credentials, live progress" width="700" />
</p>

---

> **👋 About This Repository**
> RepliMap is a **commercial tool** built with a **"Local-First"** architecture.
> This repository (`replimap-community`) hosts **documentation** and **issue tracking**.
> The engine is distributed via [PyPI](https://pypi.org/project/replimap/).
> Your AWS credentials and infrastructure data **never leave your machine** — the only network call is license validation, and generated reports load nothing from the internet.

---

> **RepliMap** is an **AWS Infrastructure Intelligence Engine** for DevOps and SRE teams.
> *Not to be confused with RepliMap HD mapping software for autonomous vehicles.*

---

## 💡 TL;DR

**RepliMap** is a read-only CLI tool that reverse-engineers existing AWS infrastructure into a Terraform adoption starting point — clean resource files plus an import scaffold that validates on day one.

**Core capabilities:**
- **Graph Engine**: Uses Tarjan's algorithm to detect Strongly Connected Components (SCCs) and automatically resolve circular dependencies (e.g., Security Groups referencing each other)
- **IaC Coverage**: Matches your live account against your Terraform state files and reports what exists in **none of them** — the question `terraform plan` structurally cannot answer
- **Smart Sanitization**: Filters read-only fields (like `root_block_device.device_name`) that cause accidental resource destruction on `terraform apply`
- **Local-First**: Your AWS credentials and infrastructure data never leave your machine

**Use RepliMap when:**
- You inherited a ClickOps AWS account and need Terraform
- `terraform import` keeps failing with cycle errors
- You need to prove what exists (and what nobody manages) for a hand-off or audit

**Keywords**: AWS, Terraform, Infrastructure as Code, IaC, reverse engineering, circular dependencies, Tarjan algorithm, SCC, brownfield migration, ClickOps to GitOps, Security Group cycles, terraform import alternative, IaC coverage

---

## The Problem

You inherited an AWS account. Or maybe you built it yourself over 3 years of "just one more click."

Now you have:
- 🤷 **Hundreds of resources** and no idea what connects to what
- 😰 **No Terraform** — or a state file that covers half of what's really there
- 📋 **A hand-off (or an audit) coming** — and no way to prove what exists

Sound familiar?

## The Solution

**RepliMap scans your AWS, builds a dependency graph, and turns it into deliverables.**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   $ replimap -p prod scan                                               │
│                                                                         │
│   ✓ Found 1,584 resources                                               │
│   ✓ Mapped 1,470 dependencies                                           │
│                                                                         │
│   Next: replimap graph / codify                                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

That's real output from a 1,584-resource production account — the same account
where IaC coverage found the entire data layer, all 27 RDS instances, in no
Terraform state at all.

---

## Features

### 🔍 Scan & Understand

**See what actually exists — and what breaks if you touch it.**

RepliMap builds a complete dependency graph of your AWS account. Parallel
scanners cover 1,500+ resource accounts in minutes, with per-item retries so a
transient AWS error never silently drops a resource.

```bash
# Scan your AWS account
replimap -p prod -r ap-southeast-2 scan

# Visualize dependencies — one self-contained HTML file, renders air-gapped
replimap -p prod -r us-east-1 graph -o architecture.html

# "What happens if I delete this security group?"
replimap -p prod -r us-east-1 deps sg-0a1b2c3d4e
```

The interactive graph nests resources into their VPC/subnet containers, and
clicking any node highlights its blast radius. D3 is inlined — zero CDN, zero
outbound requests — so you can open it on an air-gapped machine or hand it to
a client as-is.

### 🏗️ Codify: ClickOps to Terraform

**Stop writing HCL by hand. Let the engine do it.**

`codify` scans your existing resources and generates a Terraform **adoption
starting point** — clean resource files grouped per service, plus an import
scaffold with the correct import ID format for every resource type.

```bash
replimap codify -p prod -r us-east-1 -o ./terraform

# Output structure
terraform/
├── vpc.tf / networking.tf / security_groups.tf
├── ec2.tf / compute.tf / alb.tf
├── rds.tf / elasticache.tf / s3.tf / storage.tf / messaging.tf
├── providers.tf / versions.tf
└── imports.tf        # import scaffold (Pro) — one block per resource
```

**Why RepliMap's generation holds up on real accounts:**

- ✅ **Handles `root_block_device` defaults** — prevents accidental EC2 replacement ([see the trap](https://www.reddit.com/r/devops/comments/1qiun82/psa_the_root_block_device_gotcha_that_almost_cost/))
- ✅ **Resolves circular dependencies** — auto-splits Security Group rules
- ✅ **Filters AWS system tags** — no more `aws:*` tag rejection errors
- ✅ **Lifecycle protection** — `prevent_destroy` on databases and storage by default
- ✅ **Validates on day one** — run `terraform init && terraform validate` on the output immediately

The free tier generates every resource `.tf` file for the whole account, so
you can judge the quality before paying for anything.

### 📊 IaC Coverage: What Does Nobody Manage?

**The question `terraform plan` structurally cannot answer.**

`terraform plan` only sees what's already in state. IaC Coverage matches your
live account against one or more state files and reports what exists in
**none of them** — triaged into real ClickOps, embedded resources (e.g. EBS
volumes owned by an instance), and likely auto-created (service-linked roles).

```bash
# Match live resources against your state files (local or S3)
replimap codify -p prod \
  --coverage-state s3://tf-states/prod/terraform.tfstate \
  --coverage-state-dir ./states
```

Everyone gets the per-type counts and coverage percentage; the full resource
list and machine-readable `coverage.json` are Pro.

### ✅ Audit Evidence

**Evidence, not screenshots.**

Run a SOC 2-mapped audit against live AWS (Checkov-powered) and export the
findings as an evidence package your auditor can actually use. Frameworks:
SOC 2, APRA CPS 234, RBNZ BS11, NZISM.

```bash
# Full terminal summary (score, grade, top issues) — free
replimap -p prod -r us-east-1 audit

# HTML report + SOC 2 evidence export — Pro
replimap audit -p prod --state ./terraform.tfstate
```

### 🛡️ IAM Generation

**Least-privilege policies from actual dependencies.**

```bash
replimap iam for-resource -p prod -r i-0abc123 -s runtime_read
```

Graph-aware traversal generates permissions only for the resources your
workload actually touches, with precise ARNs.

### 🔄 Drift — an honest note

Attribute-level drift comparison ships free and is marked **experimental** —
on real accounts, `terraform plan` is still the better answer for "did what I
manage change?". What RepliMap ships and stands behind is the opposite
question: **IaC Coverage** — what exists that no state file manages.

---

## How It Works: The Brownfield Migration Problem

RepliMap is not just a `terraform import` wrapper. It's a **graph-based engine** designed to handle the "Brownfield Migration" problem where standard tools fail.

**The Core Problem**: Terraform requires a **Directed Acyclic Graph (DAG)** — it needs to know "create A before B." But AWS allows **cycles**. The classic example:

```
     ┌──────────────┐
     ▼              │
  [SG-App]       [SG-DB]
     │              ▲
     └──────────────┘
```

`SG-App` allows traffic to `SG-DB`. `SG-DB` allows traffic from `SG-App`. A cycle. Standard tools crash or produce broken code. RepliMap solves this.

### Step 1: Cycle Detection (Tarjan's Algorithm)

We use **Tarjan's algorithm** to find **Strongly Connected Components (SCCs)** — clusters of resources that are circularly dependent.

- If a resource is part of a normal DAG → leave it alone
- If a group forms a cycle (SCC) → flag for surgery

### Step 2: Cycle Resolution ("Shell & Fill" Pattern)

For each SCC, we apply the **Shell & Fill** pattern:

1. **Shell**: Strip the resource to pure identity (e.g., `aws_security_group` with no inline rules)
2. **Fill**: Extract logic into standalone resources (e.g., `aws_security_group_rule`)

```
Before (Cycle - Terraform fails):
  [SG-App with rules] ←→ [SG-DB with rules]

After (DAG - Terraform succeeds):
  [SG-App empty] ← [Rule: App→DB]
  [SG-DB empty]  ← [Rule: DB←App]
```

### Step 3: Sanitization

The AWS API returns fields that look normal but are **read-only** in Terraform (e.g., `root_block_device.device_name`).

If you include them in your code, Terraform shows **"must be replaced"** and tries to destroy your instance.

RepliMap's Sanitizer automatically filters these dangerous fields before generating HCL.

---

## Quick Start

### Installation

```bash
# Using pipx (recommended - isolated environment)
pipx install replimap

# Using pip
pip install replimap

# Verify installation
replimap --version
```

### Your First Scan

```bash
# 1. Configure AWS credentials (if not already done)
aws configure --profile myaccount

# 2. Scan your infrastructure
replimap -p myaccount -r us-east-1 scan

# 3. Explore the results
replimap -p myaccount -r us-east-1 graph -o architecture.html
open architecture.html
```

### Generate Terraform

```bash
# Generate Terraform from scanned infrastructure
replimap codify -p prod -r us-east-1 -o ./terraform

# Validate against the real provider schema
cd terraform
terraform init
terraform validate
```

---

## 📖 Commands

| Command | Description |
|---------|-------------|
| `replimap scan` | Scan AWS resources and build dependency graph |
| `replimap codify` | Turn ClickOps AWS into a Terraform adoption starting point + import scaffold |
| `replimap graph` | Interactive dependency graph (self-contained HTML) |
| `replimap deps` | Explore dependencies and blast radius for a resource |
| `replimap analyze` | Critical resources, SPOFs, blast radius from the cached scan |
| `replimap audit` | Security & compliance audit (SOC 2 / APRA / RBNZ / NZISM) |
| `replimap iam` | Generate least-privilege IAM policies from graph analysis |
| `replimap cost` | Estimate monthly AWS costs (estimates, clearly labeled) |
| `replimap drift` | Drift detection between Terraform state and AWS (experimental) |
| `replimap doctor` | Environment health checks |

Run `replimap --help` for the full list (cache management, license, error
explain, and more).

---

## 🔧 Configuration

### AWS Credentials

RepliMap uses the standard AWS credential chain:

```bash
# Option 1: AWS CLI profile (recommended)
replimap -p prod scan

# Option 2: Environment variables
export AWS_ACCESS_KEY_ID=xxx
export AWS_SECRET_ACCESS_KEY=xxx
replimap scan

# Option 3: IAM role (EC2/ECS)
replimap scan  # Auto-detects instance role
```

### Required IAM Permissions

RepliMap only needs **read-only** access. See [IAM_POLICY.md](IAM_POLICY.md)
for the complete minimal policy.

### Supported Resources

RepliMap scans **30+ AWS resource types** — the core of a typical production
account:

| Category | Resources |
|----------|-----------|
| **Compute** | EC2, Auto Scaling Groups, Launch Templates, EBS, EIP |
| **Database** | RDS (+ subnet & parameter groups), ElastiCache (+ subnet groups) |
| **Network** | VPC, Subnet, Security Group, Route Table, NACL, IGW, NAT Gateway, VPC Endpoint, ALB/NLB (+ listeners & target groups) |
| **Storage** | S3 buckets & bucket policies |
| **Security** | IAM Roles, IAM Policies, Instance Profiles |
| **Messaging & Monitoring** | SQS, SNS, CloudWatch log groups & alarms |

New types are added based on what real scans encounter — if your account
leans on something we don't cover yet,
[tell us](https://github.com/RepliMap/replimap-community/issues).

---

## ⚠️ Limitations & Scope

Understanding what RepliMap can and cannot do helps set correct expectations.

### ✅ What RepliMap Does

| Capability | Description |
|------------|-------------|
| **Read-only scanning** | Uses only `Describe*`, `List*` and `Get*` AWS API calls |
| **Terraform generation** | Clean HCL resource files + import scaffold — an adoption starting point that validates on day one |
| **IaC coverage** | Matches live resources against your state files; reports what nobody manages |
| **Circular dependency resolution** | Automatically splits Security Group cycles |
| **Read-only field sanitization** | Prevents accidental resource recreation |
| **Local execution** | All processing happens on your machine |

### ❌ What RepliMap Does NOT Do

| Limitation | Explanation |
|------------|-------------|
| **No multi-cloud** | AWS only. Azure and GCP are not supported. |
| **No CloudFormation/Pulumi/CDK output** | Terraform (HCL) only. |
| **No auto-apply** | Does not run `terraform apply`. You review and apply manually. |
| **No write operations** | Never creates, modifies, or deletes AWS resources. |
| **No data access** | Does not read S3 bucket contents, RDS data, or secrets values. |
| **No credential storage** | AWS credentials are never stored or transmitted. |

### 📊 Current Resource Coverage

30+ resource types covering compute, database, network, storage, IAM,
messaging, and monitoring. Notable gaps today: Lambda, ECS/EKS, DynamoDB,
KMS, Secrets Manager — if you need one of these,
[open an issue](https://github.com/RepliMap/replimap-community/issues) and it
gets prioritized by demand.

Full list: See [Supported Resources](#supported-resources) section above.

---

## ❓ Frequently Asked Questions

### General

<details>
<summary><strong>What is RepliMap?</strong></summary>

RepliMap is a CLI tool that scans your AWS account and generates a Terraform
adoption starting point. Unlike simple export tools, it builds a dependency
graph of your infrastructure and intelligently resolves issues (like circular
dependencies and read-only fields) that would otherwise cause
`terraform plan` to fail or destroy resources.

</details>

<details>
<summary><strong>How is RepliMap different from Terraformer?</strong></summary>

| Aspect | RepliMap | Terraformer |
|--------|----------|-------------|
| Dependency graph | ✅ Full graph with cycle detection | ❌ No dependency tracking |
| Circular dependency handling | ✅ Auto-splits Security Group cycles | ❌ You fix manually |
| Read-only field sanitization | ✅ Prevents "must be replaced" errors | ❌ Exports raw API values |
| IaC coverage | ✅ Match live account vs state files | ❌ None |
| Visualization | ✅ Interactive, air-gapped HTML | ❌ None |

</details>

<details>
<summary><strong>How is RepliMap different from Former2?</strong></summary>

Former2 runs in your browser and has memory limitations for large accounts.
RepliMap is a CLI tool that runs locally, handles 1,500+ resource accounts
(verified on real production scans), and includes dependency analysis that
Former2 lacks.

</details>

### Technical

<details>
<summary><strong>How does RepliMap handle circular dependencies?</strong></summary>

RepliMap uses **Tarjan's algorithm** to detect **Strongly Connected Components (SCCs)** in the dependency graph.

When cycles are found (e.g., `SG-App` references `SG-DB`, and `SG-DB` references `SG-App`), RepliMap automatically:

1. Creates "empty shell" Security Groups (no inline rules)
2. Extracts rules into separate `aws_security_group_rule` resources
3. Rules reference the shell IDs, breaking the cycle

This is the "Shell & Fill" pattern — the only way to satisfy Terraform's DAG requirement while preserving AWS's actual graph structure.

</details>

<details>
<summary><strong>What is the "root_block_device trap"?</strong></summary>

When importing EC2 instances, the AWS API returns `device_name` (e.g., `/dev/xvda`) which looks like a normal attribute. But in Terraform, it's read-only — if you include it in your code, Terraform will show "must be replaced" and try to destroy/recreate your instance.

RepliMap automatically filters these dangerous read-only fields. [Read the full story →](https://www.reddit.com/r/devops/comments/1qiun82/psa_the_root_block_device_gotcha_that_almost_cost/)

</details>

<details>
<summary><strong>What AWS permissions are required?</strong></summary>

Read-only only. RepliMap uses only `Describe*`, `List*` and `Get*` API calls. See [IAM_POLICY.md](IAM_POLICY.md) for the minimal IAM policy you can copy-paste.

</details>

<details>
<summary><strong>Is my data safe?</strong></summary>

Yes. RepliMap runs entirely on your local machine. Your AWS credentials
(`~/.aws/credentials`) never leave your laptop. The only network call to
RepliMap servers is license validation — no infrastructure data is ever
transmitted, and license entitlements are verified locally via Ed25519
signatures. Generated reports and graphs are self-contained files that load
nothing from the internet. Run `replimap --privacy` for the full statement.

</details>

<details>
<summary><strong>Does RepliMap work air-gapped?</strong></summary>

Scanning and all reports run fully offline once your license is activated
(Pro: 7-day offline grace, Team: 14 days between checks). Every generated
report — including the interactive dependency graph — is a self-contained
file. Fully offline activation for permanently air-gapped environments is
part of the Sovereign plan.

</details>

### Roadmap

<details>
<summary><strong>Does RepliMap support Azure or GCP?</strong></summary>

No. AWS only — and honestly, that focus is the point. Multi-cloud would be
considered only on overwhelming demand.

</details>

<details>
<summary><strong>Will RepliMap support CloudFormation, Pulumi or CDK output?</strong></summary>

Terraform (HCL) is the only supported output today, and nothing else is
committed. If you'd pay for another format,
[tell us](https://github.com/RepliMap/replimap-community/issues) — demand
drives the roadmap.

</details>

---

## 💼 Pricing

**Unlimited scanning. Pay only when you export.**

### Community (Free)

- ✅ Unlimited scans, unlimited resources, 1 AWS account
- ✅ Interactive dependency graph (self-contained HTML)
- ✅ Terraform resource files — full account
- ✅ IaC coverage summary (unmanaged counts)
- ✅ Audit score + terminal summary

### Pro ($29/mo)

- ✅ Everything in Community
- ✅ Unlimited AWS accounts (fair use)
- ✅ `imports.tf` — import scaffold for the whole account
- ✅ Audit HTML report + SOC 2 evidence export
- ✅ Full unmanaged-resource list + `coverage.json`
- ✅ 7-day offline grace period · 48h email support

### Team ($99/mo)

- ✅ Everything in Pro, 5 team members
- ✅ CI mode with blocking checks (`--fail-on`)
- ✅ Custom webhook payloads · custom report author tag
- ✅ 14-day offline grace period · 24h priority support

### Sovereign (from $2,500/mo)

- ✅ Everything in Team — for regulated industries
- ✅ Fully offline activation / air-gap deployment
- ✅ SSO (SAML/OIDC) · custom compliance mapping

[View full pricing →](https://www.replimap.com/#pricing)

---

## 🔒 Security & Privacy

**Your data never leaves your machine.**

- ✅ RepliMap runs entirely client-side — no SaaS cloud to trust
- ✅ Read-only AWS access (no modifications, ever)
- ✅ Sensitive data (passwords, keys) automatically redacted from output
- ✅ Generated reports and graphs are self-contained files — zero outbound requests
- ✅ License entitlements verified locally (Ed25519 signatures) — no phone-home in the scan path

Run `replimap --privacy` for the full data-handling statement.

---

## 🐛 Issues & Feature Requests

Found a bug? Have a feature request?

- [🐛 Report a Bug](https://github.com/RepliMap/replimap-community/issues/new?template=bug_report.md)
- [💡 Request a Feature](https://github.com/RepliMap/replimap-community/issues/new?template=feature_request.md)

When a command fails, RepliMap prints an error reference (e.g.
`ERR-EC2-403-A7X9`) and saves a local log — run
`replimap explain ERR-EC2-403-A7X9` and paste the summary into your issue.

We read every issue. Your feedback shapes the roadmap.

---

## Documentation

- [Full Documentation](https://replimap.com/docs)
- [IAM Policy](IAM_POLICY.md)
- [Changelog](CHANGELOG.md)
- [Pricing](https://www.replimap.com/#pricing)

---

## Support & Contact

| Purpose | Contact |
|---------|---------|
| General inquiries | [hello@replimap.com](mailto:hello@replimap.com) |
| Technical support | [support@replimap.com](mailto:support@replimap.com) |
| Enterprise & Sales | [david@replimap.com](mailto:david@replimap.com) |
| Bug reports | [GitHub Issues](https://github.com/RepliMap/replimap-community/issues) |

## Links

- Website: [replimap.com](https://replimap.com)
- Documentation: [replimap.com/docs](https://replimap.com/docs)
- Pricing: [replimap.com/#pricing](https://www.replimap.com/#pricing)
- Twitter: [@replimap_io](https://twitter.com/replimap_io)

---

## 📄 License

RepliMap is licensed under the [Business Source License 1.1](LICENSE).

---

<p align="center">
  <strong>From chaos to clarity. From ClickOps to GitOps.</strong>
</p>

<p align="center">
  Made with ☕ in New Zealand 🇳🇿
</p>
