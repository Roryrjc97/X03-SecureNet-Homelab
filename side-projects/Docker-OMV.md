# Docker Container Deployment on OpenMediaVault NAS

Implementing containerized services on a Raspberry Pi 4 NAS running OpenMediaVault 8, overcoming ARM64-specific challenges to deploy a self-hosted media streaming solution.

## Project Goals

- Deploy Docker containers on a Raspberry Pi 4 NAS
- Understand ARM64 container considerations
- Properly separate Docker storage from OS drive
- Configure persistent storage for containerized applications
- Deploy Jellyfin media server with existing media library

## The Challenge

Running Docker on OpenMediaVault with Raspberry Pi presents unique challenges:

1. **Architecture requirements** - OMV8 requires 64-bit ARM (arm64), dropping all 32-bit support
2. **Storage separation** - Docker must not run on the OS drive to prevent filling the boot media
3. **Plugin dependencies** - The compose plugin requires specific folder structures
4. **Container compatibility** - Not all Docker images support ARM64

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                 Raspberry Pi 4 (8GB)                     │
│                   OpenMediaVault 8                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌─────────────┐     ┌─────────────────────────────┐   │
│   │   OS Drive  │     │       2TB SSD (FAST)        │   │
│   │  256GB SSD  │     ├─────────────────────────────┤   │
│   │             │     │  /docker-storage (engine)   │   │
│   │  OMV System │     │  /docker-compose (files)    │   │
│   │             │     │  /docker-data (configs)     │   │
│   └─────────────┘     └─────────────────────────────┘   │
│                                                          │
│   ┌─────────────────────────────────────────────────┐   │
│   │              4TB HDD (BULK)                      │   │
│   │  /Data-Jellyfin/movies                          │   │
│   │  /Data-Jellyfin/tv                              │   │
│   │  /Data-Jellyfin/musicvideos                     │   │
│   └─────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Implementation

### Storage Strategy

**Critical lesson:** Never install Docker on the OS partition.

Created three dedicated shared folders on the fast SSD:

| Folder | Purpose |
|--------|---------|
| docker-storage | Docker engine data (images, layers) |
| docker-compose | Compose file definitions |
| docker-data | Container persistent data |

This separation ensures:
- OS drive remains stable and uncrowded
- Container data survives Docker engine updates
- Easy backup of configurations vs. rebuildable images

### Compose Plugin Configuration

Used OMV's openmediavault-compose plugin rather than manual Docker installation:

**Benefits:**
- GUI integration for managing containers
- Automatic updates with system updates
- Scheduled backup capabilities
- Community support for troubleshooting

**Pre-requisites:**
1. Enable Docker repository in omv-extras
2. Clear apt cache if package issues occur
3. Create storage folders before plugin configuration

### Container Deployment: Jellyfin

Deployed Jellyfin media server using LinuxServer.io's ARM64-compatible image:

```yaml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=100
      - TZ=America/Chicago
    volumes:
      - /path/to/docker-data/jellyfin:/config
      - /path/to/media/tv:/data/tvshows
      - /path/to/media/movies:/data/movies
    ports:
      - 8096:8096
    restart: unless-stopped
```

**Key considerations:**
- PUID/PGID match file ownership on host
- Config on fast SSD, media on bulk HDD
- Timezone set for correct metadata

## Technical Challenges Solved

### Docker Package Installation Failure

**Problem:** Compose plugin failed with "docker-ce package not found"  
**Root cause:** Docker repository not enabled, apt cache stale  
**Solution:** Enable Docker repo in omv-extras, run `apt clean`

### Storage Path Confusion

**Problem:** OMV uses UUID-based mount points like `/srv/dev-disk-by-uuid-xxxxx`  
**Solution:** Used OMV shared folders which abstract the paths, then verified actual mounts via `lsblk -f`

### Permission Denied on Docker Commands

**Problem:** Non-root user couldn't run Docker commands  
**Solution:** Either use `sudo` or add user to docker group (security tradeoff)

### ARM64 Kernel Warnings

**Problem:** Warnings about memory/swap limit support  
**Cause:** Pi kernel configuration - normal behavior  
**Impact:** None - safely ignored

## Verification Process

Before deploying services, validated the Docker installation:

| Check | Command | Expected Result |
|-------|---------|-----------------|
| Service status | `systemctl status docker` | active (running) |
| Storage location | `docker info \| grep "Docker Root Dir"` | Points to SSD |
| Pull/run test | `docker run hello-world` | Successful output |
| Compose available | `docker compose version` | Version displayed |

## Results

| Metric | Value |
|--------|-------|
| Docker storage | 2TB SSD (dedicated) |
| Container deployed | Jellyfin |
| Media accessible | TV, Movies, Music Videos |
| Access method | Web UI on port 8096 |

## Skills Demonstrated

- **Container orchestration** - Docker Compose for service definition
- **Storage architecture** - Separating concerns across drives
- **ARM64 considerations** - Understanding platform-specific requirements
- **Troubleshooting** - Diagnosing package and permission issues
- **Linux administration** - Mount points, permissions, systemd services
- **Self-hosting** - Running production services on personal infrastructure

## Lessons Learned

1. **Read the architecture requirements** - OMV8 dropping 32-bit support caused many reported failures. Always verify platform compatibility.

2. **Separate Docker from OS** - Flash-based boot media fills quickly. Docker storage belongs on dedicated drives.

3. **Use established images** - LinuxServer.io provides well-maintained ARM64 images with consistent conventions.

4. **GUI vs CLI is contextual** - OMV's compose plugin provides good GUI management. CLI remains valuable for troubleshooting.

5. **Verify before deploying** - Running test containers and checking storage paths prevents issues with actual services.

## Future Services

Candidates for containerization on this NAS:
- Navidrome (music streaming)
- Nextcloud (personal cloud)
- Paperless-ngx (document management)
- Homepage (dashboard)

## Related Projects

- [OpenMediaVault NAS](projects/OMV-NAS.md) - The underlying NAS platform
- [Proxmox Cluster](/projects/proxmox-cluster.md) - Alternative container host for heavier workloads

---

*Part of the X-03-SecureNet homelab project*
