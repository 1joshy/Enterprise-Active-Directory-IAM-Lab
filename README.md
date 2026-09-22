# Enterprise Active Directory & IAM Administration Lab

A hands-on Identity and Access Management lab built around a fictional organization, **Northstar Technologies**, using Windows Server, Active Directory Domain Services, Group Policy, PowerShell, and a domain-joined Windows 11 workstation.

The project focuses on practical IAM administration including **role-based access control, AGDLP, least privilege, identity lifecycle management, Group Policy, access reviews, and entitlement remediation**.

---

## Architecture

The environment was built locally in **VMware Workstation Pro** and consists of:

| System | Purpose |
|---|---|
| `IAM-DC01` | Windows Server 2022 Domain Controller |
| `IAM-WIN11` | Windows 11 Pro domain workstation |
| `ad.northstar.com` | Active Directory domain |
| AD DS | Centralized identity and authentication |
| DNS | Active Directory service discovery |
| Group Policy | Centralized workstation security configuration |
| SMB File Shares | Departmental resources used for authorization testing |

---

## Project Goals

The goal of this project was to move beyond simply installing Active Directory and instead practice how identities and access are **provisioned, changed, revoked, audited, and validated** in a Windows domain environment.

The lab was designed around several core IAM concepts:

- Active Directory administration
- Role-Based Access Control (RBAC)
- AGDLP group nesting
- Least privilege
- Joiner, Mover, Leaver (JML) lifecycle management
- Authentication vs. authorization
- Group Policy
- SMB and NTFS permissions
- Access reviews and entitlement remediation
- PowerShell administration

---

## Active Directory Structure

Northstar Technologies was organized using departmental and system-specific Organizational Units.

```text
Northstar
├── Users
│   ├── IT
│   ├── Human Resources
│   ├── Finance
│   ├── Sales
│   └── Disabled Users
│
├── Computers
│   ├── Workstations
│   │   └── IAM-WIN11
│   └── Servers
│
└── Groups
```

Users were assigned to departmental **Global Security Groups** rather than receiving permissions directly.

---

## Access Control Model

Departmental resource access was implemented using the **AGDLP** model:

```text
Accounts
   |
Global Groups
   |
Domain Local Groups
   |
Permissions
```

Example:

```text
Finance Employee
      |
GG-Finance-Users
      |
DL-Finance-Share-RW
      |
Finance File Share
```

Global groups represent the user's organizational role, while Domain Local groups represent access to a specific resource.

Departmental file shares included:

```text
\\IAM-DC01\IT
\\IAM-DC01\HumanResources
\\IAM-DC01\Finance
\\IAM-DC01\Sales
```

Access was controlled through both **Share and NTFS permissions**.

Testing from `IAM-WIN11` verified that users could access resources associated with their role while unrelated departmental resources were denied.

---

## IAM Lifecycle Scenarios

### 1. Employee Role Change — Mover

**Sarah Chen (`schen`)** was transferred from Finance to Sales.

Her Finance role membership was removed and Sales membership was granted without changing the underlying resource ACLs.

After establishing a fresh logon session:

```text
Finance > DENIED
Sales   > ALLOWED
```

This demonstrated how role-based access can be updated as an employee's business responsibilities change.

[View Mover Scenario](docs/scenarios/01-mover-role-change.md)

---

### 2. Employee Offboarding — Leaver

**Daniel Kim (`dkim`)** was used for a simulated employee offboarding.

The workflow included:

```text
Disable Account
      |
Remove HR Entitlement
      |
Move to Disabled Users OU
      |
Validate Authentication Failure
```

A fresh authentication attempt from `IAM-WIN11` was rejected after the account was disabled.

[View Leaver Scenario](docs/scenarios/02-leaver-offboarding.md)

---

### 3. Excessive Privilege Access Review

**Jacob Turner (`jturner`)**, a Sales employee, was intentionally assigned an inappropriate HR group membership.

This produced an unintended authorization path:

```text
jturner
   |
GG-HR-Users
   |
DL-HR-Share-RW
   |
HR Share
```

The excessive entitlement was identified through an access review, traced through the AGDLP structure, and removed.

Validation after remediation confirmed:

| Resource | Before | After |

| Sales | Allowed | Allowed |
| Human Resources | Allowed | Denied |

The remediation restored least privilege while preserving Jacob's legitimate business access.

[View Access Review Scenario](docs/scenarios/03-access-review-remediation.md)

---

## Group Policy

A workstation security GPO was created:

`Northstar - Workstation Security Policy`

and linked to the Northstar **Workstations OU**.

The policy configured:

- 10-minute machine inactivity limit
- Northstar Technologies interactive logon title
- Authorized-use security notice

Policy deployment was validated on `IAM-WIN11` using:

```cmd
gpupdate /force
gpresult /scope computer /r
```

The security notice was also confirmed visually after restarting the workstation.

This demonstrated centralized security configuration of domain-managed endpoints.

---

## PowerShell Administration

PowerShell was used throughout the lab to perform and validate Active Directory administration tasks, including:

```powershell
Get-ADUser
Get-ADGroupMember
Get-ADPrincipalGroupMembership
Add-ADGroupMember
Remove-ADGroupMember
Disable-ADAccount
Move-ADObject
```

PowerShell was also used to review the final identity state and export Active Directory user information for administrative reporting.

---

## Troubleshooting

The project also included several troubleshooting scenarios encountered while building the environment.

### DNS Namespace

The original lab attempted to use:

`ad.northstar.example`

Active Directory promotion completed, but the expected DNS zones were not created and manual zone creation failed in the lab environment.

After validating AD/DNS components, I performed an A/B test using:

`test.contoso.com`

which successfully created a DNS zone.

The lab namespace was changed to:

`ad.northstar.com`

and the final environment operated normally.

This troubleshooting process helped isolate the namespace as the differentiating variable in this environment rather than assuming the DNS service itself was failing.

### Windows Client Edition

The initial workstation was deployed using Windows 11 Home.

During domain-join testing, I identified that Windows 11 Home does not provide traditional Active Directory domain-join functionality.

The workstation was rebuilt using **Windows 11 Pro**, after which the domain join completed successfully.

---

## Documentation

Detailed build notes are available in the lab journal:

| Day | Focus |
|---|---|
| [Day 1](docs/lab-journal/01-intro.md) | Active Directory infrastructure, DNS, and troubleshooting
| [Day 2](docs/lab-journal/02-active-directory-structure.md) | OU structure, identities, security groups, and domain workstation
| [Day 3](docs/lab-journal/03-AGDLP.md) | AGDLP, departmental file shares, RBAC, and access validation
| [Day 4](docs/lab-journal/04-identity-lifecycle.md) | Identity lifecycle, Group Policy, access review, and auditing

Additional scenario documentation:

- [Employee Role Change — Mover](docs/scenarios/01-mover-role-change.md)
- [Employee Offboarding — Leaver](docs/scenarios/02-leaver-offboarding.md)
- [Excessive Privilege Access Review](docs/scenarios/03-access-review-remediation.md)

---

## Skills Demonstrated

**Identity & Access Management**
- Active Directory Domain Services
- User and group administration
- RBAC and AGDLP
- Least-privilege access
- Identity lifecycle management
- Access reviews
- Entitlement remediation

**Windows Administration**
- Windows Server 2022
- Windows 11 domain membership
- Group Policy
- DNS
- SMB file sharing
- NTFS permissions

**Administration & Troubleshooting**
- PowerShell
- DNS/service discovery validation
- Group membership auditing
- Authentication and authorization testing
- Infrastructure troubleshooting
- Technical documentation

---

## Key Takeaways

This project demonstrated that IAM involves more than creating user accounts.

The completed environment provided hands-on experience managing access throughout an identity's lifecycle:

```text
Provision → Authorize → Validate → Modify → Audit → Revoke
```

By combining Active Directory, role-based group design, resource permissions, Group Policy, lifecycle scenarios, access reviews, and endpoint validation, the lab provided practical experience with the administrative and security responsibilities involved in managing identities in a Windows enterprise environment.