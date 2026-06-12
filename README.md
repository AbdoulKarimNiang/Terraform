# Terraform Exercises — AWS

Hands-on Terraform exercises built while preparing for the HashiCorp Terraform
Associate (003) exam. Everything targets AWS in `us-east-1` and is written to stay
inside the free tier wherever possible. Each exercise is self-contained and is torn
down with `terraform destroy` once it's done.

## Goals

- Build real infrastructure rather than learning the workflow in the abstract.
- Keep a consistent, predictable layout so any exercise can be picked up cold.
- Stay at or near zero cost — no idle NAT gateways, EIPs, load balancers, or
  hosted zones left running.
- Use one naming and tagging standard across every exercise (see below).

## Prerequisites

| Tool | Version |
|------|---------|
| Terraform CLI | `>= 1.6` |
| AWS provider | `~> 5.0` |
| AWS CLI | v2, configured with a profile that can deploy to `us-east-1` |

You'll also need an AWS account. Accounts created before 15 July 2025 use the legacy
12-month free tier; newer accounts use the credit-based model — worth knowing before
you spin anything up.

## Repository structure

```
.
├── 01-basics/            # init, plan, apply, destroy; random provider
├── 02-variables/         # variables, outputs, locals, meta-arguments
├── 03-networking/        # VPC, subnets, route tables, security groups
├── 04-compute/           # EC2, launch templates, auto scaling
├── 05-iam/               # roles, policies, instance profiles
├── 06-modules/           # reusable modules
├── 07-remote-state/      # S3 backend, state locking, workspaces
├── 08-serverless/        # Lambda, API Gateway, EventBridge
└── README.md
```

## Getting started

```bash
cd 03-networking
terraform init
terraform plan
terraform apply
# ... inspect, experiment ...
terraform destroy
```

If `init` starts misbehaving after a provider typo or a stale lock file, the reliable
reset is to delete `.terraform/` and `.terraform.lock.hcl` and run `terraform init`
again, or `terraform init -upgrade` to re-resolve providers.

## Naming convention

The aim is simple: reading a resource name alone should tell you **which service it
is** and **which project and environment it belongs to**, and any teammate should be
able to reconstruct a name from a resource's purpose without guessing.

### Pattern

```
<project>-<environment>-<resource>-<descriptor>[-<suffix>]
```

| Segment | Meaning | Source | Example |
|---------|---------|--------|---------|
| `project` | Short project identifier, fixed per repo | agreed once | `tfassoc` |
| `environment` | Deployment stage | closed set: `dev`, `staging`, `prod` | `dev` |
| `resource` | Service / resource type | acronym table below | `sg` |
| `descriptor` | Which instance of that type | descriptor vocabulary | `web` |
| `suffix` | Optional uniqueness suffix | `random_id.hex` etc. | `3f9a1c` |

### Rules

- **kebab-case, lowercase only.** It's the lowest common denominator: S3 accepts it,
  and so does almost everything else. Hyphen is the only separator — never mix in
  underscores or camelCase.
- **Every segment comes from a closed set, never free text.** That's what turns a
  naming *style* into a naming *standard*. `logs` never becomes `logging` or
  `log-bucket` depending on who typed it.
- **Suffix only when uniqueness demands it** — S3 buckets and other globally- or
  account-unique resources. Don't add noise to names that are already unique.
- **No secrets, account IDs, or sensitive data in names.** They leak into logs,
  ARNs, and the console.
- **Mind length limits.** S3 buckets cap at 63 characters; IAM roles and Lambda
  functions at 64. A long project name plus a suffix can overflow.

### Resource acronyms

Abbreviations are anchored to AWS's own resource-ID prefixes where they exist
(`vpc-`, `sg-`, `igw-`, `rtb-`, `pcx-`…), since those are already familiar from the
console. Where AWS has no short prefix, the common service acronym is used instead.

| Acronym | AWS Resource | Category |
|---------|--------------|----------|
| `vpc` | VPC | Networking |
| `subnet` | Subnet | Networking |
| `igw` | Internet Gateway | Networking |
| `natgw` | NAT Gateway | Networking |
| `rtb` | Route Table | Networking |
| `sg` | Security Group | Networking |
| `nacl` | Network ACL | Networking |
| `eip` | Elastic IP | Networking |
| `eni` | Elastic Network Interface | Networking |
| `vpce` | VPC Endpoint | Networking |
| `pcx` | VPC Peering Connection | Networking |
| `tgw` | Transit Gateway | Networking |
| `vgw` | Virtual Private Gateway | Networking |
| `cgw` | Customer Gateway | Networking |
| `vpn` | Site-to-Site VPN Connection | Networking |
| `dx` | Direct Connect | Networking |
| `r53` | Route 53 Hosted Zone | Networking |
| `cf` | CloudFront Distribution | Networking |
| `alb` | Application Load Balancer | Networking |
| `nlb` | Network Load Balancer | Networking |
| `clb` | Classic Load Balancer | Networking |
| `gwlb` | Gateway Load Balancer | Networking |
| `tg` | Target Group | Networking |
| `ga` | Global Accelerator | Networking |
| `ec2` | EC2 Instance | Compute |
| `ami` | Amazon Machine Image | Compute |
| `asg` | Auto Scaling Group | Compute |
| `lt` | Launch Template | Compute |
| `lc` | Launch Configuration | Compute |
| `lambda` | Lambda Function | Compute |
| `ecs` | ECS Cluster | Compute |
| `ecssvc` | ECS Service | Compute |
| `ecstask` | ECS Task Definition | Compute |
| `eks` | EKS Cluster | Compute |
| `eksng` | EKS Node Group | Compute |
| `ecr` | Elastic Container Registry | Compute |
| `eb` | Elastic Beanstalk Environment | Compute |
| `batch` | Batch Compute Environment | Compute |
| `s3` | S3 Bucket | Storage |
| `ebs` | EBS Volume | Storage |
| `efs` | Elastic File System | Storage |
| `fsx` | FSx File System | Storage |
| `snap` | EBS Snapshot | Storage |
| `backup` | Backup Vault | Storage |
| `sgw` | Storage Gateway | Storage |
| `datasync` | DataSync Task | Storage |
| `rds` | RDS Instance | Database |
| `aurora` | Aurora Cluster | Database |
| `ddb` | DynamoDB Table | Database |
| `ec` | ElastiCache Cluster | Database |
| `redshift` | Redshift Cluster | Database |
| `dms` | Database Migration Service | Database |
| `rdsproxy` | RDS Proxy | Database |
| `iamrole` | IAM Role | Security |
| `iampolicy` | IAM Policy | Security |
| `iamuser` | IAM User | Security |
| `iamgroup` | IAM Group | Security |
| `idp` | IAM Identity Provider | Security |
| `kms` | KMS Key | Security |
| `sm` | Secrets Manager Secret | Security |
| `ssmparam` | SSM Parameter | Security |
| `acm` | ACM Certificate | Security |
| `waf` | WAF Web ACL | Security |
| `shield` | Shield Protection | Security |
| `cognito` | Cognito User Pool | Security |
| `gd` | GuardDuty Detector | Security |
| `inspector` | Inspector Assessment | Security |
| `sh` | Security Hub | Security |
| `ram` | Resource Access Manager Share | Security |
| `sqs` | SQS Queue | Integration |
| `sns` | SNS Topic | Integration |
| `ebus` | EventBridge Bus | Integration |
| `ebrule` | EventBridge Rule | Integration |
| `sfn` | Step Functions State Machine | Integration |
| `kds` | Kinesis Data Stream | Integration |
| `kfh` | Kinesis Firehose | Integration |
| `mq` | Amazon MQ Broker | Integration |
| `apigw` | API Gateway (REST) | Integration |
| `apigwv2` | API Gateway (HTTP / WebSocket) | Integration |
| `appsync` | AppSync GraphQL API | Integration |
| `cfn` | CloudFormation Stack | Management |
| `cwmetric` | CloudWatch Dashboard | Management |
| `cwalarm` | CloudWatch Alarm | Management |
| `cwlogs` | CloudWatch Log Group | Management |
| `ct` | CloudTrail Trail | Management |
| `config` | Config Rule | Management |
| `ssmdoc` | Systems Manager Document | Management |
| `org` | Organizations OU / Account | Management |
| `scp` | Service Control Policy | Management |
| `xray` | X-Ray | Management |
| `athena` | Athena Workgroup | Analytics |
| `glue` | Glue Job | Analytics |
| `emr` | EMR Cluster | Analytics |
| `msk` | Managed Kafka Cluster | Analytics |
| `opensearch` | OpenSearch Domain | Analytics |
| `quicksight` | QuickSight | Analytics |
| `cbuild` | CodeBuild Project | Dev Tools |
| `cpipe` | CodePipeline Pipeline | Dev Tools |
| `ccommit` | CodeCommit Repository | Dev Tools |
| `cdeploy` | CodeDeploy Application | Dev Tools |

### Descriptor vocabulary

The descriptor disambiguates *which* instance of a type. Keep it to a closed list and
extend deliberately:

- **Network tier:** `public`, `private`, `isolated` (add an AZ marker when split by
  zone: `public-a`, `private-b`)
- **Application tier:** `web`, `app`, `api`, `worker`, `cache`, `db`, `bastion`
- **Buckets:** `artifacts`, `assets`, `static`, `media`, `uploads`, `logs`,
  `backups`, `tfstate`, `data`, `exports`
- **IAM:** `lambda-exec`, `ec2-instance`, `ecs-task`, `ci-deploy`, `readonly`,
  `admin`, `cross-account`
- **Singletons** (one per VPC): `main` or `primary`
- **Pairs / replicas:** `primary`/`replica`, `blue`/`green`, or ordinals `1`/`2`/`3`

> Extend as needed — when you add a descriptor, add it here too so it stays a single
> source of truth.

### Examples

| Resource name | Reads as |
|---------------|----------|
| `tfassoc-dev-vpc-main` | The project's main VPC in dev |
| `tfassoc-dev-subnet-public-a` | Public subnet in AZ `a` |
| `tfassoc-dev-subnet-private-b` | Private subnet in AZ `b` |
| `tfassoc-dev-igw-main` | Internet gateway for the dev VPC |
| `tfassoc-dev-sg-web` | Security group for the web tier |
| `tfassoc-dev-sg-db` | Security group for the database tier |
| `tfassoc-dev-s3-artifacts-3f9a1c` | Artifacts bucket, suffix for global uniqueness |
| `tfassoc-dev-iamrole-lambda-exec` | Execution role for a Lambda function |
| `tfassoc-dev-lambda-image-processor` | Lambda function that processes images |
| `tfassoc-prod-rds-orders` | Production RDS instance backing the orders service |

## Tagging convention

Names identify a resource; **tags carry the metadata** — and they're what makes cost
allocation, filtering, and automation possible. Build the prefix and tags once in
`locals` and compose, so a promotion from `dev` to `prod` is a one-line change.

```hcl
locals {
  project     = "tfassoc"
  environment = "dev"
  name_prefix = "${local.project}-${local.environment}"

  common_tags = {
    Project     = local.project
    Environment = local.environment
    ManagedBy   = "terraform"
  }
}

resource "aws_security_group" "web" {
  name        = "${local.name_prefix}-sg-web"
  description = "Web tier ingress"
  vpc_id      = aws_vpc.main.id

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-sg-web"
    Role = "web"
  })
}
```

`ManagedBy = "terraform"` signals to anyone in the console that the resource is IaC
and shouldn't be hand-edited.

## Cost awareness & cleanup

Run `terraform destroy` at the end of every exercise. The resources that quietly cost
money even when idle, and are easy to forget:

- NAT Gateways and the public IPv4 addresses attached to them
- Idle Elastic IPs (charged when allocated but not associated)
- ALB / NLB / Gateway Load Balancers
- Route 53 hosted zones
- Secrets Manager secrets and customer-managed KMS keys

A billing alarm in CloudWatch plus an occasional look at Cost Explorer is enough to
catch anything that slips through.
