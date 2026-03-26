# Day 6: Managing Terraform State - Best Practices for DevOps

Part of my 30 Day Terraform Challenge journey with AWS AI/ML UserGroup Kenya, Meru HashiCorp User Group, and EveOps.
---

## Overview

This project demonstrates using Terraform to provision AWS resources with remote state management in S3 and state locking. 
It includes best practices for state security, versioning, and team collaboration.
cover what state is, why local state fails at scale, and how to set up S3 + DynamoDB correctly.
---

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
                                                              key            = "global/s3/terraform.tfstate"  #is the path to your state file inside the bucket.
                                                              region         = "us-east-1"
                                                              dynamodb_table = "terraform-state-locks"
                                                              encrypt        = true    #ensures the state file is encrypted at rest.
                                                            }
                                                          }

