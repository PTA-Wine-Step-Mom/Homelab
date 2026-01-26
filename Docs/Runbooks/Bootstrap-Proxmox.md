# Runbook: Bootstrap Proxmox for Terraform

Goal:
Enable Terraform-managed VM lifecycle via Proxmox API with minimal setup.

Prerequisites:
- Proxmox VE accessible and admin account
- Network and storage pools identified

Steps (outline):
1. Create API token/user with limited scope for Terraform
2. Verify storage (local-lvm/NFS) and ISO/template availability
3. Confirm network bridges/VLANs to use for VMs
4. Record endpoint, token, and project-specific config in local tfvars (do not commit)

Next:
- Proceed to Docs/Runbooks/Provision-VMs.md
