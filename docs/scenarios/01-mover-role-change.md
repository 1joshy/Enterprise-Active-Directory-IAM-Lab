# Scenario 01 — Employee Role Change (Mover)

## Scenario

**User:** Sarah Chen (`schen`)  
**Original Department:** Finance  
**New Department:** Sales

Sarah Chen was transferred from the Finance department to Sales.

Her existing Finance access needed to be revoked while granting the permissions required for her new Sales role.

---

## Initial Access

Before the transfer, Sarah received Finance access through the existing AGDLP structure:

```text
schen
  |
GG-Finance-Users
  |
DL-Finance-Share-RW
  |
Finance Share
```

From the domain workstation, Sarah could access the Finance share while HR, Sales, and IT resources were denied.

---

## Administrative Actions

I updated Sarah's identity to reflect the department transfer by:

1. Removing `schen` from `GG-Finance-Users`
2. Adding `schen` to `GG-Sales-Users`
3. Moving her AD user object from the Finance OU to the Sales OU
4. Verifying her new group memberships

The file-share ACLs did not require modification because access was already managed through security groups.

The new authorization path became:

```text
schen
  |
GG-Sales-Users
  |
DL-Sales-Share-RW
  |
Sales Share
```

---

## Validation

After establishing a fresh logon session on `IAM-WIN11`, I tested Sarah's access again.

| Resource | Before Transfer | After Transfer |

| Finance | Allowed | Denied |
| Sales | Denied | Allowed |

Sarah successfully created and modified a file in the Sales share after the transfer.

Her previous Finance access was no longer available.

---

## Outcome

The employee's permissions were successfully updated to match her new business role.

The scenario demonstrated how **role-based group membership allows access to change without directly modifying resource permissions for individual users**.

### Concepts Demonstrated

- Identity lifecycle management
- Mover workflow
- RBAC
- AGDLP
- Least privilege
- Group membership administration
- Access validation