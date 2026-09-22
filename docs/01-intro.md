# Day 1 — Active Directory Infrastructure & DNS

## Objectives

- Deploy a Windows Server virtual machine for the IAM lab
- Configure stable network connectivity for the future Domain Controller
- Install Active Directory Domain Services (AD DS) and DNS
- Create the Northstar Technologies Active Directory forest
- Validate DNS and Active Directory functionality
- Troubleshoot infrastructure and namespace issues encountered during deployment

---

## Lab Environment

The IAM lab was built locally using VMware Workstation Pro.

The fictional organization used throughout the project is **Northstar Technologies**.

Final Domain Controller configuration:

 Component Configuration
    Server Windows Server 2022 Standard Evaluation 
    Hostname `IAM-DC01`
    Domain `ad.northstar.com`
    NetBIOS Name `NORTHSTAR` 
    IP Address `192.168.102.131` 
    Subnet `/24` 
    Default Gateway `192.168.102.2` 
    Roles Active Directory Domain Services, DNS 

The Domain Controller was assigned a static IP because DNS and directory services require a predictable network location for domain clients.

---

## Initial Windows Server Deployment

The lab was originally started using Windows Server 2025.

Before installing Active Directory, I configured the server's networking and tested connectivity.

During this process, the server lost external network connectivity despite appearing to have the correct IPv4 configuration.

Reviewing the routing table revealed two competing default routes:

- A valid default route using the VMware NAT gateway
- An incorrect on-link `0.0.0.0/0` route

The invalid route was removed with PowerShell, restoring normal connectivity.

This reinforced the importance of validating the routing table rather than relying only on the visible IP configuration when troubleshooting network connectivity.

---

## Installing Active Directory Domain Services

After establishing network connectivity, I installed the **Active Directory Domain Services (AD DS)** and **DNS Server** roles.

The server was then promoted to the first Domain Controller in a new forest.

The original lab design attempted to use:

`ad.northstar.example`

The promotion itself completed successfully and Active Directory health checks did not initially indicate a problem.

However, the expected DNS forward lookup zones were not present.

---

## DNS Namespace Troubleshooting

This became the primary troubleshooting issue during the initial deployment.

I verified several components of the environment, including:

- AD DS health
- DNS services
- Active Directory DNS partitions
- Event logs
- Server networking
- Domain Controller functionality

Attempts to manually create the required `.example` DNS zone failed through both PowerShell and the DNS Manager GUI with a Windows error indicating an invalid parameter.

To determine whether the problem was DNS itself or the selected namespace, I performed an A/B test by attempting to create another DNS zone:

`test.contoso.com`

This zone was created successfully.

The test demonstrated that the DNS service itself was capable of creating zones and narrowed the issue to the namespace being used in this specific lab environment.

The `.example` top-level domain is reserved by RFC standards for documentation and examples. Rather than continuing to build the lab around a namespace that was producing unexpected behavior in this environment, I changed the internal lab domain to:

`ad.northstar.com`

I also rebuilt the Domain Controller using **Windows Server 2022 Standard Evaluation** to establish a clean and stable baseline.

> **Troubleshooting takeaway:** I did not assume that the operating system or DNS service itself was broken. By changing one variable and successfully creating a `.com` test zone, I isolated the namespace as the differentiating factor in this environment.

---

## Final Domain Configuration

The final Active Directory forest was created as:

`ad.northstar.com`

with the NetBIOS domain name:

`NORTHSTAR`

The server was configured as:

`IAM-DC01.ad.northstar.com`

The domain functional level was confirmed as:

`Windows2016Domain`

This is expected behavior because newer Windows Server versions continue to support the Windows Server 2016 Active Directory functional level.

---

## DNS Validation

After promotion, I verified that the expected Active Directory-integrated DNS zones existed:

- `ad.northstar.com`
- `_msdcs.ad.northstar.com`

I also queried Active Directory's LDAP SRV records.

The lookup successfully returned the Domain Controller:

`IAM-DC01.ad.northstar.com`

using LDAP port:

`389`

This confirmed that DNS could successfully locate the Domain Controller's LDAP service.

This validation is important because Active Directory clients rely heavily on DNS SRV records to discover services such as Domain Controllers.

---

## Active Directory Concepts Practiced

During this stage of the lab, I worked with several core Windows enterprise infrastructure concepts:

- Active Directory Domain Services
- Domain Controllers
- Forests and domains
- DNS
- AD-integrated DNS zones
- DNS SRV records
- LDAP
- Domain functional levels
- NetBIOS domain names
- Static server addressing
- Network route troubleshooting

One of the key lessons from this stage was that **DNS is foundational to Active Directory**. A Domain Controller can appear operational while DNS problems still prevent clients and services from locating domain resources correctly.

---

## Day 1 Result

By the end of Day 1, I had established a healthy Active Directory foundation for Northstar Technologies.

The final environment consisted of:

`IAM-DC01`

running:

- Windows Server 2022
- Active Directory Domain Services
- DNS Server

for the domain:

`ad.northstar.com`

DNS resolution and Active Directory service discovery were successfully validated, providing the foundation for creating the organization's IAM structure and joining endpoints to the domain.