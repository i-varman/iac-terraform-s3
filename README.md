# iac-terraform-s3
Defined an Amazon S3 bucket and an uploaded object in Terraform, then provisioned both from the command line using the init, plan, and apply workflow with credentials configured through the AWS CLI.
# Provision an S3 Bucket on AWS with Terraform (Infrastructure as Code)

Defined an Amazon S3 bucket and an uploaded object in Terraform, then provisioned both from the command line using the `init`, `plan`, and `apply` workflow with credentials configured through the AWS CLI.

![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC)
![AWS](https://img.shields.io/badge/AWS-S3%20%7C%20CLI-orange)

> Guided project built with [NextWork](https://nextwork.ai). Project link: [View Project](https://nextwork.ai/projects/10d3e52a-a2ad-597e-9e1c-33526bde77ba)

---

## Overview

| Item | Detail |
|---|---|
| **Goal** | Create and manage AWS infrastructure as code instead of clicking in the console |
| **Tools** | Terraform, AWS CLI, Amazon S3 |
| **Resources created** | One S3 bucket and one uploaded S3 object |
| **Hardest part** | Configuring the correct AWS credentials |

## Workflow

```
main.tf  -->  terraform init  -->  terraform plan  -->  terraform apply
(desired       (download          (preview changes)     (create resources
 state)         providers)                               in AWS)
                                                              |
                                                              v
                                                    S3 bucket + object
                                                    (verified in AWS console)
```

## Key Concepts

- **Infrastructure as Code (IaC):** Describing cloud infrastructure in files so it is repeatable, version-controlled, and reviewable.
- **Terraform:** A multi-cloud IaC tool (AWS, Azure, Google Cloud) that uses a readable, declarative language (HCL). You describe the desired state, and Terraform works out how to reach it.
- **`main.tf`:** The central configuration file that defines the infrastructure.
- **Provider block:** Tells Terraform which cloud platform and region to work with.
- **Resource block:** Describes what to create (a bucket, an object, a VM) and how it should be configured.
- **State:** Terraform tracks what it has created in a state file so it can plan accurate changes later.

## Prerequisites

- AWS account
- Terraform installed (`terraform -version`)
- AWS CLI installed and configured (`aws configure`)
- A file to upload to S3 (for example an image)

## Configuration

`main.tf` is a template. Replace the values with the ones from your own file.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "<AWS_REGION>"
}

resource "aws_s3_bucket" "demo" {
  bucket = "<GLOBALLY_UNIQUE_BUCKET_NAME>"
}

# Added after the bucket was created: upload an object
resource "aws_s3_object" "image" {
  bucket = aws_s3_bucket.demo.id
  key    = "<OBJECT_NAME>.png"
  source = "<LOCAL_FILE_PATH>"
}
```

> S3 bucket names must be globally unique across all AWS accounts.

## Steps

### 1. Initialize

```bash
terraform init
```

Prepares the working directory by:
- Downloading the required provider plugins
- Setting up the backend
- Preparing modules
- Creating a dependency lock file (`.terraform.lock.hcl`)

### 2. Format and validate

```bash
terraform fmt
terraform validate
```

### 3. Plan

```bash
terraform plan
```

Shows what Terraform will create, change, or destroy, so changes can be reviewed before anything is touched in AWS.

### 4. Apply

```bash
terraform apply
```

Creates the S3 bucket in the AWS account. Review the plan output and confirm with `yes`.

### 5. Upload an object

I added a new `aws_s3_object` resource block and ran `terraform apply` again. Terraform detected only the new resource and added it, leaving the existing bucket unchanged. I confirmed the file in the S3 console.

## Troubleshooting: "No valid credential sources found"

**Problem:** `terraform plan` failed because Terraform had no AWS credentials to authenticate with.

**Fix:**
1. Installed the AWS CLI.
2. Created access keys for an IAM user.
3. Ran `aws configure` to store the credentials locally.
4. Re-ran `terraform plan` successfully.

```bash
aws configure
aws sts get-caller-identity   # confirms which identity Terraform will use
```

| Other common issue | Cause | Fix |
|---|---|---|
| `BucketAlreadyExists` | Bucket name is taken globally | Choose a more unique name |
| `AccessDenied` | IAM user lacks S3 permissions | Attach the needed S3 permissions to the user |
| Wrong region errors | Region mismatch between provider and CLI | Align `provider "aws"` region with your intent |

## Security Notes

- **Never commit** `terraform.tfstate`, `.terraform/`, `*.tfvars` with secrets, or AWS access keys. Use the included `.gitignore`.
- Commit `.terraform.lock.hcl` so provider versions stay consistent.
- Long-lived access keys are fine for a learning lab, but delete or deactivate them afterwards. For real work, prefer IAM roles or AWS IAM Identity Center (SSO) and a remote backend.

## Screenshots

Save your screenshots in an `images/` folder and reference them like this:

```markdown
![terraform init output](images/terraform-init.png)
![terraform plan output](images/terraform-plan.png)
![terraform apply output](images/terraform-apply.png)
![Object in S3 console](images/s3-object.png)
```

## Cleanup

Remove everything Terraform created to avoid leftover resources:

```bash
terraform destroy
```

## What I Learned

- How Terraform turns a configuration file into real AWS resources
- Why the `init`, `plan`, `apply` sequence matters: set up providers, review changes, then execute
- How provider and resource blocks are structured
- How to resolve credential errors by configuring the AWS CLI
- How Terraform applies incremental changes when the configuration is updated


## Author

**Imayavarman Senthilkumar**
Email: i.varman.work@gmail.com

---

*Built with [NextWork](https://nextwork.ai)*
