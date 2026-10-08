# Enterprise IT Helpdesk & Systems Administration Lab

A self-directed home lab simulating the IT infrastructure of a fictional 17-person consulting firm — **Meridian Consulting Group** — built on VirtualBox across Windows Server 2022, Windows 11, and Ubuntu Server 24.04 LTS.

The goal was not to follow a tutorial to completion. It was to build a working multi-platform domain environment from scratch, break it, and diagnose it properly.

---

## Scope and honest status

This project reached a **working, domain-joined, cross-platform environment**. It was paused before the helpdesk ticketing phase.

| Phase | Status |
|---|---|
| Network design and VirtualBox NAT network | Complete |
| Domain Controller — AD DS, DNS, DHCP | Complete |
| Active Directory structure — OUs, 18 accounts, 7 security groups | Complete |
| Three Windows 11 clients, domain-joined | Complete |
| Ubuntu server — static IP, SSH, verified from Windows client | Complete |
| LAMP stack and osTicket deployment | Deployed, web installer not run |
| Group Policy, shared folders, NTFS permissions | Not started |
| Simulated ticket workload | Not started |

The incident reports in [`incidents/`](incidents/) document real failures encountered during the build — not scripted exercises. Those are the most substantive part of this repository.

---

## Environment

```
                  VirtualBox NAT Network "meridian-lan"
                           10.10.10.0/24
                                 │
     ┌──────────────┬────────────┼────────────┬──────────────┐
     │              │            │            │              │
┌────▼─────┐  ┌─────▼────┐  ┌────▼─────┐  ┌───▼──────┐  ┌────▼─────┐
│ MER-DC01 │  │MER-FIN01 │  │MER-SAL01 │  │ MER-HR01 │  │MER-LNX01 │
│          │  │          │  │          │  │          │  │          │
│ Server   │  │ Win 11   │  │ Win 11   │  │ Win 11   │  │ Ubuntu   │
│ 2022     │  │ Finance  │  │ Sales    │  │ HR       │  │ 24.04 LTS│
│          │  │          │  │          │  │          │  │          │
│ AD DS    │  │ Sarah    │  │ Michael  │  │ Angela   │  │ LAMP     │
│ DNS      │  │ Whitfield│  │ Ondera   │  │ Dlamini  │  │ SSH      │
│ DHCP     │  │          │  │          │  │          │  │ osTicket │
│          │  │          │  │          │  │          │  │          │
│ .10      │  │ DHCP     │  │ DHCP     │  │ DHCP     │  │ .20      │
│ static   │  │ .100–150 │  │ .100–150 │  │ .100–150 │  │ static   │
└──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
```

**Domain:** `meridian.local`
**Host constraint:** 8 GB RAM — a real limitation that shaped VM sizing and caused one documented outage.

---

## What was actually built

### Domain Controller (MER-DC01)

- Windows Server 2022 Standard Evaluation, promoted to Domain Controller for a new forest
- DNS with external forwarders; IPv6 disabled on the adapter to force IPv4 resolution
- DHCP scope `10.10.10.100–150`, scope options 003 (router), 006 (DNS), 015 (domain name)
- Renamed post-promotion using `netdom` after a DNS fault exposed the original auto-generated hostname — see [INC-001](incidents/INC-001-dc-hostname-dns-resolution-failure.md)

### Active Directory

Full OU tree under a top-level `Meridian` OU, with `Users` and `Computers` sub-OUs per department, plus a dedicated `Admin Accounts` OU under IT.

- **18 accounts** — 17 staff across Management, Finance, Sales, HR and IT, plus one separate privileged account following least-privilege practice
- **7 global security groups** — departmental, plus `GG-IT-Admins` and `GG-AllStaff`
- Rebuilt entirely in **PowerShell** after a snapshot restore — see [`scripts/build-ad-structure.ps1`](scripts/build-ad-structure.ps1)

### Windows clients

Three Windows 11 Enterprise Evaluation VMs, each domain-joined and verified by logging in as its department user and confirming identity with `whoami`.

### Linux server (MER-LNX01)

- Ubuntu Server 24.04.5 LTS, static IP via Netplan, verified persistent across reboot
- OpenSSH — administered remotely from a domain-joined Windows client over SSH
- Apache 2.4.58 / MariaDB 10.11.14 / PHP 8.3.6
- osTicket 1.18.4 deployed to Apache; dedicated least-privilege database account created and verified

---

## Incident reports

Five documented faults, each with symptom, diagnostic sequence, root cause and resolution.

| ID | Incident | Root cause |
|---|---|---|
| [INC-001](incidents/INC-001-dc-hostname-dns-resolution-failure.md) | Linux server cannot resolve DC by hostname | DC never renamed after install; no matching A record existed |
| [INC-002](incidents/INC-002-domain-controller-trust-failure.md) | DC rejects all domain logins | Unresolved; recovered from snapshot |
| [INC-003](incidents/INC-003-host-memory-exhaustion.md) | Windows client kernel panic, Linux server hang | Host RAM exhaustion from VM over-allocation |
| [INC-004](incidents/INC-004-dhcp-apipa-address.md) | Clients self-assign `169.254.x.x` | DHCP server unavailable or not yet ready at client boot |
| [INC-005](incidents/INC-005-mariadb-account-host-mismatch.md) | Database login denied despite correct password | Account created with wrong host component |

---

## Documentation

- [Architecture and design decisions](docs/01-architecture.md)
- [IP addressing plan](docs/02-ip-addressing.md)
- [Active Directory structure](docs/03-active-directory.md)
- [Build log](docs/04-build-log.md)
- [Lessons learned](docs/05-lessons-learned.md)

---

## Skills demonstrated

**Windows Server** — AD DS installation and forest promotion, DNS zones and forwarders, DHCP scopes and options, supported domain controller rename (`netdom`), Directory Services Restore Mode, `ntdsutil`, `dcdiag`

**Active Directory** — OU design for Group Policy targeting, global security groups, least-privilege account separation, bulk provisioning in PowerShell

**Networking** — subnetting and static addressing, DHCP scope options, DNS forward lookup zones, diagnosing name resolution failures across platforms

**Linux** — Ubuntu Server administration, Netplan static configuration, systemd service inspection, socket activation, LAMP deployment, MariaDB user and privilege management

**Cross-platform** — SSH administration of a Linux server from a domain-joined Windows workstation

**Troubleshooting methodology** — layered diagnosis (link → address → routing → name resolution → application), hypothesis testing against evidence, and discarding a hypothesis when the evidence contradicted it

---

## Tools

VirtualBox · Windows Server 2022 · Windows 11 Enterprise · Ubuntu Server 24.04 LTS · Active Directory Users and Computers · DNS Manager · DHCP Manager · PowerShell · `netdom` · `dcdiag` · `ntdsutil` · `nslookup` · `ipconfig` · Netplan · `systemctl` · `ss` · MariaDB · Apache · osTicket

---

## Note on credentials

All passwords used in this lab were throwaway values for an isolated environment and are not reproduced anywhere in this repository. Screenshots were reviewed before publication to ensure none expose credentials.
