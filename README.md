# WrightTech Active Directory Lab

Enterprise-style Active Directory environment built from scratch to simulate a real-world Windows Server infrastructure deployment, including Domain Services, Group Policy, PowerShell automation, and security group management.

## Overview

This project simulates the IT infrastructure of a fictional company, **Wright Technologies**, built as Project 1 of a structured Cloud Security Engineer portfolio roadmap. The goal was to move beyond certification-style tutorials and build something that reflects how a real Systems Administrator or IAM Engineer would design and manage enterprise identity infrastructure.

## Environment

- **Hypervisor:** UTM (QEMU) on Apple Silicon, x86_64 emulation
- **Operating System:** Windows Server 2022 Standard (Desktop Experience)
- **Domain:** wrighttech.local
- **Domain Controller:** WrightTech-DC01
- **Network:** Static IP (192.168.100.10/24), isolated lab subnet
## OU Structure
Departments are separated by OU rather than a flat user list to support **Group Policy targeting** — different departments can receive different security policies, software deployments, or login scripts without affecting the rest of the domain. This mirrors how real organizations structure identity for scalability and delegated administration.

## Security Groups

Department-based Security Groups were created to manage access at the group level rather than per-user, following least-privilege and standard enterprise naming conventions:

- `SG-IT-Users`
- `SG-HR-Users`
- `SG-Finance-Users`
- `SG-Sales-Users`
- `SG-Executive-Users`

## User Provisioning (PowerShell Automation)

Rather than manually creating each user through the GUI, a PowerShell script (`Create-Users.ps1`) reads a CSV file (`users.csv`) and provisions users automatically into their correct department OU. This reflects real-world onboarding automation practices used by IT/Systems teams.

**Script logic:**
- Imports user data (First Name, Last Name, Department, Job Title) from CSV
- Dynamically builds the target OU path based on the Department field
- Generates SAM Account Name and UPN from name data
- Creates the account with a temporary password and forces a password reset at first login
- ## Group Policy

A GPO (`GPO-Password-Lockout-Policy`) was created and linked to the Departments OU to enforce baseline password and account lockout security.

**Password Policy:**
- Enforce password history: 24 passwords remembered
- Maximum password age: 90 days
- Minimum password age: 1 day
- Minimum password length: 12 characters
- Complexity requirements: Enabled
- Reversible encryption: Disabled

**Account Lockout Policy:**
- Lockout duration: 30 minutes
- Lockout threshold: 5 invalid logon attempts
- Reset lockout counter after: 30 minutes

## Troubleshooting Scenarios

### Scenario 1: Account Lockout

**Objective:** Verify the Account Lockout Policy functions as configured.

**Method:** Used `runas /user:wrighttech\jcarter cmd` with intentionally incorrect passwords, 5 times, to simulate a real failed-login lockout event.

**Result:** Account locked after the 5th failed attempt, confirmed via the "Unlock account" checkbox in Active Directory Users and Computers (Account tab).

**Resolution:** Checked "Unlock account" and confirmed restored access — validating the GPO was correctly applied and enforced.

### Scenario 2: DNS Resolution Testing

**Objective:** Simulate a DNS misconfiguration by pointing the Domain Controller's Preferred DNS server at a public DNS server (8.8.8.8) instead of itself.

**Result:** Unexpected — `nslookup` still resolved `wrighttech.local` successfully via IPv6 loopback (`::1`), even after flushing the DNS cache (`ipconfig /flushdns`).

**Finding:** Because this machine hosts the DNS Server role itself, Windows' DNS client can still resolve queries locally regardless of the configured external Preferred DNS value. This behavior is unique to DNS servers — a regular domain-joined client would not have this fallback and would fail cleanly under the same misconfiguration.

**Takeaway:** DNS failure testing must be performed from a client machine, not the DNS server itself, to produce a realistic result. This is planned as a follow-up test once a domain-joined client is added to the environment (Project 2).
## Skills Demonstrated

- Active Directory Domain Services (AD DS) deployment and forest/domain design
- Organizational Unit structure design for scalable Group Policy targeting
- Security Group management following least-privilege principles
- PowerShell scripting for automated, CSV-driven user provisioning
- Group Policy Object creation and configuration (password/lockout policy)
- DNS troubleshooting and diagnostic methodology
- Documentation of real, reproducible troubleshooting scenarios

## Resume Bullet Points

- Deployed and configured an Active Directory Domain Services environment on Windows Server 2022, including forest/domain design, OU structure, and Group Policy for a simulated enterprise network
- Automated user provisioning for 9 accounts across 5 departments using a CSV-driven PowerShell script, reducing manual account creation time and enforcing consistent onboarding standards
- Configured and validated enterprise password and account lockout policies via Group Policy, confirmed through simulated failed-login testing
- Diagnosed and documented DNS resolution behavior unique to Domain Controllers, identifying a key distinction between DC-side and client-side DNS troubleshooting

## What's Next

This is Project 1 of a 12-project Cloud Security Engineer portfolio roadmap. Project 2 will extend this environment with **Hybrid Identity** — connecting this on-premises AD forest to Microsoft Entra ID via Entra Connect, enabling MFA, Conditional Access, and hybrid domain join for a client machine.

---

*Built as part of a structured cybersecurity/cloud engineering portfolio. See full roadmap and other projects at [repository links to come].*
