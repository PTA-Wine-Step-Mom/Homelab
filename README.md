# Remote Desktop Platform on Proxmox (Learning/Demo)

A Linux-based remote desktop platform inspired by Azure RDS, implemented using Proxmox VE, Terraform (VM lifecycle), Ansible (configuration), and Linux-native services (XRDP, Guacamole, NFS, HAProxy).

### Architecture at a Glance
- Proxmox VE hypervisor hosts VMs
- Terraform manages VM lifecycle and basic networking/storage
- Ansible configures base OS and services (XRDP, Guacamole, NFS client, HAProxy)
- HAProxy terminates TLS and routes to Guacamole; sessions reach XRDP hosts; NFS provides shared storage

See: Docs/Architecture/Platform.md and Docs/Architecture/Networking.md

### Tech Stack
- Proxmox VE
- Terraform
- Ansible
- XRDP, Guacamole
- NFS, HAProxy

### Quickstart
- Bootstrap Proxmox: Docs/Runbooks/Bootstrap-Proxmox.md
- Provision VMs: Docs/Runbooks/Provision-VMs.md
- Configure Services: Docs/Runbooks/Configure-Services.md
- Demo Scenarios: Docs/Runbooks/Demo-Scenarios.md

### Repository Layout
- infra/: Terraform and Ansible (modules, inventories, roles, playbooks)
- services/: Service assets and configs (xrdp, guacamole, haproxy, nfs)
- Docs/: Overview, Architecture, Decisions (ADRs), Runbooks, Diagrams, Build-Log
- scripts/: Helper scripts (bootstrap, validate, demo)

### Milestones & Branching
TBD

### Scope & Non-goals
- Demo only; not a production SaaS
- Secrets stored locally via tfvars/Ansible vault; never committed

### About This Homelab
This repository also documents broader homelab topics (Networking, Automation, Server Hosting).
See Docs/Wiki.md for general homelab information and a network layout diagram.
