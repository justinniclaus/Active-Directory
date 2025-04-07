# My Active Directory Lab Setup Guide

## Prerequisites

### Hardware Requirements I Used
- CPU: Intel i7 with virtualization support enabled in BIOS
- RAM: 32GB
- Storage: 1TB SSD
- Host OS: Fedora 41

### Software Used
- VMware Workstation Pro 17
- Proxmox VE 8.3 (running as VM)
- Windows Server 2025/2022 ISOs
- Windows 10 Education ISO
- OPNsense ISO
- VirtIO drivers ISO

## 1. Nested Virtualization Setup

### Installing Proxmox in VMware Workstation Pro
1. Created a new VM in VMware Workstation Pro with the following specs:
   - Guest OS: Other Linux 6.x kernel 64-bit
   - CPU: Allocated cores matching my physical CPU cores
   - Memory: 24GB
   - Storage: 500GB
   - Network: Bridged mode

2. Enabled nested virtualization:
   - VM Settings → Processors → Virtualize Intel VT-x/EPT or AMD-V/RVI

3. Installed Proxmox VE 8.3 within this VM:
   - Booted from Proxmox ISO
   - Configured IP address with .80 suffix
   - Set root password and email
   - Completed installation

## 2. Proxmox Configuration and Network Setup

### Proxmox Initial Setup
1. Accessed Proxmox web interface at https://[Proxmox-IP]:8006
2. Created non-root user account with administrative privileges:
   ```bash
   apt install sudo
   useradd -mU -G sudo -s /bin/bash failsafe
   passwd failsafe
   ```
3. Modified repositories to use non-subscription sources:
   ```
   vi /etc/apt/sources.list.d/pve-enterprise.list
   vi /etc/apt/sources.list.d/ceph.list
   ```
4. Applied system updates:
   ```
   apt update
   apt dist-upgrade
   ```

### Custom Network Configuration
I had to create a custom network configuration to solve connectivity issues with the default setup which is to use vmbr0 as the external network:

1. Created a secondary bridge interface (vmbr1) for NAT:
   ```
   # vi /etc/network/interfaces
   
   auto vmbr1
   iface vmbr1 inet static
       address 10.42.77.1/24
       bridge-ports none
       bridge-stp off
       bridge-fd 0
       post-up echo 1 > /proc/sys/net/ipv4/ip_forward
       post-up iptables -t nat -A POSTROUTING -s '10.42.77.0/24' -o ens33 -j MASQUERADE
       post-down iptables -t nat -D POSTROUTING -s '10.42.77.0/24' -o ens33 -j MASQUERADE
   ```

2. Connected vmbr1 (NAT) to vmbr0 (bridge) to enable internet connectivity for VMs
3. Created SDN zones and VNets as in the lab:
   - Created VLAN zone "primary" on vmbr0
   - Created VNets: DMZ (tag 78) and Internal (tag 79)
   - Applied SDN configuration

## 3. OPNsense Firewall Setup

### Creating the VM with Modified Network Setup
1. Created VM (ID: 7801) with following specifications:
   - OS: OPNsense ISO
   - CPU: 2 cores
   - Memory: 2048MB
   - Storage: 40GB (SCSI)
   - Network: 3 interfaces
     - Net0: Connected to vmbr1 (NAT) for WAN
     - Net1: DMZ network
     - Net2: Internal network

### Modified Network Configuration
1. Installed OPNsense and assigned interfaces:
   - WAN: Connected to vmbr1 for internet access (vtnet0)
   - LAN: Internal network (vtnet2)
   - OPT1: DMZ network (vtnet1)
2. Used DHCP for WAN interface initially
3. Set LAN IP to 10.42.79.1/24
4. Set DMZ IP to 10.42.78.1/24
5. Configured DNS to use upstream DNS server from vmbr1

### Firewall Rules
1. Create DMZ rules:
   - Block traffic from DMZ to LAN
   - Allow all outbound from DMZ
2. Create port forward:
   - WAN -> DMZ web server (HTTP)

## 4. Primary Domain Controller (DC1)

### Creating the VM
1. Created VM (ID: 7902) with these specifications:
   - Name: dc1
   - OS: Windows Server 2025 ISO
   - Added VirtIO drivers ISO as second CD drive
   - CPU: 2 cores
   - Memory: 2048MB
   - Storage: 32GB (SCSI)
   - Network: Connected to Internal network VLAN

### Challenges and Workarounds
1. Initially had SCSI driver issues during installation
   - Used VirtIO driver from the 2k25/amd64 folder
   - Had to manually load storage drivers during Windows setup

2. Network connectivity required custom configuration
   - Installed network drivers from VirtIO ISO
   - Manually configured static IP address: 10.42.79.2/24
   - Set gateway to OPNsense LAN interface: 10.42.79.1

### Active Directory Installation
1. Installed Windows Server 2025 with Desktop Experience
2. Set computer name to "dc1"
3. Updated Windows through Settings app
4. Added AD DS and DNS server roles through Server Manager
5. Promoted to domain controller:
   - Created new forest: cs596.internal
   - Set functional level to Windows Server 2016
   - Set DSRM password and completed installation

### DNS Configuration
1. After AD installation, added reverse lookup zones:
   - 10.42.77.0/24
   - 10.42.78.0/24
   - 10.42.79.0/24
2. Added these DNS records:
   - www.cs596.internal → 10.42.78.2
   - gw.cs596.internal → 10.42.79.1

## 5. Backup Domain Controller (DC2)

### Creating the VM
1. Create new VM (ID: 7903)
2. OS: Windows Server 2022 ISO
3. Add VirtIO drivers ISO
4. CPU: 2 cores
5. Memory: 2048MB
6. Storage: 32GB
7. Network: Internal network (10.42.79.3/24)

### Installation and Configuration
1. Install Windows Server 2022 with Desktop Experience
2. Set computer name to "dc2"
3. Join to domain cs596.internal
4. Add AD DS and DNS server roles
5. Promote to additional domain controller
6. Configure synchronization between DCs

## 6. Windows 10 Workstation (WKS)

### Creating the VM
1. Create new VM (ID: 7906)
2. OS: Windows 10 Education ISO
3. Add VirtIO drivers ISO
4. CPU: 2 cores
5. Memory: 2048MB
6. Storage: 40GB
7. Network: Internal network

### Installation and Configuration
1. Install Windows 10 Education
2. Set computer name to "wks"
3. Install VirtIO drivers
4. Join to domain cs596.internal
5. Log in with domain user account

## 7. DMZ Web Server

### Creating the Container
1. Download Debian 12 container template
2. Create new LXC container (ID: 7802)
3. Set hostname to "docker"
4. Storage: 8GB
5. Memory: 512MB
6. Network: DMZ network (10.42.78.2/24)

### Web Server Configuration
1. Update system and install sudo
2. Create non-root user
3. Install nginx web server
4. Configure basic website
5. Test access through firewall

## 8. Member Server and Server Core

### Member Server
1. Create new VM (ID: 7904)
2. OS: Windows Server 2025 Standard
3. Set hostname to "srv"
4. Join to domain

### Server Core
1. Create new VM (ID: 7905)
2. OS: Windows Server 2022 (non-Desktop Experience)
3. Set hostname to "core"
4. Join to domain

## Implementation Challenges and Solutions

### Network Connectivity Issues
**Problem**: Initially couldn't get external network connectivity using just vmbr0
**Solution**: Created a NAT network (vmbr1) connected to vmbr0 to provide internet access

**Problem**: OPNsense WAN interface couldn't reach the internet
**Solution**: Connected OPNsense WAN to vmbr1 NAT network instead of vmbr0

**Problem**: DNS resolution issues between networks
**Solution**: Modified DNS forwarding in OPNsense to use upstream DNS servers

### Virtualization Challenges
**Problem**: Performance issues with nested virtualization
**Solution**: Reduced VM memory allocation and optimized CPU settings

**Problem**: VirtIO drivers not detected properly
**Solution**: Mounted ISO properly and manually navigated to correct driver folders based on OS version

## Verification and Testing

### What Worked Well
1. Successfully created multi-tier network (Internal, DMZ, WAN)
2. Domain controllers properly configured and replicating
3. DNS resolution working between all networks
4. Web server accessible through port forwarding

### Lessons Learned
1. Nested virtualization requires careful resource allocation
2. Network design is critical for proper isolation and connectivity
3. VirtIO drivers dramatically improve performance but require proper installation
4. Understanding NAT and routing essential for proper network connectivity

This lab taught me skills in network troubleshooting, Windows domain administration, and virtualization management that will be applicable to real-world enterprise environments.
