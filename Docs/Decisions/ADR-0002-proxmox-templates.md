# ADR-0002: Proxmox VM Templates and Cloud-Init Strategy

## Context

The platform requires reusable, non-interactive VM templates to support Terraform-driven infrastructure provisioning. Templates must support cloud-init for per-instance customization (hostnames, SSH keys, users) and enable rapid, consistent VM cloning across the Proxmox cluster.

## Decision

### Base OS and Version
**Choice:** Ubuntu Server 24.04.03 LTS (Noble Numbat)

**Rationale:**
- LTS release ensures 5-year support window (April 2029)
- Mature ecosystem; widely used in RDS and cloud infrastructure
- cloud-init first-class support with predictable behavior
- Community and recruitment appeal for demo/learning context

### Desktop Environment
**Choice:** XFCE 4.18 (for session-host template)

**Rationale:**
- Lightweight; performs well over RDP/Guacamole with higher session density
- Lower resource overhead than GNOME (CPU, memory, disk)
- Faster login/logout cycles for demo scenarios
- GNOME deferred to M08 (polish) if demo usability requires it

### Template Strategy
**One base template + role-specific cloning**

**Rationale:**
- Single `rds-base-u2404-s` template contains minimal, reusable OS configuration
- `rds-desktop-u2404-d` cloned from base; adds XFCE and display libraries only
- Service installation (XRDP, Guacamole, etc.) deferred to Ansible roles post-cloning
- Reduces template build/maintenance burden; Ansible becomes source of truth for service config

### Cloud-Init Approach
**Template includes minimal config; Terraform passes user_data per-instance**

**Rationale:**
- Template sets only: timezone, hostname placeholder, enable SSH, disable password auth
- Terraform/cloud-init handles: per-VM hostnames, SSH public keys, custom packages, network config
- Enables non-interactive, deterministic provisioning from Terraform definitions
- Easy to version control Terraform user_data blocks; harder to version templates

### Storage and Naming
**Local Proxmox storage (local-lvm); templates stored as VMs with `tpl-` prefix**

**Rationale:**
- Simplicity for single/dual-host Proxmox clusters
- Naming convention `tpl-rds-<purpose>-<osversion>` avoids accidental cloning
- Future migration to shared NFS storage possible without API changes

### QEMU Guest Agent
**Required in all templates**

**Rationale:**
- Enables Proxmox to query VM status, IP addresses, and system info
- Supports clean shutdown and memory ballooning
- Required by Terraform provisioner timeouts and health checks

## Consequences

**Positive:**
- Simple, maintainable template lifecycle
- Terraform becomes single source of truth for instance config
- Easy to iterate: modify Terraform user_data without template rebuild
- XFCE choice reduces RDP resource contention in demo scenarios

**Negative:**
- XFCE may feel less polished in recruitment demos; GNOME deferral noted for M08
- Cloud-init errors in Terraform user_data harder to debug than template issues
- No pre-installed application tooling; all service setup via Ansible (longer first-run)

## Status
**ACCEPTED** — Ready for implementation in feat/proxmox-templates branch.

## Next Steps
- Build templates per Docs/Runbooks/Build-Proxmox-Templates.md
- Integrate templates into Terraform VM module (M02: feat/m02-terraform-vm-lifecycle)
- Consider Packer-based template automation for M02b (out of scope for this branch)
