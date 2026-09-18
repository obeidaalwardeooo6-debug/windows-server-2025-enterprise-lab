# Windows Server 2025 Enterprise Lab

A hands-on Windows Server 2025 infrastructure project demonstrating the design, configuration, testing, and troubleshooting of Active Directory Domain Services, DNS, DHCP, Group Policy, shared network resources, and Windows 11 domain integration.

---

## Project Overview

This project was created as a practical Windows Server infrastructure lab for the fictional company **KubenData**.

The goal was to build a small enterprise-style Windows environment where users, computers, network services, and security policies are centrally managed from a Windows Server 2025 Domain Controller.

The lab includes:

- Windows Server 2025
- Active Directory Domain Services
- Domain Controller
- DNS
- DHCP
- Organizational Units
- Domain users
- Group Policy Objects
- Shared network folders
- Windows 11 Pro domain client
- Testing and troubleshooting

---

## Lab Environment

| Component | Configuration |
|---|---|
| Server hostname | DC01 |
| Server OS | Windows Server 2025 |
| Server IPv4 | 192.168.0.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.0.1 |
| Active Directory Domain | kubendata.local |
| Client hostname | CLIENT-PC2 |
| Client OS | Windows 11 Pro |
| DHCP Scope | 192.168.0.100 - 192.168.0.200 |
| DNS Server | 192.168.0.10 |

---

## Network Architecture

```text
                    Internet
                       |
                       |
              Router / Gateway
                 192.168.0.1
                       |
            -----------------------
            |                     |
            |                     |
          DC01               CLIENT-PC2
      192.168.0.10          Windows 11 Pro
            |
            |
    ---------------------
    |        |          |
   AD DS    DNS        DHCP
    |
    |
Group Policy
```

`DC01` provides the main infrastructure services for the domain.

The router remains the network gateway, while DHCP and DNS services are provided by the Windows Server environment.

---

# Active Directory Domain Services

## Domain Configuration

Active Directory Domain Services was installed on Windows Server 2025.

The server was promoted to a Domain Controller for:

```text
kubendata.local
```

The Domain Controller hostname is:

```text
DC01
```

The server uses a static IP address:

```text
192.168.0.10
```

---

## Organizational Units

Three Organizational Units were created to represent the departments in KubenData:

```text
OU-IT
OU-HR
OU-SALG
```

The OUs are used to organize users and control where Group Policy Objects apply.

---

## Domain Users

### IT Department

```text
ola.nordmann
kari.hansen
per.olsen
```

### HR Department

```text
anne.larsen
nina.berg
hans.jensen
```

### Sales Department

```text
liv.andersen
erik.holm
tom.nilsen
```

Each user was placed in the correct Organizational Unit.

---

# Windows 11 Domain Client

A Windows 11 Pro virtual machine was configured as the domain client.

Hostname:

```text
CLIENT-PC2
```

The client was successfully joined to:

```text
kubendata.local
```

Domain users were then able to sign in using domain credentials.

Examples:

```text
KUBENDATA\ola.nordmann
KUBENDATA\anne.larsen
KUBENDATA\liv.andersen
```

A local administrator account was also retained for administrative and troubleshooting tasks.

---

# DNS Configuration

DNS is installed on `DC01` and integrated with the Active Directory environment.

Domain clients use:

```text
192.168.0.10
```

as their DNS server.

This allows clients to locate Active Directory services and resolve the internal domain.

---

## Internal DNS Test

The domain was tested using:

```powershell
nslookup kubendata.local
```

The result confirmed:

```text
kubendata.local
192.168.0.10
```

This verified that the domain name resolves correctly to the Domain Controller.

---

## External DNS Test

External DNS resolution was tested using:

```powershell
nslookup google.com
```

The lookup returned valid IP addresses, confirming that external DNS resolution was working.

---

## DNS Forwarding

The DNS server was configured to forward external DNS requests that it cannot resolve locally.

The configured forwarder is:

```text
192.168.0.1
```

The DNS flow is therefore:

```text
CLIENT-PC2
    |
    | DNS request
    v
DC01
192.168.0.10
    |
    | External request
    v
192.168.0.1
    |
    v
Internet DNS
```

---

## DNS Diagnostics

DNS functionality on the Domain Controller was also tested using:

```powershell
dcdiag /test:dns
```

The DNS diagnostic test completed successfully.

---

# DHCP Configuration

DHCP Server was installed on `DC01`.

The DHCP server was authorized in Active Directory before being used to distribute network configuration.

---

## DHCP Scope

The DHCP scope was configured as:

```text
Start IP:       192.168.0.100
End IP:         192.168.0.200
Subnet Mask:    255.255.255.0
```

This means domain clients can automatically receive an IP address from this range.

---

## DHCP Options

The following DHCP options were configured:

```text
003 Router
192.168.0.1
```

```text
006 DNS Servers
192.168.0.10
```

```text
015 DNS Domain Name
kubendata.local
```

This ensures that DHCP clients receive the correct gateway, DNS server, and domain information automatically.

---

## DHCP Migration from Router

Before activating DHCP on `DC01`, the DHCP service on the lab router was disabled.

This was done to avoid having two DHCP servers distributing addresses on the same network.

After the router DHCP service was disabled, the DHCP scope on `DC01` was activated.

The router continued to operate as:

```text
Default Gateway: 192.168.0.1
```

while `DC01` became responsible for assigning network configuration.

---

## Client DHCP Test

The Windows 11 client was configured to obtain its IP address and DNS server automatically.

The following commands were used:

```powershell
ipconfig /release
ipconfig /renew
ipconfig /all
```

The client successfully received:

```text
IPv4 Address:     192.168.0.102
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.0.1
DHCP Server:      192.168.0.10
DNS Server:       192.168.0.10
```

This confirmed that DHCP was operating correctly from `DC01`.

---

## DHCP Authorization Test

DHCP authorization in Active Directory was verified using:

```powershell
Get-DhcpServerInDC
```

The result confirmed:

```text
192.168.0.10
dc01.kubendata.local
```

---

## DHCP Service Test

The DHCP Server service was checked using:

```powershell
Get-Service DHCPServer
```

The service status was:

```text
Running
```

This confirmed that the DHCP service was active.

---

# Group Policy

Separate Group Policy Objects were created for the three departments.

```text
GPO-IT
GPO-HR
GPO-SALG
```

The policies were linked as follows:

```text
GPO-IT
   |
   v
OU-IT
```

```text
GPO-HR
   |
   v
OU-HR
```

```text
GPO-SALG
   |
   v
OU-SALG
```

This allows different departments to receive different Windows configurations.

---

## IT Group Policy

`GPO-IT` was linked to:

```text
OU-IT
```

The IT policy included user-based configuration such as:

- Restricted access to Control Panel / Settings
- Start menu configuration
- Automatic program launch at user logon

The policy was tested using:

```text
ola.nordmann
```

A program configured through Group Policy successfully started after user logon.

---

## HR Group Policy

`GPO-HR` was linked to:

```text
OU-HR
```

The HR configuration included:

- Restricted access to Windows settings
- Restrictions on changing desktop configuration
- Department desktop wallpaper

The policy was tested using:

```text
anne.larsen
```

The configured HR wallpaper appeared successfully on the client.

---

## Sales Group Policy

`GPO-SALG` was linked to:

```text
OU-SALG
```

The Sales configuration included:

- Windows restrictions
- Desktop configuration
- Sales department wallpaper
- Shared network drive mapping

The policy was tested using:

```text
liv.andersen
```

The Sales wallpaper appeared correctly.

---

# Shared Network Resource

A shared folder was created on the server for the Sales department.

Server path:

```text
C:\SalgFelles
```

Network path:

```text
\\DC01\SalgFelles
```

The shared folder was automatically mapped using Group Policy Preferences as:

```text
S:
```

The Sales user successfully accessed the mapped network drive from `CLIENT-PC2`.

This verified both:

- Group Policy Preferences
- Access to a server-hosted shared resource

---

# Group Policy Validation

Group Policy was refreshed manually using:

```powershell
gpupdate /force
```

The command confirmed that both computer and user policy processing completed successfully.

Applied policies were checked using:

```powershell
gpresult /r
```

Tests were performed with users from different departments.

### IT

```text
ola.nordmann
Applied GPO: GPO-IT
```

### HR

```text
anne.larsen
Applied GPO: GPO-HR
```

### Sales

```text
liv.andersen
Applied GPO: GPO-SALG
```

This confirmed that the correct policies were applied according to Organizational Unit membership.

---

# Domain Controller Validation

Domain Controller health was tested using:

```powershell
dcdiag
```

Core Active Directory tests completed successfully.

Additional DNS testing was performed using:

```powershell
dcdiag /test:dns
```

DNS diagnostics passed successfully.

During broader diagnostics, some Windows System Event Log warnings were observed.

The core Active Directory, DNS, replication, services, and domain functionality remained operational.

---

# SPN Troubleshooting

As part of troubleshooting, duplicate Service Principal Names were checked using:

```powershell
setspn -X
```

The result showed:

```text
0 groups of duplicate SPNs
```

This confirmed that no duplicate SPN entries were detected in the domain.

---

# Testing Summary

The following functionality was successfully tested:

- Windows Server 2025 configuration
- Static server IP configuration
- Active Directory Domain Services
- Domain Controller functionality
- Organizational Units
- Domain users
- Windows 11 domain join
- Internal DNS resolution
- External DNS resolution
- DNS diagnostics
- DHCP Server installation
- DHCP authorization
- DHCP scope configuration
- DHCP client lease
- DHCP service operation
- Group Policy linking
- Group Policy processing
- Department-specific policies
- Desktop wallpaper policies
- Shared drive mapping
- Domain user logon
- PowerShell administrative validation

---

# Troubleshooting Experience

The project also involved troubleshooting several real configuration issues.

Examples included:

- Verifying that domain clients use the Domain Controller as DNS
- Differentiating between local administrator and domain accounts
- Testing Group Policy application with `gpresult`
- Refreshing policies using `gpupdate`
- Validating DHCP authorization
- Avoiding multiple active DHCP servers on the same network
- Testing DNS after DHCP configuration
- Investigating Domain Controller diagnostic warnings
- Checking duplicate SPNs

These troubleshooting steps helped validate both the infrastructure and the administrative workflow.

---

# Commands Used

Some of the main commands used during the project:

```powershell
hostname
ipconfig /all
ipconfig /release
ipconfig /renew
ping 192.168.0.10
nslookup kubendata.local
nslookup google.com
gpupdate /force
gpresult /r
dcdiag
dcdiag /test:dns
Get-DhcpServerInDC
Get-Service DHCPServer
setspn -X
```

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Windows Server 2025 administration
- Active Directory Domain Services
- Domain Controller configuration
- Active Directory Users and Computers
- Organizational Unit design
- Domain user management
- DNS configuration
- DNS troubleshooting
- DHCP installation and configuration
- DHCP scopes and options
- DHCP authorization in Active Directory
- Group Policy Management
- Group Policy Preferences
- Windows 11 domain integration
- Shared network resources
- PowerShell administration
- Network troubleshooting
- Infrastructure validation
- Technical documentation

---

# Repository Structure

The repository documentation will be organized as:

```text
windows-server-2025-enterprise-lab/
|
|-- README.md
|
|-- docs/
|   |-- 01-active-directory.md
|   |-- 02-dns.md
|   |-- 03-dhcp.md
|   `-- 04-group-policy.md
|
`-- screenshots/
    |-- ad/
    |-- dns/
    |-- dhcp/
    `-- gpo/
```

Detailed screenshots and configuration evidence will be stored in the relevant folders.

---

# Security

No passwords, authentication secrets, recovery keys, private credentials, or other sensitive authentication information are included in this repository.

The IP addresses and domain names documented here belong to the isolated training lab environment.

---

# Project Status

**Completed**

The Windows Server 2025 infrastructure was successfully configured and tested with:

```text
Active Directory
DNS
DHCP
Group Policy
Windows 11 Domain Client
```

The project demonstrates a complete small-scale Windows domain environment from server configuration through client validation and troubleshooting.
