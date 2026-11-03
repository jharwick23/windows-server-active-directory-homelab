# Windows Server 2022 Active Directory Home Lab

## Overview

This project documents the setup and administration of a virtualized Windows Server environment designed to develop practical IT infrastructure, networking, and systems administration skills.

The lab uses Windows Server 2022 as a domain controller and multiple Windows 11 workstations as domain-joined clients. It is hosted on Ubuntu using KVM/QEMU and Virtual Machine Manager.

The project focuses on Active Directory, DNS, DHCP, Group Policy, user and group management, file sharing, PowerShell administration, and Windows network troubleshooting.

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
- Group Policy Management
- PowerShell

## Lab Environment

| System | Operating System | Role |
|---|---|---|
| DC01 | Windows Server 2022 | Domain Controller, DNS, DHCP |
| Windows 11 Client 1 | Windows 11 | Domain-joined workstation |
| WIN11-02 | Windows 11 | Second domain-joined workstation |
| Ubuntu Host | Ubuntu 24.04 | Virtualization host |

### Virtualization Configuration

The virtual machines run on Ubuntu using KVM/QEMU, managed through libvirt and Virtual Machine Manager.

Each Windows client has access to the lab's isolated network. The virtual machines can also use the default NAT network for internet connectivity.

## Domain Configuration

| Setting | Value |
|---|---|
| Domain | homelab.test |
| NetBIOS Domain | HOMELAB |
| Domain Controller | DC01 |
| Domain Functional Environment | Active Directory Domain Services |

## Network Architecture

The lab uses two separate virtual networks.

### Default NAT Network

| Setting | Value |
|---|---|
| Network | default |
| Subnet | 192.168.122.0/24 |
| Gateway | 192.168.122.1 |
| DHCP | Managed by libvirt |

This network provides internet connectivity to the virtual machines through the Ubuntu host.

### Isolated Lab Network

| Setting | Value |
|---|---|
| Network | lab-isolated |
| Subnet | 10.10.10.0/24 |
| Gateway | None |
| DHCP | Windows Server |
| DNS | Windows Server |

This network provides an isolated environment for internal communication, domain services, and networking exercises.

No default gateway is configured because the isolated network does not have a router.

### Static Server Configuration

DC01 has two network adapters:

| Adapter | IP Address | Purpose |
|---|---|---|
| Ethernet | 192.168.122.39 | Default NAT network |
| Ethernet 2 | 10.10.10.10 | Isolated lab network |

The isolated network adapter uses a statically assigned IP address.

### Client Network Configuration

Both Windows 11 clients have network connectivity through the isolated lab network.

| Device | Lab Network IP | Assignment |
|---|---|---|
| Windows 11 Client 1 | 10.10.10.100 | DHCP reservation |
| WIN11-02 | 10.10.10.101 | Dynamic DHCP |

Both clients receive their DNS configuration from DC01.

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

### User and Group Management

Created fictional test user accounts for different departments.

Created the following global security groups:

- GG_Accounting
- GG_HR
- GG_IT
- GG_Sales

Assigned users to their respective departmental security groups.

Practiced managing user accounts, group memberships, and account status through Active Directory Users and Computers and PowerShell.

### Domain-Joined Workstations

Joined two Windows 11 workstations to the `homelab.test` domain.

Configured both clients to use DC01 for DNS resolution over the isolated lab network.

Verified domain authentication using domain user credentials.

The second workstation, `WIN11-02`, successfully received its network configuration through DHCP before being joined to the domain.

## DNS Configuration

Configured Windows Server DNS as part of the Active Directory domain environment.

Configured the Windows 11 clients to use DC01 (`10.10.10.10`) as their DNS server on the isolated network.

### Forward Lookup

Configured and tested forward DNS resolution.

Example:

```cmd
nslookup DC01.homelab.test 10.10.10.10
```

The hostname successfully resolved to the domain controller's registered IP addresses:

- 10.10.10.10
- 192.168.122.39

### Reverse Lookup

Created an IPv4 reverse lookup zone for the isolated network:

```text
10.10.10.0/24
```

Created a Pointer (PTR) record mapping:

```text
10.10.10.10 → DC01.homelab.test
```

This allows DNS to resolve an IP address back to its corresponding hostname.

## DHCP Configuration

Installed the Windows Server DHCP role and authorized the DHCP server in Active Directory.

Configured the DHCP server to provide addresses exclusively through the isolated lab network adapter.

Created and activated the `Lab Network` scope.

### DHCP Scope Configuration

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

### DHCP Reservations

Created a DHCP reservation for the first Windows 11 client.

| Setting | Value |
|---|---|
| Reserved IP | 10.10.10.100 |
| Device | Windows 11 Client 1 |
| Identification | Client network adapter MAC address |

The reservation ensures that the first workstation consistently receives the same IP address.

### DHCP Testing

Verified that both Windows 11 clients automatically received IP configurations from the Windows Server DHCP service.

Used the following commands to release and renew DHCP leases:

```cmd
ipconfig /release "Ethernet 2"
ipconfig /renew "Ethernet 2"
ipconfig /all
```

Verified assigned IP addresses and DHCP server information through Command Prompt and DHCP Manager.

## Group Policy

Created and configured Group Policy Objects to practice centralized Windows configuration management.

### Workstation Restrictions

Created a domain-level Group Policy Object named `Workstation Restrictions`.

Configured:

- Prohibit access to Control Panel and PC settings.

Created a separate `Accounting Department Policy` linked to the Accounting organizational unit.

### Account Lockout Policy

Configured account lockout settings through Default Domain Policy:

- Account lockout threshold: 5 invalid attempts
- Account lockout duration: 15 minutes
- Reset account lockout counter after: 15 minutes

Tested account lockout and manually unlocked a user account through Active Directory Users and Computers.

Practiced refreshing Group Policy using:

```cmd
gpupdate /force
```

Verified applied policies using:

```cmd
gpresult /r
```

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

Both share permissions and NTFS permissions are used to control access to shared folders.

## PowerShell Administration

Practiced using the Active Directory PowerShell module to manage users and groups.

Commands practiced include:

### Domain Information

```powershell
Get-ADDomain
Get-ADDomainController
```

### User Management

```powershell
Get-ADUser -Filter *

Get-ADUser -Filter * -Properties Department |
    Select Name, SamAccountName, Department

Disable-ADAccount
Unlock-ADAccount
```

### Group Management

```powershell
Get-ADGroupMember

Add-ADGroupMember
```

These exercises provide experience with basic Active Directory administration and command-line management.

## Current Project Status

### Completed

- Virtual machine environment setup using KVM/QEMU and libvirt
- Windows Server 2022 installation
- Active Directory domain creation
- Organizational unit and user creation
- Security group creation and membership management
- Two Windows 11 workstations joined to the domain
- DNS forward lookup configuration and testing
- DNS reverse lookup zone and PTR record configuration
- DHCP server installation and authorization
- DHCP scope configuration
- DHCP reservation creation
- Automatic IP assignment to multiple Windows clients
- Group Policy configuration
- Account lockout policy configuration and testing
- Initial departmental file-sharing configuration
- PowerShell Active Directory administration exercises

### Next Steps

- Further validate departmental file-sharing permissions
- Practice creating and managing additional domain users
- Configure additional Group Policy restrictions
- Practice workstation administration through Active Directory
- Expand PowerShell automation exercises
- Practice troubleshooting common DNS, DHCP, and domain connectivity issues

## Learning Objectives

- Understand Windows Server administration.
- Practice Active Directory user, group, and computer management.
- Configure and troubleshoot DNS and DHCP.
- Understand IP addressing, subnets, and virtual networking.
- Implement Group Policy and account security policies.
- Manage shared folders and NTFS permissions.
- Develop practical PowerShell administration skills.
- Understand centralized management of multiple domain-joined workstations.

## About

A virtualized Windows Server 2022 home lab built with Ubuntu, KVM/QEMU, and Windows 11 to practice Active Directory administration, DNS, DHCP, Group Policy, and Windows networking.
