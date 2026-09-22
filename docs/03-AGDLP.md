# Day 3 — AGDLP, File Shares & Least-Privilege Access

## Objectives

- Implement an AGDLP-based authorization model
- Create departmental file shares
- Configure Share and NTFS permissions
- Apply role-based access using Active Directory security groups
- Validate authorized and unauthorized access from `IAM-WIN11`

---

## Building the AGDLP Model

With users and departmental Global Security Groups already created, I built the resource-access layer of the environment using **AGDLP**:

**Accounts > Global Groups > Domain Local Groups > Permissions**

The existing Global groups represented departmental roles:

```text
GG-IT-Users
GG-HR-Users
GG-Finance-Users
GG-Sales-Users
```

I then created Domain Local Security Groups representing access to specific resources:

```text
DL-IT-Share-RW
DL-HR-Share-RW
DL-Finance-Share-RW
DL-Sales-Share-RW
```

Each departmental Global group was nested into its corresponding Domain Local group.

For example:

```text
Sarah Chen
    |
GG-Finance-Users
    |
DL-Finance-Share-RW
    |
Finance Share
```

This separates **who a user is / what role they have** from **what permissions exist on a resource**.

---

## Creating Departmental File Shares

On `IAM-DC01`, I created folders for each department:

```text
C:\Northstar-Shares\IT
C:\Northstar-Shares\Human Resources
C:\Northstar-Shares\Finance
C:\Northstar-Shares\Sales
```

These were shared across the network as:

```text
\\IAM-DC01\IT
\\IAM-DC01\HumanResources
\\IAM-DC01\Finance
\\IAM-DC01\Sales
```

Access was assigned through the corresponding Domain Local groups rather than directly to individual users.

For example:

```text
DL-Finance-Share-RW → Finance Share
DL-HR-Share-RW      → HR Share
```

---

## Configuring Share and NTFS Permissions

Both **Share permissions** and **NTFS permissions** were configured for each departmental resource.

The corresponding Domain Local group received:

- **Share permissions:** Change + Read
- **NTFS permissions:** Modify

Administrative and system permissions were preserved.

No individual employee accounts were added directly to the file permissions.

This kept authorization group-based and easier to manage.

Effective network access therefore followed the full chain:

```text
User Account
     ↓
Global Department Group
     ↓
Domain Local Resource Group
     ↓
Share + NTFS Permissions
     ↓
Department Resource
```

---

## Validating Access from IAM-WIN11

To verify the authorization model, I signed into the domain workstation as Finance employee:

`NORTHSTAR\schen`

I confirmed the authenticated identity with:

```cmd
whoami
```

which returned:

```text
northstar\schen
```

Sarah was then able to access:

```text
\\IAM-DC01\Finance
```

and successfully created and modified:

```text
Finance-Access-Test.txt
```

This confirmed that her Finance group memberships provided the expected read/write access.

---

## Testing Unauthorized Access

I then attempted to access resources belonging to other departments.

Sarah was denied access to:

```text
\\IAM-DC01\HumanResources
\\IAM-DC01\Sales
\\IAM-DC01\IT
```

This was an important part of the test.

The goal was not only to prove that authorized access worked, but also that the permissions enforced **least privilege** by preventing access to resources outside the user's assigned role.

The final result was:

| Resource | Result |
|---|---|
| Finance | Allowed |
| Human Resources | Denied |
| Sales | Denied |
| IT | Denied |

---

## Key IAM Concepts

This stage of the lab demonstrated several core access-control concepts:

- Role-Based Access Control (RBAC)
- AGDLP group nesting
- Least privilege
- Group-based authorization
- Share permissions
- NTFS permissions
- Positive and negative access testing

Instead of managing permissions user-by-user, access could now be controlled through Active Directory group membership.

For example, changing a user's departmental role could be handled by changing their Global group membership rather than modifying the resource ACL directly.

This provided the foundation for the identity lifecycle scenarios performed later in the lab.

---

## Day 3 Result

By the end of Day 3, Northstar Technologies had a functioning role-based authorization model.

Departmental users received access through:

```text
Account
  - Global Role Group
  - Domain Local Resource Group
  - Resource Permission
```

Testing from the domain-joined Windows 11 workstation confirmed that authorized departmental access succeeded while unauthorized cross-department access was denied.

The environment was now ready to test how these permissions would behave during real identity lifecycle events such as employee transfers, offboarding, and access reviews.