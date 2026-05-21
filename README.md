# Scanning & Enumeration Lab — Nmap + Metasploit

A hands-on penetration testing lab focused on the **reconnaissance and enumeration**
phases of an engagement. Each target on an isolated VMware host-only network is scanned
independently with **Nmap** and **Metasploit auxiliary scanners**, services and versions
are enumerated, and the results are analyzed to build a (non-exploitative) attack plan.

> ⚠️ **Scope & ethics:** All activity was performed against deliberately vulnerable lab VMs
> on a private, host-only network (`VMnet1`) that I own and control. No systems outside the
> lab were touched, and **no vulnerabilities were exploited** — this exercise stops at the
> enumeration phase. Do not run any of these techniques against systems you are not
> explicitly authorized to test.

---

## Objectives

- Identify live hosts and open ports across the lab subnet
- Enumerate services and software versions
- Run Nmap NSE scripts (including vulnerability scripts)
- Use Metasploit auxiliary scanner modules
- Analyze the collected data and prioritize targets — **without exploitation**

---

## Lab Environment

| VM        | Role                          | Operating System         |
| --------- | ----------------------------- | ------------------------ |
| ATTACK01  | Attacker                      | Kali Linux               |
| MSF01     | Vulnerable target             | Metasploitable 2 (Linux) |
| DC01      | Domain Controller (AD)        | Windows Server 2019      |
| WS01      | Domain-joined client          | Windows 10               |
| Network   | Isolated host-only segment    | VMware `VMnet1`          |

### Addressing

| Host                          | Address              |
| ----------------------------- | -------------------- |
| Kali Linux (ATTACK01)         | `192.168.206.133`    |
| Metasploitable 2 (MSF01)      | `192.168.206.166`    |
| Windows Server 2019 (DC01)    | `192.168.206.151`    |
| Windows 10 (WS01)             | `192.168.206.163`    |
| Subnet (all scans)            | `192.168.206.0/24`   |

---

## Tooling

| Tool                          | Purpose                                                  |
| ----------------------------- | -------------------------------------------------------- |
| **Nmap**                      | Host discovery, port scanning, service/version detection |
| **Nmap NSE (`--script vuln`)**| Scripted vulnerability detection (CVE/CVSS references)    |
| **Metasploit Framework**      | Auxiliary scanner modules for service enumeration        |
| **smbclient**                 | SMB share enumeration                                     |

---

## Methodology

The lab follows a target-by-target workflow. Only one target VM (plus the attacker)
is powered on at a time, so each scan is attributable to a single host.

1. **Preparation (ATTACK01)** — confirm the attacker's IP and network reachability.
2. **Metasploitable 2 (MSF01)** — full service scan, NSE vuln scan, Metasploit auxiliary enumeration.
3. **Windows Server / AD (DC01)** — targeted scan of Active Directory ports, SMB enumeration.
4. **Windows 10 (WS01)** — service scan of common Windows ports, SMB/RDP enumeration.
5. **Analysis** — compare attack surfaces and prioritize targets.

Full, copy-pasteable commands for every step live in
[`commands/reference.md`](commands/reference.md).

---

## Findings Summary

| Target | Focus                      | Key observations                                                                 |
| ------ | -------------------------- | -------------------------------------------------------------------------------- |
| MSF01  | Linux service exposure     | Large attack surface — many open ports running outdated, well-documented services |
| DC01   | Active Directory footprint | Classic AD service spread: DNS (53), Kerberos (88), LDAP (389/636/3268/3269), SMB (445) |
| WS01   | Windows client             | SMB (445) and **RDP (3389)** exposed — strong lateral-movement candidate          |

A deeper discussion of the results and target prioritization is in
[`docs/analysis.md`](docs/analysis.md).

---

## Repository Structure

```
scanning-enumeration-lab/
├── README.md              # this file — lab overview & summary
├── commands/
│   └── reference.md       # every command, organized by lab part
├── docs/
│   └── analysis.md        # enumeration analysis & attack-surface write-up
├── screenshots/
│   └── README.md          # what each of the 10 required screenshots should show
├── .gitignore
└── LICENSE
```

---

## How to Reproduce

1. Build the lab in VMware with the four VMs above on a host-only network (`VMnet1`).
2. Boot **ATTACK01 (Kali)** plus **one** target at a time.
3. Work through [`commands/reference.md`](commands/reference.md) section by section.
4. Capture the screenshots described in
   [`screenshots/README.md`](screenshots/README.md) and drop them in that folder.
5. Review your results against [`docs/analysis.md`](docs/analysis.md).

---

## Skills Demonstrated

- Network host discovery and port scanning with Nmap
- Service/version fingerprinting and NSE scripting
- Metasploit auxiliary module usage
- Active Directory service identification
- SMB and RDP enumeration
- Attack-surface analysis and target prioritization
