# 4-Node Proxmox Virtualization Cluster

A high-availability virtualization cluster built with repurposed enterprise and consumer hardware, providing a platform for learning enterprise infrastructure concepts and hosting homelab services.

## Project Goals

- Build a multi-node virtualization cluster with high availability capabilities
- Create a platform for learning enterprise technologies (Active Directory, GPO, WDS)
- Gain hands-on experience with cluster management, quorum, and node failover
- Integrate shared storage via NFS for centralized backups and ISO management
- Repurpose existing hardware efficiently rather than purchasing dedicated servers

## Cluster Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    x03-datacenter Cluster                       │
│                     (Quorum: 3/4 nodes)                         │
└─────────────────────────────────────────────────────────────────┘
        │               │               │               │
   ┌────┴────┐     ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
   │ Node 1  │     │ Node 2  │    │ Node 3  │    │ Node 4  │
   │  Dell   │     │  Ryzen  │    │ Lenovo  │    │ Lenovo  │
   │ Laptop  │     │ Desktop │    │  M70q   │    │  M90q   │
   ├─────────┤     ├─────────┤    ├─────────┤    ├─────────┤
   │ i5-1335U│     │ R5 4500 │    │i5-12500T│    │i5-10500T│
   │  32GB   │     │  32GB   │    │  32GB   │    │  32GB   │
   │ Battery │     │ RX 3050 │    │  1L PC  │    │  1L PC  │
   └────┬────┘     └────┬────┘    └────┬────┘    └────┬────┘
        │               │               │               │
        └───────────────┴───────┬───────┴───────────────┘
                                │
                           NFS Storage
                                │
                    ┌───────────┴───────────┐
                    │   OpenMediaVault NAS  │
                    │   (Backups + ISOs)    │
                    └───────────────────────┘
```

## Hardware Specifications

| Node | Hardware | CPU | RAM | Storage | Role |
|------|----------|-----|-----|---------|------|
| 1 | Dell Latitude 5540 | i5-1335U (10C/12T) | 32GB | 512GB NVMe | Infrastructure, HA services |
| 2 | Custom Desktop | Ryzen 5 4500 (6C/12T) | 32GB | 256GB + 500GB | GPU workloads, testing |
| 3 | Lenovo M70q Gen 3 | i5-12500T (6C/12T) | 32GB | 512GB NVMe | Production workloads |
| 4 | Lenovo M90q | i5-10500T (6C/12T) | 32GB | 512GB NVMe | Lab/staging |

**Total Cluster Resources:** 28 cores / 48 threads, 128GB RAM, ~1.8TB storage

### Node Specializations

**Node 1 (Dell Laptop):**
- Built-in battery provides UPS functionality (2-4 hours backup)
- 15W TDP CPU for power efficiency
- Hosts critical always-on services
- Screen removed for headless operation

**Node 2 (Ryzen Desktop):**
- Discrete AMD GPU for passthrough workloads
- Additional SATA SSD for high-performance VM storage
- Suitable for gaming VMs or GPU-accelerated tasks

**Node 3 & 4 (Lenovo Mini PCs):**
- Ultra-compact 1-liter form factor
- Enterprise-grade reliability
- Acquired from workplace hardware refresh (free)

## Cluster Configuration

### High Availability Design
- **Quorum:** 3 of 4 nodes required for cluster operations
- **Failover:** Single node can fail without service interruption
- **Management:** Any node provides full cluster administration

### Storage Architecture

| Type | Purpose | Location |
|------|---------|----------|
| Local NVMe | VM disks, fast storage | Each node |
| NFS Share | Backups, ISOs, templates | Centralized NAS |

The NFS integration enables:
- Centralized backup storage accessible from all nodes
- Single ISO library (upload once, deploy anywhere)
- Container template repository

### Network Design
- All nodes on same subnet (flat network)
- Dedicated IP range for Proxmox hosts
- Separate range for VMs and services
- Future: VLAN segmentation for traffic isolation

## Workloads Deployed

### Infrastructure Services
- **DNS Filtering** - Network-wide ad blocking via Pi-hole
- **VPN Gateway** - WireGuard for secure remote access
- **Monitoring** - Uptime Kuma for service health tracking

### Active Directory Lab
Complete Windows Server environment for enterprise skill development:
- **Domain Controller** - Windows Server 2022, AD DS, DNS, GPO
- **Admin Workstation** - RSAT tools, domain management
- **Test Workstations** - GPO testing, user simulation
- **File Server** - DFS, NTFS permissions, share management (planned)

## Technical Challenges Solved

### Mixed CPU Architecture
**Challenge:** Cluster contains both Intel and AMD processors.  
**Solution:** Proxmox handles heterogeneous clusters well. Live migration between different CPU vendors requires CPU type set to "host" or a compatible baseline. VMs pinned to specific nodes when hardware-specific features needed.

### GPU Passthrough on Node 2
**Challenge:** Ryzen system with discrete GPU caused installer issues.  
**Solution:** Added `nomodeset` kernel parameter to boot configuration. Made permanent in GRUB for persistence across updates.

### Laptop as Server Node
**Challenge:** Dell Latitude had no traditional server features.  
**Solution:** Removed broken screen for headless operation. Built-in battery repurposed as UPS. Laptop NIC used directly (docking station caused network issues).

### Quorum Management
**Challenge:** Understanding cluster voting and failure scenarios.  
**Learning:** With 4 nodes, losing 2 nodes breaks quorum. Cluster becomes read-only to prevent split-brain. Documented recovery procedures for various failure scenarios.

## Skills Demonstrated

- **Virtualization** - Proxmox VE installation, configuration, cluster formation
- **High Availability** - Quorum concepts, failover planning, node redundancy
- **Linux Administration** - Debian-based system management, kernel parameters, systemd
- **Storage Integration** - NFS client configuration, shared storage for HA
- **Hardware Repurposing** - Converting consumer hardware to server roles
- **Network Planning** - IP allocation, subnet organization, future VLAN design
- **Windows Server** - Active Directory, Group Policy, enterprise administration
- **Documentation** - Comprehensive technical documentation and topology diagrams

## Naming Convention

**Hostnames:** `pve-<hardware>-node-<number>.<domain>`  
**Examples:**
- pve-dell-node-1
- pve-ryzen-node-2
- pve-12500t-node-3
- pve-10500t-node-4

This convention immediately identifies:
- Service type (pve = Proxmox)
- Physical hardware
- Cluster position
- Network domain

## Lessons Learned

1. **Repurposed hardware works well** - Enterprise mini PCs and even laptops make capable virtualization nodes. Total hardware cost: $0 (all repurposed).

2. **Quorum matters** - Understanding cluster voting prevents panic during outages. Document what happens at each failure level.

3. **Battery backup is valuable** - The laptop's built-in battery has already saved running VMs during brief power outages.

4. **Start with shared storage** - Adding NFS early made backup configuration trivial. Should have done this on day one.

5. **Mixed hardware is fine** - Intel and AMD coexist in the same cluster without issues for most workloads.

## Future Enhancements

- [ ] Upgrade Node 2 CPU (Ryzen 5 4500 → Ryzen 7 5700)
- [ ] VLAN segmentation for cluster traffic isolation
- [ ] Ceph distributed storage for true HA (requires dedicated drives)
- [ ] Automated backup scheduling with retention policies
- [ ] High Availability groups for critical VMs
- [ ] 5th node addition (max planned capacity)

## Related Projects

- [OpenMediaVault NAS](projects/OMV-NAS.md) - Shared storage for cluster backups
- [Active Directory Lab](projects/ad-lab.md) - Windows Server environment
- [Homelab Network Infrastructure](projects/network-infrastructure.md) - Underlying network

---

*Part of the X-03-SecureNet homelab project*
