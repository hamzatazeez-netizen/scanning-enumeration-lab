# Analysis & Enumeration Summary

This write-up interprets the scan and enumeration results gathered across the three
targets and prioritizes them for a (hypothetical, non-executed) attack. No exploitation
was performed.

---

## 1. Which system exposed the largest attack surface?

**Metasploitable 2 (MSF01) — the Linux target.**

Metasploitable is intentionally built to be vulnerable, and the full service scan
(`nmap -sS -sV -Pn`) reflected that: it returned the highest number of open ports of any
host in the lab, and many of those ports were running noticeably outdated service
versions. The NSE vulnerability scan (`--script vuln`) then mapped a number of those
services to known CVEs with exploit references. More open, out-of-date services means more
potential entry points — so MSF01 has by far the broadest attack surface.

---

## 2. Which services appear most often in Windows / AD environments?

The targeted scan of DC01 surfaced the service set that characterizes an Active Directory
domain controller:

- **DNS (53)** — name resolution for the domain
- **Kerberos (88)** — domain authentication
- **LDAP (389 / 636 / 3268 / 3269)** — directory queries (plain, LDAPS, and Global Catalog)
- **SMB (445) / NetBIOS (139)** — file sharing and host/name enumeration
- **DHCP** — address assignment (commonly co-located in these environments)

This DNS / DHCP / Kerberos / LDAP / SMB cluster is the recurring fingerprint of a Windows
AD environment and is exactly what you look for when identifying a domain controller.

---

## 3. How do Metasploit auxiliary scans complement Nmap?

Nmap is excellent at the breadth-first questions: *what hosts are alive, what ports are
open, and what service/version is behind each port?* Metasploit's auxiliary scanners pick
up where that leaves off by drilling into a specific service to pull richer detail —
SMB dialect and OS info from `smb_version`, NetBIOS names from `nbname`, banner data from
`ftp_version`, RDP availability from `rdp_scanner`, and so on.

In short: **Nmap maps the open ports and the services on them; Metasploit auxiliary
modules enumerate those services more deeply**, helping confirm which weaknesses each
exposed service is likely susceptible to. Used together they give a fuller, more
actionable picture than either tool alone.

---

## 4. Which target would you attack first, and why? (No exploitation)

**The Windows 10 client (WS01).**

WS01 exposes **Remote Desktop (RDP) on port 3389** alongside SMB. RDP is a high-value
foothold: because the workstation is domain-joined, a legitimate local or domain account
reaching RDP would let an attacker establish an interactive session and then **move
laterally** through the domain, behaving like a persistent, hard-to-distinguish presence
on the network. That combination — an interactive remote-access service on a machine that
already trusts the domain — makes WS01 the most attractive first objective for gaining a
foothold and pivoting onward.

(MSF01 has the larger raw attack surface, but its value here is as a noisy, easily
compromised box rather than a stepping stone into the AD domain. WS01's RDP + domain
membership offers a cleaner path toward the actual prize: DC01.)

---

## Target prioritization at a glance

| Priority | Target | Rationale                                                              |
| -------- | ------ | ---------------------------------------------------------------------- |
| 1        | WS01   | RDP (3389) + domain membership → foothold and lateral movement         |
| 2        | MSF01  | Largest attack surface; many outdated, well-documented services        |
| 3        | DC01   | The high-value goal — approached *after* establishing a domain foothold |
