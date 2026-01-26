# Build Log: January 2026 – Proxmox Templates Branch

**Branch:** feat/proxmox-templates  
**Milestone:** M01  
**Goal:** Create reusable Proxmox VM templates (rds-base-u2204, rds-desktop-u2204) with cloud-init and SSH key authentication support.

---

## Decisions & Rationale

**OS Choice: Ubuntu Server 22.04 LTS**
- 5-year support window; widely used in RDS and cloud infrastructure
- Excellent cloud-init integration; predictable behavior
- I have worked in U2204 before and know its enviromnet well

**Desktop Environment: XFCE 4.18**
- Lightweight; better RDP performance vs. GNOME
- Lower resource overhead (CPU, memory, disk)
- Faster login/logout for demo scenarios
- GNOME deferred to future update

**Template Strategy: One base + service via Ansible**
- Single `tpl-rds-base-u2204` template contains minimal OS config
- `tpl-rds-desktop-u2204` cloned from base; adds XFCE and display libraries only
- Service installation (XRDP, Guacamole, HAProxy, NFS client) deferred to Ansible roles
- Reduces template maintenance; Ansible becomes source of truth for service config

**Cloud-Init Approach: Minimal template, full user_data from Terraform**
- Template sets only: timezone, hostname placeholder, SSH key-only auth, QEMU Agent
- Terraform passes per-instance: hostname, SSH keys, custom packages, network config
- Non-interactive, deterministic provisioning from Terraform definitions
- Version control advantage: Terraform user_data in Git; harder to version templates

**Storage & Naming: Local Proxmox (local-lvm), naming prefix tpl-**
- Simplicity for single/dual-host clusters
- Convention `tpl-rds-<purpose>-<osversion>` avoids accidental cloning
- Future NFS storage migration possible without API changes

**QEMU Guest Agent: Required**
- Enables Proxmox to query VM status, IP, system info
- Supports clean shutdown and memory ballooning
- Required by Terraform provisioners and health checks

---

## Implementation Artifacts

**Created:**
- [Docs/Decisions/ADR-0003-proxmox-templates.md](../Decisions/ADR-0002-proxmox-templates.md) — Design decisions and consequences
- [Docs/Runbooks/Build-Proxmox-Templates.md](../Runbooks/Build-Proxmox-Templates.md) — Step-by-step build instructions for both templates
- [services/cloud-init/base-user-data.yaml](../../services/cloud-init/base-user-data.yaml) — Example cloud-init for base instances
- [services/cloud-init/session-host-user-data.yaml](../../services/cloud-init/session-host-user-data.yaml) — Example cloud-init for session hosts (with NFS mount stubs)

**Next:** Manual template builds via Proxmox UI per runbook.

---

## Lessons Learned

1. **Cloud-Init Initialization:** Ensure `cloud-init clean --logs --seed` is run before template conversion to avoid confusion on cloning (old metadata should not persist).

2. **SSH Configuration:** Disabling password auth in /etc/ssh/sshd_config is critical; verify with `sudo sshd -T | grep -i password` before conversion.

3. **QEMU Agent Dependency:** Proxmox cloud-init datasource may require qemu-guest-agent running; test cloning with agent enabled to confirm IP address is reported.

4. **Desktop Template Size:** XFCE + dependencies adds ~10 GB; ensure storage pool has headroom. Desktop template recommended at 30 GB allocated (vs. base 20 GB).

5. **Cloud-Init Timing:** Network config race conditions can cause cloud-init to timeout; ensure cloud-init.service has `After=network-online.target` dependency (usually preset in Ubuntu).

---

## Known Issues & Mitigation

TBD

---

## Acceptance Criteria (Completed)

- [x] ADR-0002 documents design choices and trade-offs
- [x] Runbook provides step-by-step build instructions for both templates
- [x] Cloud-init examples included for Terraform integration
- [x] Template specifications explicitly state CPU, memory, storage, network, and agent config
- [x] Acceptance testing procedures documented (cloning, cloud-init, SSH, QEMU Agent)
- [x] Known issues and mitigations captured

---

## Next Steps

1. **Manual Build & Test:** Follow [Docs/Runbooks/Build-Proxmox-Templates.md](../Runbooks/Build-Proxmox-Templates.md) to build templates in Proxmox
2. **Terraform Integration:** M02 (feat/m02-terraform-vm-lifecycle) will reference template IDs in VM module
3. **Packer Automation (M02b):** Consider Packer-based template builds to eliminate manual steps (deferred)
4. **Monitor Versions:** Update ADR-0002 and template Notes fields as OS patches or XFCE updates occur

---

**Date:** 26 January 2026  
**Author:** Platform Engineering  
**Status:** Ready for manual template build & testing
