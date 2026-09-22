# Day 2 — Active Directory Structure, Identities & Domain Workstation

## Objectives

- Design an organizational structure for Northstar Technologies in Active Directory
- Create Organizational Units for users, computers, and departments
- Provision employee accounts for multiple business departments
- Create role-based security groups
- Deploy a Windows 11 client workstation
- Configure the workstation to use the Domain Controller for DNS
- Join the workstation to the `ad.northstar.com` domain
- Validate domain authentication and Active Directory service discovery

---

## Designing the Organizational Unit Structure

With the Domain Controller and DNS infrastructure operational, I began organizing Active Directory to represent the fictional company **Northstar Technologies**.

Rather than placing all objects into Active Directory's default containers, I created a dedicated `Northstar` OU and organized objects underneath it.

The resulting structure was:

```text
Northstar
├── Users
│   ├── IT
│   ├── Human Resources
│   ├── Finance
│   └── Sales
│
├── Computers
│   ├── Workstations
│   └── Servers
│
└── Groups
```

This structure separates identities, endpoints, servers, and security groups while also allowing users to be organized according to business function.

Protection from accidental deletion was enabled on the organizational structure to reduce the risk of administrative mistakes.

---

## Provisioning Northstar Employee Accounts

I created eight employee identities across four departments.

| Department | Employee | Username |

| IT | Ethan Brooks | `ebrooks` |
| IT | Maya Patel | `mpatel` |
| Human Resources | Olivia Carter | `ocarter` |
| Human Resources | Daniel Kim | `dkim` |
| Finance | Sarah Chen | `schen` |
| Finance | Marcus Reed | `mreed` |
| Sales | Emily Rodriguez | `erodriguez` |
| Sales | Jacob Turner | `jturner` |

Each user was placed in the OU corresponding to their department.

Users were initially provisioned with temporary credentials and required to change their password at first logon.

This simulated a basic **joiner/provisioning workflow** in which an administrator creates an identity and places it into the appropriate organizational structure before access is assigned.

> Passwords used in the lab are intentionally not included in the repository.

---

## Creating Role-Based Security Groups

Rather than assigning permissions directly to individual employee accounts, I created **Global Security Groups** representing departmental roles:

```text
GG-IT-Users
GG-HR-Users
GG-Finance-Users
GG-Sales-Users
```

Each group was configured with:

- Group scope: **Global**
- Group type: **Security**

Users were then assigned to the Global group corresponding to their department.

For example:

```text
Sarah Chen
    ↓
GG-Finance-Users
```

At this stage, these groups represented **who the users were from an access-control perspective**.

They would later become the first layer of the lab's AGDLP authorization model.

---

## Deploying the Windows 11 Workstation

To test authentication and authorization from an actual domain endpoint, I deployed a Windows 11 virtual machine in VMware Workstation Pro.

The workstation was named:

`IAM-WIN11`

A local administrative account named:

`LabAdmin`

was created for workstation administration.

The endpoint received its network address through DHCP on the VMware NAT network, while its DNS configuration was manually changed to use the Domain Controller:

`192.168.102.131`

This distinction was intentional.

The workstation did not require a static IP address, but it **did** need to use Active Directory DNS so it could locate domain services.

The Domain Controller remained statically addressed because it provided infrastructure services that clients needed to locate consistently.

---

## Windows 11 Edition Troubleshooting

The initial Windows 11 VM was installed using **Windows 11 Home**.

When attempting to join the workstation to the Northstar domain, the option to join an Active Directory domain was unavailable.

I determined that Windows 11 Home does not support joining a traditional Active Directory domain.

I initially attempted to change the Windows edition, but the edition-change attempt was unsuccessful.

Rather than continue troubleshooting an unnecessary limitation, I reinstalled the VM using:

**Windows 11 Pro**

After reinstalling, the domain membership option became available.

> **Troubleshooting takeaway:** The problem was not Active Directory or DNS. The client operating system edition lacked the required domain-join capability. Identifying that requirement prevented unnecessary changes to the working Domain Controller.

---

## Local Account Setup During Windows OOBE

During Windows 11 setup, the normal Out-of-Box Experience attempted to require online account configuration.

To create a local administrative account for the lab, I opened Command Prompt during OOBE and used:

```cmd
ipconfig /release
start ms-cxh:localonly
```

This allowed creation of the local `LabAdmin` account without making the workstation dependent on a personal Microsoft account.

The local administrator account remained separate from the Northstar domain identities.

This created a useful distinction between:

```text
IAM-WIN11\LabAdmin
```

and domain identities such as:

```text
NORTHSTAR\schen
```

The first is authenticated by the workstation's local account database, while the second is authenticated through Active Directory.

---

## Validating DNS Before Domain Join

Before joining the workstation to the domain, I validated connectivity between `IAM-WIN11` and `IAM-DC01`.

The workstation successfully reached the Domain Controller by both IP address and hostname.

I also tested DNS resolution for Active Directory service records.

The client successfully resolved the LDAP SRV records for:

`_ldap._tcp.dc._msdcs.ad.northstar.com`

and identified:

`IAM-DC01.ad.northstar.com`

as the Domain Controller.

This confirmed that the workstation could use DNS to discover Active Directory services before attempting the domain join.

---

## Joining IAM-WIN11 to the Domain

With DNS working and Windows 11 Pro installed, I joined the workstation to:

`ad.northstar.com`

using authorized domain administrative credentials.

The domain join completed successfully, and the workstation was restarted.

After restart, the machine was now a member of the Northstar Active Directory domain.

Domain users could authenticate using credentials such as:

```text
NORTHSTAR\username
```

instead of relying exclusively on local workstation accounts.

---

## Organizing the Computer Object

When a computer joins an Active Directory domain, its computer account is initially created in the default `Computers` container unless another location has been specified.

After joining `IAM-WIN11`, I located its computer object in Active Directory and moved it into:

```text
Northstar
└── Computers
    └── Workstations
        └── IAM-WIN11
```

This was important for more than organization.

Placing workstations into a dedicated OU allows administrators to later target those systems with **Group Policy** without applying the same workstation policies indiscriminately to Domain Controllers or servers.

This became important later in the lab when a workstation security GPO was linked specifically to the `Workstations` OU.

---

## Authentication vs. Authorization

This stage of the lab also demonstrated an important IAM distinction.

**Authentication** answers:

> Who are you?

For example, Active Directory validating that the supplied credentials belong to:

`NORTHSTAR\schen`

**Authorization** answers:

> What are you allowed to access?

At the end of Day 2, Northstar had centralized identities and role groups, but the resource-permission model had not yet been fully implemented.

That would become the focus of the next stage of the lab.

---

## Day 2 Result

By the end of Day 2, the lab had progressed from basic Active Directory infrastructure into a functioning enterprise identity environment.

Northstar Technologies now had:

- A structured Organizational Unit hierarchy
- Eight employee identities across four departments
- Department-based Global Security Groups
- A Windows 11 Pro workstation
- A dedicated local workstation administrator
- Correct Active Directory DNS configuration
- A successfully domain-joined endpoint
- The `IAM-WIN11` computer object organized within the `Workstations` OU
- Successful DNS-based Active Directory service discovery

The environment was now ready for the next stage: implementing **AGDLP, departmental file shares, NTFS/share permissions, RBAC, and least-privilege access validation**.