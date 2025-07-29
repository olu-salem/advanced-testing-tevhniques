# Bulk Tagging Strategy Using AWS Config and Tag Mapping

## 1. Set Up AWS Config Rule: Required Tags

### Step 1.1: Enable the `required-tags` AWS Config Managed Rule

- Use the rule `required-tags` to check all AWS resources for the presence of required tags.
- Avoid long tag key names like `Name`. Use a shorter version such as `nametag`.
- Ensure AWS Config records all resource types.

### Step 1.2: Define Required Tags

Use keys like:

- `Owner`
- `Environment`
- `Application`
- `Project`
- `ManagedBy`

---

## 2. Define Your Tagging Strategy and Governance

### Step 2.1: Tag Categories

| Tag Key | Description |
| --- | --- |
| `Owner` | Person or team responsible |
| `Environment` | dev, staging, prod, etc. |
| `Application` | System or component name |
| `Project` | Business or initiative name |
| `ManagedBy` | Terraform, CloudFormation, Manual |

### Step 2.2: Tag Policies (Optional)

- Use SCPs to prevent resource creation without tags.
- Enforce allowed values (e.g., `Environment` must be one of `dev`, `staging`, `prod`).

---

## 3. Prepare for Bulk Tagging

### Step 3.1: Export Resources

- Go to **AWS Resource Groups > Tag Editor**
- Filter by region/resource types
- Export as `resources.csv`

### Step 3.2: Create Tag Mapping File

- Create `tag-mapping-wildcards-expanded.csv`
- Define tagging rules using wildcard patterns:

| NamePattern | Environment | Application | Owner | Project |
| --- | --- | --- | --- | --- |
| `*dev*` | dev |  |  |  |
| `*prod*` | production |  |  |  |
| `*staging*` | staging |  |  |  |
| `*mubarak*` |  |  | mubarak |  |
| `*demo-infra*` |  |  |  | demo-infra |
| `*grafana*` | staging | monitoring | ops |  |
| `*momentum*` | dev | analytics | data-team |  |
| `*glue*` | dev | data-etl | data-team |  |
| `*lambda*` | dev | serverless | devops |  |
| `*cloudtrail*` | test | audit | secops |  |
| `*sagemaker*` | test | ml | ml-team |  |
| `*sg-*` | dev | security | network-team |  |
| `*eks*` | dev | eks-cluster | devops |  |
| `*subnet-*` | dev | networking | infra-team |  |
| `*codebuild*` | dev | ci | devops |  |
| `*retail-store*` | dev | ecs-app | web-team |  |

---

## 4. Bulk Tagging via Script

### Step 4.1: Use Region-Aware Tagging Script

- Use `enhanced_bulk_tagging.py`
- Reads `resources.csv` and `tag-mapping-wildcards-expanded.csv`
- Supports dry run and logs actions to CSV

### Key Features:

- Extracts region from each ARN
- Uses correct `boto3` client per resource/region
- Merges new tags with existing ones (does not delete others)
- Writes output to `tagging-log-*.csv`

### Example Usage:

```bash
# Dry run
python enhanced_bulk_tagging.py --dry-run

# Actual tagging
python enhanced_bulk_tagging.py

```

---

## 5. Remediation for Future Resources

### Step 5.1: Automate Enforcement with AWS Config

Use `required-tags` rule + remediation:

### A. AWS Lambda (Recommended)

- Triggered on non-compliance
- Auto-tags using business logic
- Uses `boto3` to apply tags

### B. SSM Automation (Fallback)

- Applies static tags
- Useful when default tagging is sufficient