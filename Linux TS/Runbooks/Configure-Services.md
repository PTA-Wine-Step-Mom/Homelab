# Runbook: Configure Services with Ansible

Goal:
Install and configure base OS + services (XRDP, Guacamole, NFS client, HAProxy).

Prerequisites:
- VM IPs/hosts recorded from Terraform outputs
- SSH access verified

Steps (outline):
1. Prepare `infra/ansible/inventory` with groups (gateway, session-hosts)
2. Run base role on all hosts
3. Apply service roles per host group
4. Verify services and basic connectivity

Next:
- Proceed to Docs/Runbooks/Demo-Scenarios.md
