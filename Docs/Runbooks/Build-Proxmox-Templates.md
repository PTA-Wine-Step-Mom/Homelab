# Runbook: Build Proxmox VM Templates (feat/proxmox-templates)

## Goal

Create two reusable Proxmox VM templates that support cloud-init, SSH key authentication, cloning, and non-interactive provisioning:
1. **tpl-rds-base-u2404-s**: Minimal Ubuntu 24.04.03-liveserver base
2. **tpl-rds-desktop-u2404-d**: Ubuntu 24.04.03-desktop with XFCE desktop for session hosts

Templates are the foundation for Terraform VM provisioning (M02).

## Prerequisites

- Proxmox VE node with SSH access and sufficient storage (local-lvm or shared NFS)
- Ubuntu 24.04.03 Server ISO (24.04.03-liveserver) uploaded to Proxmox (e.g., `/var/lib/vz/template/iso/ubuntu-24.04.03-live-server-amd64.iso`)
- Proxmox account with VM creation and modification privileges
- Linux workstation for ISO verification (optional)

## Template Specifications

| Property | tpl-rds-base-u2404-s | tpl-rds-desktop-u2404-d |
|----------|-------------------|----------------------|
| **OS** | Ubuntu Server 24.04.03-liveserver | Ubuntu Server 24.04.03-desktop + XFCE |
| **CPU** | 2 vCPU (adjustable per clone) | 2 vCPU (adjustable per clone) |
| **Memory** | 2 GB (adjustable per clone) | 2 GB (adjustable per clone) |
| **Storage** | 20 GB (local-lvm or NFS) | 30 GB (local-lvm or NFS) |
| **Network** | DHCP + cloud-init | DHCP + cloud-init |
| **Cloud-Init** | Enabled (cloudinit) | Enabled (cloudinit) |
| **QEMU Agent** | Enabled | Enabled |
| **SSH** | Keys only; password auth disabled | Keys only; password auth disabled |
| **Notes Field** | `tpl-rds-base-u2404-s v1.0 [date]` | `tpl-rds-desktop-u2404-d v1.0 [date]` |

## Build Steps: tpl-rds-base-u2404-s

### 1. Create Base VM via Proxmox UI or CLI

**Via UI:**
- Proxmox → Create VM → General → Name: `tpl-rds-base-u2404-s`, VM ID: auto
- OS: Linux, ISO: Ubuntu 22.04 Server
- System: SCSI controller (VirtIO-SCSI), BIOS (OVMF/UEFI)
- Disks: 20 GB (local-lvm or NFS storage)
- CPU: 2 sockets, 1 core (totaling 2 vCPU)
- Memory: 2048 MB, Swap: 512 MB
- Network: vmbr0 (or appropriate bridge), DHCP
- Confirm → Create

**Alternatively via CLI:**
```bash
qm create 100 --name tpl-rds-base-u2404-s \
  --ide2 local:iso/ubuntu-22.04-live-server-amd64.iso,media=cdrom \
  --sockets 1 --cores 2 --memory 2048 \
  --net0 virtio,bridge=vmbr0 \
  --scsi0 local-lvm:20 \
  --machine q35 --bios ovmf
```

### 2. Boot and Install Ubuntu

- Boot VM; select "Install Ubuntu Server"
- **Hostname:** `rds-base` (will be overridden by cloud-init per-instance)
- **Network:** Accept DHCP
- **Storage:** Use full disk; LVM optional
- **Software:** Only OpenSSH server (do NOT select other packages yet)
- **Allow SSH key import:** Yes (if available; skip otherwise)
- **Finish installation; reboot**

### 3. Post-Install: Enable cloud-init and SSH

**SSH into the newly booted VM:**
```bash
ssh -i <your-key> ubuntu@<vm-ip>
```

**Enable cloud-init and configure SSH:**
```bash
# Update package index
sudo apt-get update

# Install QEMU Guest Agent (enables Proxmox integration)
sudo apt-get install -y qemu-guest-agent

# Ensure cloud-init is installed (usually preinstalled on Ubuntu cloud images)
sudo apt-get install -y cloud-init

# Disable password authentication permanently
sudo sed -i 's/#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Set timezone (UTC for consistency)
sudo timedatectl set-timezone UTC

# Clean cloud-init and logs for template reuse
sudo cloud-init clean --logs --seed
sudo truncate -s 0 /var/log/auth.log /var/log/syslog
```

### 4. Power Off and Convert to Template

**From the VM or via Proxmox UI:**
```bash
sudo poweroff
```

**Via Proxmox UI:**
- Right-click VM `tpl-rds-base-u2404-s` → Convert to Template

**Alternatively via CLI:**
```bash
qm set 100 --template 1
```

### 5. Edit Template Notes

**Via Proxmox UI:**
- Select template → Notes tab → Add: `tpl-rds-base-u2404-s v1.0 [date] | Ubuntu 24.04.03-liveserver, SSH key-only, cloud-init enabled`

---

## Build Steps: tpl-rds-desktop-u2404-d

### 1. Clone Base Template

**Via Proxmox UI:**
- Right-click `tpl-rds-base-u2404-s` → Clone
- New ID: auto-increment (e.g., 101)
- New name: `tpl-rds-desktop-u2404-d`
- Mode: Full clone
- Storage: Same as base template

**Alternatively via CLI:**
```bash
qm clone 100 101 --name tpl-rds-desktop-u2404-d --full
```

### 2. Boot Cloned VM and Install Desktop

**Via Proxmox UI:**
- Right-click `tpl-rds-desktop-u2404-d` → Start
- Wait for cloud-init to finish (check logs)

**SSH into the VM:**
```bash
ssh -i <your-key> ubuntu@<vm-ip>
```

**Install XFCE and display libraries:**
```bash
sudo apt-get update
sudo apt-get install -y xfce4 xfce4-goodies xorgxfce4-terminal
sudo apt-get install -y xvfb xauth

# Optional: Install lightweight media/codec support for RDP
sudo apt-get install -y pulseaudio alsa-utils

# Clean package cache
sudo apt-get clean
sudo apt-get autoclean

# Clean cloud-init and logs again
sudo cloud-init clean --logs --seed
sudo truncate -s 0 /var/log/auth.log /var/log/syslog
```

### 3. Power Off and Convert to Template

```bash
sudo poweroff
```

**Via Proxmox UI:**
- Right-click `tpl-rds-desktop-u2404-d` → Convert to Template

**Alternatively via CLI:**
```bash
qm set 101 --template 1
```

### 4. Edit Template Notes

**Via Proxmox UI:**
- Select template → Notes tab → Add: `tpl-rds-desktop-u2404-d v1.0 [date] | Ubuntu 24.04.03-liveserver + XFCE 4.18, SSH key-only, cloud-init enabled, for RDP session hosts`

---

## Acceptance Criteria & Testing

### Test 1: Template Cloning (Proxmox UI)

1. Right-click `tpl-rds-base-u2404-s` → Clone → `test-base-clone`
2. Boot clone; verify cloud-init applies
3. SSH login succeeds with SSH key; password auth fails
4. Verify QEMU Guest Agent running: `sudo systemctl status qemu-guest-agent`
5. Verify cloud-init logs: `sudo tail -50 /var/log/cloud-init-output.log`
6. Confirm hostname matches clone name or cloud-init override
7. Delete clone after test

### Test 2: Desktop Template Cloning

1. Right-click `tpl-rds-desktop-u2404-d` → Clone → `test-desktop-clone`
2. Boot clone; verify cloud-init applies
3. SSH login succeeds with SSH key
4. Verify XFCE installed: `which xfce4-session`
5. Verify display server capabilities: `Xvfb -version`
6. Delete clone after test

### Test 3: Cloud-Init User Data (Optional, for M02 integration)

**Create a test user_data file (e.g., for Terraform testing):**

```yaml
#cloud-config
hostname: test-rds-host-001
fqdn: test-rds-host-001.local
users:
  - name: admin
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh-authorized-keys:
      - ssh-rsa AAAA...your-public-key...
packages:
  - curl
  - vim
  - htop
runcmd:
  - echo "Custom cloud-init applied" >> /var/log/cloud-init-output.log
```

**Clone base template; at creation, provide custom cloud-init config:**
1. Proxmox UI → New VM from template → Advanced → cloud-init → paste user_data
2. Boot; verify hostname, admin user, SSH keys, and custom packages present
3. Confirm custom runcmd executed: `cat /var/log/cloud-init-output.log`

---

## Known Issues & Mitigation

| Issue | Cause | Mitigation |
|-------|-------|-----------|
| Cloud-init hangs on boot | Network config race condition | Ensure cloud-init.service has `After=network-online.target` |
| QEMU Agent missing IP | Agent not installed in template | Verify `sudo systemctl status qemu-guest-agent` and reinstall if needed |
| SSH key auth fails | PermitRootLogin or PubkeyAuthentication not configured | Verify /etc/ssh/sshd_config; restart sshd after edits |
| XFCE fails to start | Display libraries missing | Verify `xfce4` and `xorg` packages installed; check Xvfb availability |
| Disk space exceeded (desktop template) | Extra packages during XFCE install | Use `sudo apt-get clean` before template conversion |

---

## Documentation & Version Control

1. **Decisions:** See [Docs/Decisions/ADR-0003-proxmox-templates.md](../Decisions/ADR-0003-proxmox-templates.md) for design rationale and choices.
2. **Build Log:** Add notes and lessons learned to [Docs/Build-Log/2026-01-Templates.md](../Build-Log/2026-01-Templates.md).
3. **Git:** Commit this runbook and ADR; templates themselves are Proxmox artifacts (not in Git; reference by ID in Terraform).

---

## Next Steps

- Terraform integration: [Docs/Runbooks/Provision-VMs.md](Provision-VMs.md) (M02)
- Packer-based automation (deferred to M02b): Document in Build-Log
- Monitor template versioning: Update ADR/Notes field if OS patches or XFCE upgrades occur
