# Recommended Terraform folder structure

A well-organized folder structure is critical for managing a mid-size Terraform project, especially as the number of modules, environments, and team members grows.

Here’s a recommended folder structure that balances modularity, reusability, and environment separation:

```
terraform-project/
├── modules/                    # Reusable modules
│   ├── network/
│   │   └── main.tf
│   │   └── variables.tf
│   │   └── outputs.tf
│   ├── compute/
│   │   └── main.tf
│   │   └── variables.tf
│   │   └── outputs.tf
│   └── database/
│       └── main.tf
│       └── variables.tf
│       └── outputs.tf
│
├── environments/              # Environment-specific configurations
│   ├── dev/
│   │   └── main.tf
│   │   └── backend.tf
│   │   └── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   └── main.tf
│   │   └── backend.tf
│   │   └── variables.tf
│   │   └── terraform.tfvars
│   └── prod/
│       └── main.tf
│       └── backend.tf
│       └── variables.tf
│       └── terraform.tfvars
│
├── global/                    # Global/shared resources (e.g., IAM, org-level policies)
│   └── main.tf
│   └── variables.tf
│   └── outputs.tf
│
├── scripts/                   # Helper scripts (e.g., CI/CD, bootstrap, plan/apply wrappers)
│   └── init.sh
│   └── plan.sh
│   └── apply.sh
│
├── .terraform.lock.hcl        # Terraform dependency lock file
├── versions.tf                # Required providers and Terraform version
├── README.md
└── Makefile                   # Optional: helpful Terraform CLI shortcuts

```

Notes:
- modules/: Custom modules that encapsulate specific infrastructure (VPCs, subnets, EC2s, etc.). Each module should have its own input/output files.
- environments/: Separate directories for dev, staging, prod, each with their own state and variable configurations.
- global/: Resources shared across environments, like IAM roles or logging sinks.
- scripts/: Useful for CI pipelines or repeatable CLI operations.
- backend.tf: Typically includes remote backend config (e.g., S3, Azure Storage, GCS).
- Makefile (optional): Simplifies frequent tasks like terraform plan, terraform apply, etc.

## How to Use global/ or Other Subdirectories
If you want to use the resources defined in global/, you need to:

- Make global/ a module, and then
- Call it from a root module like so:

```
# main.tf (in root or environment directory)
module "global" {
  source = "../global"
  # pass any required variables
}
```