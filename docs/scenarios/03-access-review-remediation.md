# Scenario 03 — Excessive Privilege Access Review

## Scenario

**User:** Jacob Turner (`jturner`)  
**Department:** Sales  
**Finding:** Unauthorized Human Resources access

A simulated access-control error was introduced by assigning Sales employee Jacob Turner membership in the HR role group.

The objective was to identify the excessive entitlement, determine how it granted access, remediate the issue, and verify that legitimate access remained intact.

---

## Initial Access

Jacob's legitimate authorization path was:

```text
jturner
  |
GG-Sales-Users
  |
DL-Sales-Share-RW
  |
Sales Share
```

The incorrect membership in `GG-HR-Users` introduced a second authorization path:

```text
jturner
  |
GG-HR-Users
  |
DL-HR-Share-RW
  |
HR Share
```

Testing from `IAM-WIN11` confirmed that Jacob could access both the Sales and Human Resources shares.

Sales access was expected.

HR access represented **excessive privilege**.

---

## Access Review

I used PowerShell to inspect the membership of the HR role:

```powershell
Get-ADGroupMember "GG-HR-Users" |
    Select-Object Name,SamAccountName,ObjectClass
```

Jacob appeared as an unexpected member.

I then reviewed his identity and group memberships to confirm that his organizational role belonged to Sales.

Additional group inspection allowed the access path to be traced through the existing AGDLP structure:

```text
jturner
  |
GG-HR-Users
  |
DL-HR-Share-RW
  |
HR Share
```

The investigation showed that the file permissions themselves were functioning correctly.

The excessive access originated from **incorrect identity group membership**.

---

## Remediation

The unauthorized entitlement was removed with:

```powershell
Remove-ADGroupMember -Identity "GG-HR-Users" -Members jturner
```

Jacob's legitimate membership in:

`GG-Sales-Users`

was preserved.

This was important because the goal of remediation was not to remove all access—it was to restore **least privilege while maintaining required business access**.

---

## Validation

After establishing a fresh logon session on `IAM-WIN11`, I tested both resources again.

| Resource | Before Remediation | After Remediation |

| Sales | Allowed | Allowed |
| Human Resources | Allowed | Denied |

The unauthorized HR access was successfully revoked while Jacob's legitimate Sales access continued to function.

---

## Outcome

The access review successfully identified and remediated an excessive privilege caused by incorrect role-group membership.

The scenario demonstrated the full access-review lifecycle:

```text
Identify
   |
Investigate
   |
Trace Authorization
   |
Remediate
   |
Validate
```

### Concepts Demonstrated

- Access reviews
- Entitlement auditing
- Excessive privilege identification
- Least-privilege remediation
- RBAC
- AGDLP
- PowerShell administration
- Positive and negative access validation