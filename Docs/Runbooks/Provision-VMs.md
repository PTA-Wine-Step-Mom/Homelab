# Runbook: Provision VMs with Terraform

Goal:
Create session hosts and gateway VMs using Terraform.

Prerequisites:
- Completed Proxmox bootstrap and local tfvars

Steps (outline):
1. Initialize Terraform in infra/terraform
2. Set environment tfvars (dev/demo) and review planned changes
3. Apply changes to create/update VMs
4. Record outputs (IPs/IDs) for Ansible inventory

Next:
- Proceed to Docs/Runbooks/Configure-Services.md
