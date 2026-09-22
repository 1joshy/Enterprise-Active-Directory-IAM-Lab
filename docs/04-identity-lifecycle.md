# Day 4 — Identity Lifecycle, Group Policy & Access Review

## Objectives

- Simulate an employee role change using Active Directory group membership
- Perform an employee offboarding workflow
- Apply centralized workstation security settings through Group Policy
- Conduct an access review and remediate excessive privilege
- Use PowerShell to audit users and group memberships

---

## Mover Scenario — Finance to Sales

To simulate an employee changing roles within Northstar Technologies, I transferred **Sarah Chen (`schen`)** from Finance to Sales.

Before the change, Sarah received Finance access through:

```text
schen
  |
GG-Finance-Users
  |
DL-Finance-Share-RW
  |
Finance Share
```

To update her access, I:

1. Removed `schen` from `GG-Finance-Users`
2. Added `schen` to `GG-Sales-Users`
3. Moved her Active Directory user object from the Finance OU to the Sales OU
4. Verified the new group membership

This changed Sarah's authorization without modifying the file-share ACLs themselves.

---

## Validating the Role Change

Before signing Sarah out of `IAM-WIN11`, her existing session continued to reflect the previous access state.

After establishing a fresh logon session, I tested both departmental shares again.

| Resource | Result |

| Finance | Denied |
| Sales | Allowed |

Sarah successfully created a test file in the Sales share while her previous Finance access was denied.

This demonstrated an important aspect of Active Directory authorization: changes to group membership may require a fresh authentication session before the user's access token fully reflects the new memberships.

It also demonstrated the advantage of the AGDLP design. The resource permissions did not need to be rewritten when Sarah changed departments—only her role-group membership changed.

---

## Leaver Scenario — Employee Offboarding

Next, I simulated the offboarding of **Daniel Kim (`dkim`)**, an HR employee.

Before offboarding, I confirmed that Daniel could access the HR share while being denied access to unrelated departmental resources.

The offboarding process included:

```text
Disable user account
        |
Remove HR role membership
        |
Move identity to Disabled Users OU
        |
Validate authentication is blocked
```

Using PowerShell, I disabled Daniel's Active Directory account and removed him from:

`GG-HR-Users`

I then created a dedicated:

```text
Northstar
└── Users
    └── Disabled Users
```

OU and moved Daniel's account into it.

The account was retained rather than deleted, preserving the identity for administrative and audit purposes.

---

## Validating Offboarding

After the account was disabled, I attempted a fresh authentication to `IAM-WIN11` using Daniel's domain credentials.

Windows rejected the login and displayed:

> Your account has been disabled. Please see your system administrator.

This confirmed that the offboarding action prevented the identity from establishing a new domain session.

The scenario demonstrated a basic leaver workflow:

- Revoke the ability to authenticate
- Remove unnecessary access
- Retain the identity for administrative purposes
- Validate that access is no longer possible

---

## Centralized Workstation Security with Group Policy

I next implemented a Group Policy Object to demonstrate centralized workstation administration.

The GPO was named:

`Northstar - Workstation Security Policy`

It was linked specifically to:

```text
Northstar
└── Computers
    └── Workstations
```

This allowed workstation settings to be applied without targeting servers or the Domain Controller.

The policy configured:

- **Machine inactivity limit:** 600 seconds
- **Interactive logon title:** `Northstar Technologies`
- **Interactive logon security notice**

The notice informed users that the system was restricted to authorized Northstar Technologies users and that activity could be monitored for security and administrative purposes.

---

## Validating Group Policy

On `IAM-WIN11`, I refreshed Group Policy with:

```cmd
gpupdate /force
```

I then verified the computer-side policy using:

```cmd
gpresult /scope computer /r
```

`Northstar - Workstation Security Policy` appeared under the applied Group Policy Objects.

After restarting the workstation, the Northstar Technologies security notice appeared before login.

This provided both command-line and visible confirmation that the centrally configured policy had reached the domain workstation.

---

## Access Review — Excessive HR Privilege

The final IAM scenario simulated an access-control mistake.

**Jacob Turner (`jturner`)** was a Sales employee whose legitimate access should have been limited to Sales resources.

For the exercise, Jacob was intentionally added to:

`GG-HR-Users`

Because of the existing AGDLP structure, this created the following authorization path:

```text
jturner
  |
GG-HR-Users
  |
DL-HR-Share-RW
  |
HR Share
```

From `IAM-WIN11`, I confirmed that Jacob could now access both:

- Sales — legitimate access
- Human Resources — excessive access

This demonstrated how a single incorrect group membership can propagate into unintended resource access.

---

## Investigating and Remediating the Finding

I reviewed the membership of the HR role using PowerShell and identified Jacob as an unexpected member.

I also inspected his Active Directory identity and group memberships to verify that his organizational role belonged to Sales.

The excessive entitlement was remediated by removing:

`jturner`

from:

`GG-HR-Users`

His legitimate Sales membership was left intact.

After establishing a fresh logon session, access was tested again:

| Resource | Before Remediation | After Remediation |

| Sales | Allowed | Allowed |
| Human Resources | Allowed | Denied |

The goal of remediation was not to remove all of Jacob's access, but to remove the **unauthorized entitlement while preserving legitimate business access**.

---

## PowerShell Identity & Access Audit

To finish the technical portion of the lab, I used PowerShell to review the final state of the Northstar environment.

I queried users within the Northstar OUs and reviewed whether their accounts were enabled or disabled.

I also audited membership across the departmental Global Security Groups:

```powershell
$groups = @(
    "GG-IT-Users",
    "GG-HR-Users",
    "GG-Finance-Users",
    "GG-Sales-Users"
)

foreach ($group in $groups) {
    Write-Host "`n=== $group ==="
    Get-ADGroupMember $group |
        Select-Object Name,SamAccountName
}
```

The final group memberships reflected the lifecycle changes performed during the lab:

```text
IT
├── Ethan Brooks
└── Maya Patel

Human Resources
└── Olivia Carter

Finance
└── Marcus Reed

Sales
├── Emily Rodriguez
├── Jacob Turner
└── Sarah Chen
```

Daniel was no longer an active HR member following offboarding, Sarah was now assigned to Sales following her role change, and Jacob's unintended HR entitlement had been removed.

I also exported Active Directory user information to CSV to demonstrate how directory data could be used for administrative reporting and access reviews.

---

## Key IAM Concepts

Day 4 brought together the major concepts from the entire lab:

- Joiner, Mover, Leaver (JML) lifecycle management
- Role-Based Access Control
- Least privilege
- Entitlement management
- Access reviews
- Excessive privilege remediation
- Active Directory security groups
- Group Policy
- Windows authentication tokens
- PowerShell administration and auditing

Rather than simply creating identities and permissions, these scenarios demonstrated how access can be **changed, revoked, audited, and validated throughout an identity's lifecycle**.

---

## Day 4 Result

By the end of Day 4, the Northstar environment supported several realistic IAM administration workflows.

I successfully:

- Transferred an employee between departments while updating access
- Offboarded an employee and blocked future authentication
- Applied and validated centralized workstation security policy
- Identified and remediated excessive resource access
- Preserved legitimate access during remediation
- Audited identities and group memberships using PowerShell

This completed the technical build of the **Enterprise Active Directory & IAM Administration Lab** and left the environment in a validated least-privilege state.