# Windows Server Home Lab

A hands-on Windows Server infrastructure project built to develop System Administration skills through the deployment, administration, and troubleshooting of Active Directory, DNS, PKI, Group Policy, Hyper-V networking, and secure remote administration.

## Project Goals

- Build an enterprise-style Active Directory environment
- Configure supporting infrastructure services
- Practice administration and troubleshooting workflows
- Document lessons learned and real-world issues encountered
- Prepare for a Junior System Administrator role

## Technologies Used

### Infrastructure
- Windows Server 2022
- Windows 11 Pro
- Hyper-V
- Parallels Desktop

### Identity Services
- Active Directory Domain Services
- Organizational Units (OUs)
- User Administration
- Security Groups

### Network Services
- DNS
- DHCP (In Progress)
- DNS Forwarders
- Split DNS

### Security
- Active Directory Certificate Services (AD CS)
- Enterprise Root CA
- Certificate Auto-Enrollment
- Administrative Account Separation

### Remote Administration
- Remote Desktop Services (RDP)
- Tailscale

### Policy Management
- Group Policy
- Group Policy Preferences
  
## Active Directory Deployment

### Domain

Created and configured:

```text
josue.lab
```

### Installed Roles

- Active Directory Domain Services
- DNS Server

### Organizational Unit Structure

```text
Administration
Corp Users
Groups
Service Accounts
Workstations
```

### User Accounts

Created:

```text
jjarquin
jjarquin-admin
```

Implemented administrative account separation by using a dedicated privileged account for administrative tasks.

Added:

```text
jjarquin-admin
```

to:

```text
Domain Admins
```

Successfully tested interactive logons using both standard and administrative accounts.

---

## Lab Architecture

```text
MacBook M2
│
├── Tailscale
│
└── Parallels Desktop
     └── Windows 11 Pro Client

Dell Laptop
│
├── Windows 11 Pro Host
│
└── Hyper-V
      │
      └── DC01
            ├── Active Directory Domain Services
            ├── DNS
            ├── DHCP
            ├── Active Directory Certificate Services
            ├── File Shares
            └── Tailscale

Domain:
josue.lab
```

---

## DNS Administration and Troubleshooting

### DNS Configuration

Verified the following Active Directory integrated DNS zones:

```text
josue.lab
_msdcs.josue.lab
```

Configured DNS forwarders to enable external name resolution for domain clients.

### Validation Tools

Used the following tools to validate DNS functionality:

```cmd
nslookup
dcdiag /test:dns
```

### Outcome

- Successfully resolved internal domain records.
- Restored internet name resolution through DNS forwarders.
- Verified healthy Active Directory DNS integration.

### Lessons Learned

Active Directory relies heavily on DNS for authentication, domain joins, Group Policy processing, and service discovery.

---

## Active Directory Certificate Services (AD CS)

### Objective

Implement an internal Public Key Infrastructure (PKI) to support certificate-based services within the lab environment.

### Deployment

Installed:

```text
Active Directory Certificate Services
```

Configured:

```text
Enterprise Root CA
```

with:

- New private key
- SHA256 cryptography

### Configuration

Enabled:

```text
Certificate Services Client Auto-Enrollment
```

through Group Policy.

### Outcome

- Certificates deploy automatically to domain computers.
- Simplified certificate management.
- Improved trust relationships for internal services.

---

## RDP Certificate Trust Troubleshooting

### Issue

Remote Desktop connections generated certificate trust warnings when connecting to DC01.

### Root Cause

The management workstation did not trust the certificate being presented by the server.

### Resolution

Exported the certificate from DC01 and imported it into the trusted certificate store on the management workstation.

### Result

Successfully established Remote Desktop connections without certificate trust warnings.

---

## File Services

### Shared Resources

Created shared folders hosted on DC01.

### Security Groups

Created:

```text
Fileshare_RW
```

to manage access using security group-based permissions.

### Outcome

Implemented a more scalable permission model by assigning permissions to groups rather than individual users.

---

## Group Policy Administration

Group Policy became one of the primary areas of focus during this project.

### Control Panel Restriction GPO

Created a User Configuration GPO to restrict Control Panel access.

Applied security filtering to target only:

```text
Corp Users
```

### Outcome

Standard users were prevented from accessing Control Panel while administrative users retained access.

---

### Drive Mapping GPO

### Objective

Automatically deploy shared storage to domain users.

### Implementation

Used:

```text
Group Policy Preferences
→ Drive Maps
```

to deploy a mapped drive.

Mapped:

```text
S:
```

to a shared resource on DC01.

### Outcome

Domain users automatically received the mapped drive upon sign-in.

---

## Wallpaper Deployment Project

This became one of the most valuable troubleshooting exercises within the lab.

### Objective

Deploy a corporate wallpaper to users located in:

```text
Administration
Corp Users
```

OUs.

### Initial Design

Wallpaper stored on:

```text
\\dc01\shared\wallpaper\josue.lab.wallpaper.jpg
```

Configured through a User Configuration Group Policy.

---

### Issue 1: Wallpaper Not Applying

#### Symptoms

Wallpaper failed to appear after policy deployment.

#### Resolution

Ran:

```cmd
gpupdate /force
```

on both:

- DC01
- Windows 11 Client

#### Lesson Learned

Group Policy processing is not always immediate and should be validated during testing.

---

### Issue 2: Wallpaper Disappeared After User Switching

#### Symptoms

Wallpaper applied correctly when using:

```text
jjarquin-admin
```

but disappeared after logging into:

```text
jjarquin
```

#### Investigation

Discovered the policy relied on a network share accessed through a UNC path.

Tailscale connectivity and DNS resolution impacted access to the wallpaper source during user logon.

#### Root Cause

The wallpaper depended on:

```text
\\dc01\shared\wallpaper
```

being accessible during logon.

If DNS resolution or share access failed, the wallpaper could not be applied.

---

### Solution

Implemented Group Policy Preferences.

### Folder Creation

Automatically created:

```text
C:\Deploy\Wallpaper
```

on client computers.

### File Copy

Copied wallpaper from:

```text
\\dc01\shared\wallpaper
```

to:

```text
C:\Deploy\Wallpaper
```

### Updated Policy

Modified the wallpaper GPO to use:

```text
C:\Deploy\Wallpaper\josue.lab.wallpaper.jpg
```

instead of the network share.

### Result

- Reliable wallpaper deployment.
- Reduced dependency on network availability.
- Improved understanding of Group Policy Preferences.

---

## Secure Remote Administration with Tailscale

Installed Tailscale on:

- Windows 11 client VM 
- DC01 VM

### Benefits

- Secure remote access
- No port forwarding required
- Remote administration from external networks
- Reduced attack surface

### Administrative Tasks Performed

- Active Directory Administration
- DNS Administration
- Group Policy Management
- Remote Desktop Access

---

## Split DNS Troubleshooting

### Issue

Remote access only worked when the Windows client was manually configured to use DC01 as its preferred DNS server.

This created issues with internet access.

### Investigation

Determined that:

- Domain resources required DC01 DNS.
- Internet traffic required standard DNS resolution.

### Resolution

Implemented:

```text
Tailscale Split DNS
```

Configured:

```text
josue.lab
```

to resolve through DC01 while allowing internet traffic to continue using standard DNS services.

### Result

- Domain name resolution functional.
- Internet connectivity maintained.
- Reliable remote administration through Tailscale.

---

## DHCP and Hyper-V Network Expansion

### Objective

Simplify onboarding future virtual machines into the Active Directory environment.

### Design

Created a dedicated internal Hyper-V network.

Added a second virtual network adapter to DC01 and configured an isolated lab subnet.

Configured DHCP to automatically provide:

- IP Address
- Subnet Mask
- DNS Server
- Domain Configuration

### Goal

Allow future virtual machines to:

- Automatically receive network settings.
- Resolve Active Directory resources.
- Join the domain without manual DNS configuration.

**Validation and testing currently in progress.**

---

## Key Skills Demonstrated

### Windows Server Administration

- Active Directory Domain Services
- DNS
- DHCP
- Active Directory Certificate Services
- Group Policy
- File Services

### Networking

- DNS Troubleshooting
- DNS Forwarders
- Split DNS
- DHCP Configuration
- Hyper-V Networking

### Security

- Enterprise PKI
- Certificate Management
- Administrative Account Separation
- Secure Remote Access

### Troubleshooting

- DNS Resolution Issues
- RDP Certificate Trust Problems
- Group Policy Deployment Issues
- UNC Path Dependencies
- Split DNS Configuration
- Hyper-V Network Design

---

## Future Improvements

- Complete DHCP validation testing
- Configure NAT routing for lab network
- Build a Windows 11 Gold Image
- PowerShell automation
- Deploy a second Domain Controller
- Active Directory replication testing
- Backup and recovery procedures
- Microsoft Entra ID
- Azure Administration (AZ-104)
