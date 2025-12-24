# Raspberry Pi NAS Build with OpenMediaVault

A 4-bay NAS solution built on Raspberry Pi 4 with OpenMediaVault, providing centralized storage for a Proxmox homelab cluster.

## Project Goals

- Provide shared storage for a 4-node Proxmox virtualization cluster
- Centralize VM backups and ISO storage via NFS
- Create organized file shares accessible via SMB/CIFS
- Implement proper access controls and security hardening
- Gain hands-on experience with enterprise storage concepts

## Hardware

| Component | Specification |
|-----------|---------------|
| Compute | Raspberry Pi 4 8GB |
| Enclosure | Argon EON 4-Bay NAS Case |
| OS Drive | 256GB SATA SSD |
| Fast Storage | 2TB SATA SSD |
| Bulk Storage | 4TB HDD |
| General Storage | 1TB HDD |

The Argon EON enclosure provides 4 SATA bays (2x 3.5" + 2x 2.5"), integrated cooling, OLED status display, and RTC battery backup.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Network (x.x.x.0/24)                     │
└─────────────────────────────────────────────────────────────┘
         │              │              │              │
    ┌────┴────┐    ┌────┴────┐   ┌────┴────┐   ┌────┴────┐
    │Proxmox  │    │Proxmox  │   │Proxmox  │   │Proxmox  │
    │ Node 1  │    │ Node 2  │   │ Node 3  │   │ Node 4  │
    └────┬────┘    └────┬────┘   └────┬────┘   └────┬────┘
         │              │              │              │
         └──────────────┴──────┬───────┴──────────────┘
                               │
                          NFS/SMB
                               │
                    ┌──────────┴──────────┐
                    │   OpenMediaVault    │
                    │   Raspberry Pi 4    │
                    ├─────────────────────┤
                    │ 256GB SSD (OS)      │
                    │ 2TB SSD (Fast)      │
                    │ 4TB HDD (Bulk)      │
                    │ 1TB HDD (General)   │
                    └─────────────────────┘
```

## Storage Organization

| Volume | Type | Purpose | Protocol |
|--------|------|---------|----------|
| 2TB-SSD-FAST | SSD | VM backups, ISOs, templates | NFS + SMB |
| 4TB-HDD-BULK | HDD | Archives, media, large files | SMB |
| 1TB-HDD-MISC | HDD | General purpose, projects | SMB |

### Naming Convention
Format: `<Size>-<Type>-<Purpose>`  
Examples: `2TB-SSD-FAST`, `4TB-HDD-BULK`, `1TB-HDD-MISC`

Intentionally used 4-letter descriptors for consistency.

## Services Configured

### File Sharing
- **SMB/CIFS** - Windows-compatible shares for desktop access
- **NFS** - High-performance shares for Proxmox cluster integration
- **FileBrowser** - Web-based file management interface

### Security
- **fail2ban** - Brute force protection for SSH, web UI, and SMB
- **Tiered user accounts** - Separate daily admin and break-glass emergency accounts
- **Group-based permissions** - Principle of least privilege

### Backup
- **OMV Config Backup** - Weekly automated backup of NAS configuration

## User Access Model

Implemented role-based access using Linux groups:

| Role | Access Level | Use Case |
|------|--------------|----------|
| Daily Admin | Full (SSH + Web + Shares) | Day-to-day management |
| Break Glass | Full (SSH + Web) | Emergency recovery only |
| Power User | Limited (SSH + Shares) | Advanced users, no sudo |
| Standard User | Shares only | Family/guest access |

### Key Groups
- `sudo` - Administrative CLI access
- `openmediavault-admin` - Web UI administration
- `_ssh` - SSH login permission
- `users` - File share access

## Proxmox Integration

The NAS provides datacenter-level storage visible to all cluster nodes:

**Storage Configuration:**
- Protocol: NFSv4
- Export: `/export/2TB-SSD-FAST`
- Content: Backups, ISO images, Container templates
- Access: All cluster nodes via single configuration

This enables:
- Centralized VM/CT backup storage
- Shared ISO library (upload once, available everywhere)
- Container template repository

## Technical Challenges Solved

### NFS Service Startup Failure
**Problem:** NFS server failed with `lockd configuration failure` and memory allocation errors.  
**Root Cause:** Kernel module loading order issue after system updates.  
**Solution:** Clean reboot resolved module dependencies; lockd is kernel-built on this Pi image.

### SMB Share Path Misconfiguration
**Problem:** Shares connected but showed no files.  
**Root Cause:** OMV created shares pointing to non-existent subdirectories.  
**Solution:** Corrected relative paths to expose drive roots.

### NTFS to ext4 Migration
**Problem:** 2TB SSD was NTFS-formatted from previous Windows use.  
**Solution:** Backed up needed data, wiped drive, reformatted as ext4 for native Linux performance.

### SMB Authentication Confusion
**Problem:** SSH credentials didn't work for SMB shares.  
**Root Cause:** Samba maintains separate password database.  
**Solution:** Set SMB passwords via `smbpasswd` command.

## Plugins Utilized

| Plugin | Function |
|--------|----------|
| sharerootfs | Enable shares on system drive |
| writecache | Reduce SSD wear via tmpfs overlay |
| cputemp | Temperature monitoring on dashboard |
| filebrowser | Web-based file manager |
| fail2ban | Intrusion prevention |
| backup | Configuration backup automation |

## Skills Demonstrated

- **Linux System Administration** - Service management, user/group permissions, filesystem operations
- **Network Storage Protocols** - NFS and SMB configuration and troubleshooting
- **Security Hardening** - fail2ban, tiered access controls, principle of least privilege
- **Virtualization Integration** - Proxmox datacenter storage configuration
- **Troubleshooting** - Systematic diagnosis of service failures and permission issues
- **Documentation** - Comprehensive technical documentation for reproducibility

## Future Enhancements

- [ ] VLAN segmentation for storage traffic isolation
- [ ] Restrict NFS exports to specific Proxmox node IPs
- [ ] Implement mergerfs for HDD pooling
- [ ] Add snapraid for parity-based data protection
- [ ] Uptime Kuma monitoring integration
- [ ] Automated drive health alerts via SMART monitoring

## Lessons Learned

1. **Test network connectivity first** - Hours of troubleshooting revealed the Pi was connected to the wrong network segment.

2. **SMB ≠ Linux authentication** - Samba maintains its own password database; SSH credentials don't automatically work.

3. **Reboot solves kernel module issues** - Sometimes the oldest IT advice is still valid.

4. **Name things consistently** - A clear naming convention (`SIZE-TYPE-PURPOSE`) prevents confusion as storage grows.

5. **Separate admin accounts** - Having a break-glass account saved time when testing lockouts.

## Related Projects

- [Proxmox Cluster Build](link-to-repo) - 4-node virtualization cluster
- [Homelab Network Infrastructure](link-to-repo) - ASUS ExpertWiFi setup

---

*Part of the X-03-SecureNet homelab project*
