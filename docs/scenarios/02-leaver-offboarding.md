# Scenario 02 — Employee Offboarding (Leaver)

## Scenario

**User:** Daniel Kim (`dkim`)  
**Department:** Human Resources  
**Status:** Offboarded

Daniel Kim was selected for a simulated employee offboarding workflow.

The objective was to prevent future authentication, revoke his departmental access, and retain the identity for administrative purposes.

---

## Initial Access

Before offboarding, Daniel was an active HR user.

His resource access followed:

```text
dkim
  |
GG-HR-Users
  |
DL-HR-Share-RW
  |
HR Share
```

Testing from `IAM-WIN11` confirmed that Daniel could access the Human Resources share while unrelated departmental resources remained restricted.

---

## Administrative Actions

The offboarding process consisted of:

```text
Disable Account
      |
Remove HR Group Membership
      |
Move Account to Disabled Users OU
      |
Validate Authentication Failure
```

Using Active Directory and PowerShell, I:

1. Disabled the `dkim` account
2. Verified that `Enabled` was set to `False`
3. Removed Daniel from `GG-HR-Users`
4. Created a dedicated `Disabled Users` OU
5. Moved Daniel's account into the new OU

The account was retained instead of deleted so the identity remained available for administrative and audit purposes.

---

## Validation

I attempted a fresh login to `IAM-WIN11` using Daniel's domain account.

Windows rejected the authentication attempt with:

> Your account has been disabled. Please see your system administrator.

This confirmed that the disabled identity could no longer establish a new domain session.

Group membership was also reviewed to verify that Daniel's HR entitlement had been removed.

---

## Outcome

The employee was successfully offboarded while the Active Directory identity was retained.

The workflow demonstrated two separate offboarding controls:

**Authentication was revoked** by disabling the account.

**Authorization was revoked** by removing the user's departmental role membership.

### Concepts Demonstrated

- Identity lifecycle management
- Leaver/offboarding workflow
- Account disabling
- Entitlement revocation
- Authentication vs. authorization
- Active Directory administration
- PowerShell validation