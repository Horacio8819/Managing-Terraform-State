# Day 6: Managing Terraform State - Best Practices for DevOps

Part of my 30 Day Terraform Challenge journey with AWS AI/ML UserGroup Kenya, Meru HashiCorp User Group, and EveOps.

---

## Overview

This project demonstrates using Terraform to provision AWS resources with remote state management in S3 and state locking. 
It includes best practices for state security, versioning, and team collaboration.
cover what state is, why local state fails at scale, and how to set up S3 + DynamoDB correctly.


## Challenge 

Migration of Terraform state from local to remote using AWS S3 for storage and DynamoDB for state locking, including testing the lock

### Create S3 Bucket and DynamoDB Table

#### S3 Bucket #### S3 Buckez
set versioning_configuration to enable ---> Versioning: Keeps historical versions of your state.
apply_server_side_encryption_by_default ---> Server-side encryption: Secures sensitive Terraform outputs in S3.
set prevent_destroy = true ---> Prevent destroy: Protects the bucket from accidental deletion.

#### DynamoDB Table for State Locking
DynamoDB is used for state locking, preventing concurrent modifications.
DynamoDB Table → LockID as hash key, PAY_PER_REQUEST billing

### Configure Terraform Backend
In your Terraform configuration main.tf, add:

                                                                  terraform {
                                                                    backend "s3" {
                                                                      bucket         = "your-unique-terraform-state-bucket"
                                                                      key            = "global/s3/terraform.tfstate"  # the path to your state file inside the bucket.
                                                                      region         = "us-east-1"
                                                                      dynamodb_table = "terraform-state-locks"   # the table you just created for locks
                                                                      encrypt        = true    # ensures the state file is encrypted at rest.
                                                                    }
                                                                  }

### Migrate Local State to Remote
- run terraform init ---> validate with yes to to copy your existing state to the new backend
- Open two terminal windows pointing to the same Terraform configuration. In the first, run terraform apply. While it is running, attempt terraform plan in the second
- Lock error: This prevents multiple people (or CI/CD pipelines) from simultaneously modifying the state, which could corrupt it.
---

## Key Concepts Covered

Terraform keeps track of your infrastructure in a **state file**, which maps your configuration to real-world resources.
Local state works fine for personal projects, but it fails at scale in team environments:

- **No collaboration:** Multiple people modifying state locally can corrupt the file.
- **No history:** Losing a local state file can break your environment.
- **No locking:** Concurrent operations may overwrite changes.

The state file contains sensitive information such as resource IDs, IP addresses, and potentially secrets in plaintext. Committing it to Git exposes this data and creates security risks. Additionally, the file is frequently updated, which can lead to merge conflicts when multiple engineers run Terraform operations concurrently.

The bootstrap problem refers to the challenge of setting up the infrastructure required to manage Terraform state. You cannot initially rely on Terraform to create the S3 bucket or locking mechanism it needs for remote state. These resources must be provisioned first (often manually or via a separate configuration), after which the backend is configured and terraform init  is used to migrate the state.

State locking prevents multiple users from running Terraform operations (such as apply) at the same time on the same state file. This avoids race conditions, state corruption, and inconsistent infrastructure changes. S3 versioning protects against accidental deletion or corruption of the Terraform state file. Each update creates a new version, allowing you to restore a previous known-good state if something goes wrong, ensuring greater resilience and recoverability.

---

### Author
Horace Djousse Fofe;

#30DayTerraformChallenge #AWSUserGroupKenya #EveOps
