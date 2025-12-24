# Ansible Configuration Management Setup

Implementing infrastructure automation with Ansible to manage a 4-node Proxmox virtualization cluster, eliminating repetitive manual configuration tasks.

## Project Goals

- Automate user account creation across multiple Linux servers
- Learn infrastructure-as-code principles
- Reduce configuration drift between cluster nodes
- Build foundation for future automation tasks

## The Problem

After manually configuring admin accounts on the first Proxmox node, I faced repeating the same steps on three more nodes:
- Create user accounts
- Set passwords
- Configure sudo access
- Verify permissions

Manual repetition across 4 nodes is tedious and error-prone. At scale (10, 100, 1000 servers), it's impossible.

## The Solution

Ansible automates configuration by:
1. Defining desired state in YAML playbooks
2. Connecting to target hosts via SSH
3. Executing tasks in parallel across all hosts
4. Reporting success/failure for each host

## Architecture

```
┌─────────────────────┐
│   Control Node      │
│   (WSL Ubuntu)      │
│                     │
│   ansible-playbook  │
└──────────┬──────────┘
           │
     SSH (parallel)
           │
     ┌─────┼─────┐
     │     │     │
     ▼     ▼     ▼
┌──────┐┌──────┐┌──────┐
│pve-02││pve-03││pve-04│
└──────┘└──────┘└──────┘
```

## Implementation

### Inventory Definition

Hosts organized into logical groups:

```ini
[proxmox_nodes]
pve-02 ansible_host=x.x.x.32
pve-03 ansible_host=x.x.x.33
pve-04 ansible_host=x.x.x.34

[proxmox_nodes:vars]
ansible_user=root
```

### User Creation Playbook

```yaml
---
- name: Create admin accounts on Proxmox nodes
  hosts: proxmox_nodes
  become: yes

  tasks:
    - name: Create admin account
      user:
        name: Admin-Rory
        shell: /bin/bash
        groups: sudo
        append: yes
        create_home: yes

    - name: Create breakglass account
      user:
        name: breakglass
        shell: /bin/bash
        groups: sudo
        append: yes
        create_home: yes
```

### Execution

Single command configures all nodes:

```bash
ansible-playbook -i inventory.ini create_users.yml
```

**Result:** Three servers configured in seconds instead of 15+ minutes of manual work.

## Key Concepts Learned

| Concept | Description |
|---------|-------------|
| Agentless | No software installed on managed nodes - just SSH |
| Idempotent | Running twice produces same result - safe to re-run |
| Declarative | Define desired state, Ansible figures out how |
| Parallel execution | All hosts configured simultaneously |
| Modules | Pre-built actions (apt, user, service, copy, etc.) |

## Ad-Hoc Commands

Beyond playbooks, Ansible runs one-off commands across all hosts:

```bash
# Test connectivity
ansible proxmox_nodes -m ping

# Install package on all nodes
ansible proxmox_nodes -m apt -a "name=sudo state=present"

# Check if user exists
ansible proxmox_nodes -m shell -a "id Admin-Rory"
```

## Technical Challenges

### SSH Key Authentication
**Challenge:** Password-based SSH doesn't scale and requires additional packages.  
**Solution:** Generated ED25519 key pair, distributed public key to all nodes using `ssh-copy-id`. Passwordless authentication enabled.

### Missing sudo on Proxmox
**Challenge:** Proxmox uses minimal Debian without sudo installed.  
**Solution:** Single Ansible command installed sudo across all nodes:
```bash
ansible proxmox_nodes -m apt -a "name=sudo state=present"
```

### Special Characters in Passwords
**Challenge:** Bash interprets `!` in passwords as history expansion.  
**Solution:** Wrap variables in single quotes to prevent shell interpretation.

## Results

| Metric | Manual | Ansible |
|--------|--------|---------|
| Time to configure 3 nodes | ~15 min | ~30 sec |
| Risk of typos | High | None |
| Documented/repeatable | No | Yes |
| Scales to 100 nodes | No | Yes |

## Skills Demonstrated

- **Configuration Management** - Ansible playbooks and inventory
- **Infrastructure as Code** - Version-controllable configuration
- **SSH Key Management** - Passwordless authentication setup
- **Linux Administration** - User creation, sudo configuration
- **YAML** - Playbook syntax and structure
- **Problem Solving** - Debugging SSH and shell escaping issues

## Future Automation Candidates

- System updates across all nodes
- SSH hardening configuration
- Monitoring agent deployment
- Backup job configuration
- Security baseline enforcement

## Lessons Learned

1. **Do it manually first** - Understanding the process makes automation easier and debugging possible.

2. **SSH keys are essential** - Password-based automation is fragile. Invest time in proper key management.

3. **Idempotency matters** - Well-written playbooks can be safely re-run without side effects.

4. **Start small** - A simple user creation playbook teaches core concepts before tackling complex deployments.

5. **Document as you go** - Playbooks ARE documentation when written clearly.

## Related Projects

- [Proxmox Cluster](../projects/proxmox-cluster.md) - The infrastructure being managed
- [OpenMediaVault NAS](../projects/OMV-NAS.md) - Future Ansible target

---

*Part of the X-03-SecureNet homelab project*
