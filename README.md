# Windows Server 2022 Active Directory Home Lab

## Overview

This project documents the setup and administration of a virtualized Windows Server environment designed to develop practical IT infrastructure, networking, and systems administration skills.

The lab uses Windows Server 2022 as a domain controller and Windows 11 as a domain-joined client. It is hosted on Ubuntu using KVM/QEMU and Virtual Machine Manager.

The project focuses on Active Directory, DNS, DHCP, Group Policy, user and group management, and Windows network administration.

## Technologies Used

- Ubuntu 24.04
- KVM/QEMU
- libvirt
- Virtual Machine Manager
- Windows Server 2022 Standard Evaluation
- Windows 11
- Active Directory Domain Services (AD DS)
- DNS Server
- DHCP Server
- Group Policy
- PowerShell

## Lab Environment

| System | Operating System | Role |
|---|---|---|
| DC01 | Windows Server 2022 | Domain Controller, DNS, DHCP |
| Windows 11 Client | Windows 11 | Domain-joined workstation |
| Ubuntu Host | Ubuntu 24.04 | Virtualization host |

### Domain Configuration

| Setting | Value |
|---|---|
| Domain | homelab.test |
| NetBIOS Domain | HOMELAB |
| Domain Controller | DC01 |

## Network Architecture

The lab uses two virtual networks.

### Default NAT Network

| Setting | Value |
|---|---|
| Network | default |
| Subnet | 192.168.122.0/24 |
| Gateway | 192.168.122.1 |
| DHCP | Managed by libvirt |

This network provides internet connectivity to the virtual machines.

### Isolated Lab Network

| Setting | Value |
|---|---|
| Network | lab-isolated |
| Subnet | 10.10.10.0/24 |
| Gateway | None |
| DHCP | Windows Server |

This network is used for internal lab communication and DHCP exercises.

### Static Server Configuration

DC01 has two network adapters:

| Adapter | IP Address | Purpose |
|---|---|---|
| Ethernet | 192.168.122.39 | NAT network |
| Ethernet 2 | 10.10.10.10 | Isolated lab network |

The isolated adapter uses a static IP address.

### DHCP Scope

Windows Server provides DHCP for the isolated network.

| Setting | Value |
|---|---|
| Scope Name | Lab Network |
| Network | 10.10.10.0/24 |
| Starting Address | 10.10.10.100 |
| Ending Address | 10.10.10.200 |
| Subnet Mask | 255.255.255.0 |
| Lease Duration | 8 days |
| DNS Server | 10.10.10.10 |
| DNS Domain | homelab.test |
| Default Gateway | Not configured |

No default gateway is configured because the isolated network does not have a router.

## Active Directory Configuration

Installed and configured Active Directory Domain Services and promoted DC01 to a domain controller.

Created the `homelab.test` domain.

Created the following organizational unit structure:

```text
homelab.test
├── Employees
│   ├── Accounting
│   ├── HR
│   ├── IT
│   └── Sales
├── Workstations
└── Groups
```

Created test user accounts and department-based security groups:

- GG_Accounting
- GG_HR
- GG_IT
- GG_Sales

Assigned users to their respective departmental groups.

## DNS Configuration

Configured Windows Server DNS as part of the Active Directory domain environment.

Configured the Windows 11 client to use DC01 for DNS on the isolated network.

Verified DNS resolution using:

```cmd
nslookup DC01.homelab.test 10.10.10.10
```

The hostname successfully resolved to the domain controller's IP addresses.

## DHCP Configuration

Installed the Windows Server DHCP role and authorized the DHCP server in Active Directory.

Created and activated the `Lab Network` scope.

Configured DHCP options for the DNS server and domain name.

Verified that the Windows 11 client automatically received an IP address from the configured scope.

Example client configuration:

```text
IPv4 Address: 10.10.10.100
Subnet Mask: 255.255.255.0
DHCP Server: 10.10.10.10
DNS Server: 10.10.10.10
DNS Suffix: homelab.test
```

## Group Policy

Created and configured Group Policy Objects to practice centralized Windows configuration management.

Configured restrictions on access to Control Panel and Windows Settings.

Created a separate policy linked to the Accounting organizational unit.

Configured an account lockout policy:

- Account lockout threshold: 5 invalid attempts
- Account lockout duration: 15 minutes
- Reset account lockout counter after: 15 minutes

Tested account lockout and manually unlocked a user account through Active Directory Users and Computers.

## File Sharing and Permissions

Created departmental folders under:

```text
C:\CompanyShares
├── Accounting
├── HR
├── IT
└── Sales
```

Configured network shares and permissions using departmental security groups.

The goal is to provide department-specific access to shared resources through Active Directory group membership.

## PowerShell Administration

Practiced using the Active Directory PowerShell module to manage users and groups.

Commands practiced include:

```powershell
Get-ADDomain
Get-ADDomainController
Get-ADUser -Filter *
Get-ADGroupMember
Add-ADGroupMember
Disable-ADAccount
Unlock-ADAccount
```

## Current Project Status

Completed:
- Virtual machine environment setup
- Windows Server 2022 installation
- Active Directory domain creation
- Organizational unit and user creation
- Security group creation and membership management
- DNS configuration and testing
- DHCP server installation and scope configuration
- Windows 11 DHCP lease verification
- Group Policy configuration
- Account lockout testing
- Initial departmental file-sharing configuration

In Progress:
- Testing network connectivity across the isolated network
- Further validating departmental file permissions
- Expanding PowerShell administration exercises

## Learning Objectives

- Understand Windows Server administration.
- Practice Active Directory user and group management.
- Configure and troubleshoot DNS and DHCP.
- Understand IP addressing, subnets, and virtual networking.
- Implement Group Policy and account security policies.
- Manage shared folders and NTFS permissions.
- Develop practical PowerShell administration skills.