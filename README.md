# Active Directory Lab

## Overview
This repository documents my implementation of the Active Directory lab assignment from CS 596: Applied Computer Security. This lab was completed as coursework, with the lab instructions and requirements designed by the course instructor. My implementation demonstrates a secure Windows-based corporate network including domain controllers, workstations, and server roles.

## Environment Architecture

### Network Components
- **Domain Controllers**: Primary (Windows Server 2025) and Secondary (Windows Server 2022)
- **Internal Network**: Protected corporate network (10.42.79.0/24)
- **DMZ Network**: Semi-trusted zone for public-facing services (10.42.78.0/24)
- **Firewall**: OPNsense firewall connecting all networks
- **Client Workstation**: Windows 10 workstation joined to the domain

### Key Features
- Complete Active Directory domain setup (cs596.internal)
- DNS infrastructure integrated with Active Directory
- Multi-tier network security with proper segmentation
- Domain user management and group policy implementation
- DMZ web server with proper firewall rules and port forwarding

## Technical Implementation

### Domain Controllers
- Configured primary DC (dc1) with Windows Server 2025
- Established backup DC (dc2) with Windows Server 2022
- Configured DNS zones for internal and reverse lookups
- Implemented proper LDAP authentication

### Network Security
- OPNsense firewall configuration with three network interfaces
- Network segmentation between internal and DMZ networks
- Firewall rules preventing DMZ to internal network traffic
- Port forwarding for DMZ web services

### Client Configuration
- Domain-joined Windows 10 workstation
- User authentication through domain credentials
- Network configuration via DHCP from domain controller

### Web Services
- Nginx web server in DMZ running on Debian 12 (containerized)
- Proper port forwarding through firewall
- Secure access controls

## Skills Developed
- Windows Server administration
- Active Directory domain implementation
- Network security and firewall configuration
- DNS server setup and configuration
- Containerization with LXC/Docker
- Virtualization with Proxmox

## Tools and Technologies
- **Hypervisor**: Proxmox VE 8.3
- **Firewall**: OPNsense 24.7
- **Server OS**: Windows Server 2025/2022
- **Client OS**: Windows 10 Education
- **Web Server**: Nginx on Debian 12
- **Virtualization**: Mix of full VMs and LXC containers

## Learning Outcomes
This project provided hands-on experience with enterprise network infrastructure, focusing on:
1. Active Directory domain design and implementation
2. Multi-tier network security architecture
3. DNS configuration and management
4. Firewall configuration and security policies
5. Understanding the relationship between various network components
6. Server hardening and security best practices
