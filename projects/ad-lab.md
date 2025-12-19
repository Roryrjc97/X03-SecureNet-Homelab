# Active Directory Lab

## Overview

Currently building a Windows Server 2022 Active Directory environment simulating 
a small enterprise. This lab demonstrates hands-on experience with 
identity management, group policy, DNS, and enterprise imaging.

## Architecture

| VM | Role | OS |
|----|------|-----|
| DC01 | Domain Controller + DNS | Windows Server 2022 |
| FS01 | File Server | Windows Server 2022 |
| WDS01 | Imaging Server (WDS/MDT) | Windows Server 2022 |
| ADMIN01 | Admin Workstation | Windows 11 |
| WS01-03 | Domain Workstations | Windows 11 |

**Domain:** X03.local  
**Hypervisor:** Proxmox VE (4-node cluster)

## Organizational Structure

| OU | Purpose |
|----|---------|
| IT | Admin accounts, IT staff |
| Executives | C-suite accounts |
| Finance | Finance department |
| HR | Human resources |
| Employees | General staff |
| Workstations | Domain-joined PCs |
| Servers | Member servers |

## Security Groups

| Group | Purpose |
|-------|---------|
| IT-Staff | IT department access |
| Executives-Staff | Executive access |
| Finance-Staff | Finance share access |
| HR-Staff | HR share access |
| Employees-Staff | General employee access |

## Skills Demonstrated

- AD DS installation and domain promotion
- DNS integration with Active Directory
- Organizational Unit design
- User and group management
- Security group strategy for file share permissions
- Windows Server administration

## Planned Additions

- [ ] Group Policy implementation
- [ ] File server with department shares
- [ ] GPO-mapped drives
- [ ] WDS/MDT imaging server
- [ ] PXE boot deployment with image selection

## What I Learned

- AD and DNS are tightly coupled - domain controllers must host DNS 
  for AD to function properly
- Organizational Units enable targeted Group Policy application
- Security groups should map to resource access, not org chart
- Separate admin accounts from daily-use accounts (rory.admin vs rory)
- Enterprise environments separate workstation and server policies
