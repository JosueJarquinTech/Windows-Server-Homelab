# Windows Server Home Lab

A hands-on Windows Server infrastructure project built to develop System Administration skills through the deployment, administration, and troubleshooting of Active Directory, DNS, PKI, Group Policy, Hyper-V networking, and secure remote administration.

## Skills Demonstrated

- Windows Server Administration
- Active Directory
- DNS
- DHCP
- PKI / Active Directory Certificate Services
- Group Policy
- Group Policy Preferences
- Hyper-V
- Tailscale / Split DNS
- Network Troubleshooting
- Remote Administration
- File Services
- NAT

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
- DHCP 
- DNS Forwarders
- Split DNS
- NAT

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
Internet
    │
    ▼
Home Router
    │
    ├─────────────────────────────────────────┐
    │                                         │
    ▼                                         ▼
Dell Precision 5560                      MacBook M2
Windows 11 Pro Host                      macOS Host
│                                         │
└── Hyper-V                               └── Parallels Desktop
     │                                         │
     └── DC01                                 └── Windows 11 Pro Client
          Windows Server 2022                      └── Tailscale
          │
          ├── External NIC
          │     192.168.x.x
          │
          ├── Internal NIC
          │     10.0.0.10
          │
          ├── Active Directory Domain Services
          ├── DNS
          ├── DHCP
          ├── NAT
          ├── Active Directory Certificate Services
          ├── File Services
          └── Tailscale
                │
                ▼
          Hyper-V Internal Switch
                │
                ├── Windows 11 Test VM 01
                ├── Windows 11 Test VM 02
                └── Future Domain Clients

Domain:
josue.lab
```

### Architecture Overview

The primary lab environment is hosted on a Dell Precision 5560 running Windows 11 Pro and Hyper-V.

DC01 is a Windows Server 2022 virtual machine that provides:

- Active Directory Domain Services
- DNS
- DHCP
- NAT
- Active Directory Certificate Services (AD CS)
- File Services
- Secure remote access through Tailscale

The server is configured with two network adapters:

```text
External NIC
192.168.x.x
```

Connected to the home network and internet.

```text
Internal NIC
10.0.0.10
```

Connected to a dedicated Hyper-V internal network used for domain-joined lab systems.

A separate Windows 11 Pro virtual machine hosted in Parallels Desktop on a MacBook M2 is used for remote administration and testing. Tailscale and Split DNS allow secure administration of lab resources from external networks.

---

## Server Network Configuration

### Static IP Configuration

#### Initial State

During the initial deployment phase, DC01 was configured to receive its IP address through DHCP.

#### Problem

Domain Controllers provide critical infrastructure services such as:

- Active Directory
- DNS
- DHCP
- Group Policy

These services depend on clients being able to reliably locate the server.

Using a dynamically assigned IP address can create operational issues if the address changes due to:

- DHCP lease expiration
- Network reconfiguration
- Router configuration changes

#### Resolution

Configured a static IP address on the external network adapter of DC01.

```text
External Interface
IP Address: 192.168.x.x/22
```

The internal lab network interface remained configured as:

```text
Internal Interface
IP Address: 10.x.x.x/24
```

#### Outcome

- Consistent access to domain services
- Stable DNS functionality
- Reliable network configuration
- Improved infrastructure stability

#### Lessons Learned

Infrastructure servers should use static IP addresses whenever possible.

Active Directory and DNS services are easier to manage and troubleshoot when infrastructure systems maintain predictable network addresses.

## DNS Administration and Troubleshooting

### DNS Configuration

Verified the following Active Directory integrated DNS zones:

```text
josue.lab
_msdcs.josue.lab
```

### DNS Forwarders

Configured DNS forwarders to provide external name resolution for domain clients.

Forwarders configured:

```text

8.8.8.8 (Google DNS)

1.1.1.1 (Cloudflare DNS)

```

This allowed domain clients to continue resolving Active Directory resources while also accessing internet services.

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

Implement an internal Public Key Infrastructure (PKI) to support certificate-based services and automated certificate deployment throughout the lab environment.

### Deployment

Installed:

```text
Active Directory Certificate Services
```

Configured:

```text
Enterprise Root CA
```

using:

- New private key
- SHA256 cryptography

Created the Certification Authority:

```text
josue-DC01-CA
```

This established a trusted internal Certificate Authority capable of issuing certificates to domain-joined systems.

---

### Certificate Template Deployment

Created a custom computer certificate template:

```text
Computer AutoEnroll
```

Configured the following permissions for:

```text
Domain Computers
```

- Read
- Enroll
- AutoEnroll

Published the template through:

```text
josue-DC01-CA
```

to allow automatic certificate issuance.

---

### Automatic Certificate Enrollment

Created and linked a Group Policy Object named:

```text
PKI - Certificate Auto Enrollment
```

Configured:

```text
Certificate Services Client - Auto-Enrollment
```

to automatically enroll eligible domain-joined computers.

This enables certificate deployment without requiring manual administrator intervention.

---

### Validation

Validated the deployment using newly provisioned Windows 11 domain clients connected to the Hyper-V internal network.

Verified:

- Group Policy processing
- Automatic certificate enrollment
- Computer certificate issuance
- Enterprise Root CA functionality
- Successful certificate requests from domain-joined computers

Issued certificates were successfully generated using the:

```text
Computer AutoEnroll
```

template and issued by:

```text
josue-DC01-CA
```

---

### Outcome

- Automated certificate deployment
- Reduced certificate administration overhead
- Improved understanding of enterprise PKI concepts
- Successfully implemented certificate lifecycle automation through Active Directory and Group Policy
- Established a foundation for certificate-based authentication and secure internal services

### Lessons Learned

- Enterprise PKI becomes significantly more powerful when integrated with Active Directory.
- Certificate templates control who can request certificates and under what conditions.
- Auto-enrollment allows domain-joined computers to automatically obtain certificates without manual installation.
- Group Policy can automate certificate enrollment and renewal throughout the environment.
- Certificate Services provides the foundation for many enterprise technologies, including secure authentication, VPNs, Remote Desktop Services, and web services.

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

---
## DHCP, NAT, and Automated Client Provisioning

### Objective

Design and validate an internal Hyper-V network that allows new virtual machines to automatically receive network configuration, discover Active Directory resources, join the domain, obtain certificates, and access the internet without manual configuration.

### Internal Network Design

To simplify future VM deployments, a dedicated internal Hyper-V network was created and connected to a second network adapter on DC01.

#### Network Configuration

```text
DC01

External NIC
IP Address: 192.168.x.x

Internal NIC
IP Address: 10.0.x.x
```

The external interface provides access to the home network and internet resources, while the internal interface hosts Active Directory infrastructure services for lab systems.

---

### DHCP Deployment

Configured a DHCP scope on DC01 to automatically provide:

- IP Address
- Subnet Mask
- DNS Server
- Default Gateway

This allows newly deployed virtual machines to obtain the required network configuration without manual intervention.

#### Benefits

- Simplified client deployment
- Consistent DNS configuration
- Reduced manual administration
- Improved scalability for future lab expansion

---

### NAT Configuration and Internet Access

#### Issue

During initial testing, domain connectivity functioned correctly, but clients connected to the internal network were unable to access internet resources.

#### Investigation

Verified:

- DHCP leases were being assigned successfully.
- DNS resolution was functioning correctly.
- Domain Controller discovery was operational.
- Active Directory domain joins were successful.

Further investigation determined that Network Address Translation (NAT) had not been configured for the internal subnet.

#### Resolution

Configured NAT using PowerShell:

```powershell
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix "10.0.0.0/24"
```

Configured DHCP Option 003 (Router) to provide:

```text
10.0.0.10
```

as the default gateway for internal clients.

#### Result

- Internet connectivity restored for internal clients.
- Domain connectivity maintained.
- Centralized routing through DC01.
- Successful end-to-end network functionality.

---

### Windows 11 Test Client Validation

To validate the new network design, a Windows 11 Pro virtual machine was deployed on the internal Hyper-V network.

#### Virtual Machine Specifications

```text
2 vCPU
4 GB RAM
64 GB Dynamic Disk
```

#### Validation Results

Successfully verified that the client automatically:

- Obtained a DHCP lease
- Received DNS configuration
- Resolved domain resources
- Located the Domain Controller
- Joined the Active Directory domain
- Received a computer certificate through auto-enrollment
- Accessed internet resources through NAT

No manual network configuration was required.


---

## Key Lessons Learned

- DHCP dramatically simplifies onboarding of new systems.
- DNS is critical for Active Directory authentication and service discovery.
- A correctly configured default gateway is required for communication outside the local subnet.
- NAT is required to provide internet access to isolated internal networks.
- Certificate auto-enrollment extends the value of Active Directory Certificate Services by automating certificate deployment and renewal.
- Hyper-V networking can be used to simulate enterprise network segmentation and client provisioning workflows.

### Outcome

The final design allows a newly deployed virtual machine to automatically:

```text
Boot
↓
Receive DHCP Configuration
↓
Receive DNS Configuration
↓
Locate DC01
↓
Join Active Directory
↓
Receive Group Policy
↓
Auto-Enroll for a Certificate
↓
Access Internet Through NAT
```

This significantly reduced manual configuration requirements and created a repeatable process for onboarding future systems into the lab environment.
---

---

## Future Improvements

- Build a Windows 11 Gold Image
- PowerShell automation
- Deploy a second Domain Controller
- Active Directory replication testing
- Backup and recovery procedures
- Microsoft Entra ID
- Azure Administration (AZ-104)
