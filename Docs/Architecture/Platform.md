# Platform Architecture Overview

Components:
- Proxmox VE: hypervisor hosting VMs
- Terraform: VM lifecycle and basic network/storage provisioning
- Ansible: OS configuration and service installation
- XRDP: remote desktop service on Linux hosts
- Guacamole: web gateway for remote access
- NFS: shared storage for user profiles or data
- HAProxy: edge routing/TLS termination to gateway

Responsibilities:
- Terraform: create/update/destroy VM instances and base networking
- Ansible: configure base OS and services (XRDP, Guacamole, NFS client, HAProxy)
- Services: provide remote desktop access and gateway

Data/Control Flows (high level):
- Users connect via HAProxy → Guacamole → XRDP on target VM
- NFS provides shared mounts to session hosts
- Admins use Terraform/Ansible from control workstation
