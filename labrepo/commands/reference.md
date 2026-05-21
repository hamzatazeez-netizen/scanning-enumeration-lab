# Commands Reference

Every command used in the lab, organized by part. Replace IP addresses with your own
lab values if they differ.

> All commands are run from **ATTACK01 (Kali)** unless stated otherwise.

---

## Tab labeling (required)

Set a prompt that includes the date and username so it appears in every terminal
screenshot. Set this in each new tab you open:

```bash
PS1='[`date "+%D"`] azeez@\h:\w\$ '
```

> In `msfconsole`, set a matching prompt:
> ```
> set PROMPT %yel%L %grn%T %grn Azeez
> ```

---

## Part 1 — Kali Preparation (ATTACK01)

### 1.1 Confirm network settings

```bash
ip a
```

Record your attacker IP — it is reused for all scans. In this lab: `192.168.206.133`.

---

## Part 2 — Scanning Metasploitable 2 (MSF01)

**Powered on:** ATTACK01 + MSF01

### 2.1 Identify the Metasploitable IP (host discovery)

```bash
nmap -sn 192.168.206.0/24
```

### 2.2 Full service scan

```bash
nmap -sS -sV -Pn 192.168.206.166
```

Observe: number of open ports, and any outdated service versions.

### 2.3 NSE vulnerability scan

```bash
sudo nmap -sV --script vuln 192.168.206.166
```

Observe: CVEs, CVSS scores, and exploit references reported by the scripts.

### 2.4 Metasploit auxiliary scanning

```bash
msfconsole
```

TCP port scan:

```
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.206.166
run
```

FTP version enumeration:

```
use auxiliary/scanner/ftp/ftp_version
set RHOSTS 192.168.206.166
run
```

> **Note:** The original lab sheet listed `192.168.27.142` here, which is outside this
> lab's `192.168.206.0/24` subnet — almost certainly a copy/paste artifact from the
> template. Use the MSF01 address (`192.168.206.166`) so the scan actually hits the target.

---

## Part 3 — Scanning Windows Server / AD (DC01)

**Powered on:** ATTACK01 + DC01

### 3.1 Identify DC01

```bash
nmap -sn 192.168.206.0/24
```

### 3.2 Targeted Active Directory port scan

```bash
nmap -sS -sV -Pn -p 53,88,135,139,389,445,464,636,3268,3269 192.168.206.151
```

Watch for: DNS (53), Kerberos (88), LDAP (389 / 636 / 3268 / 3269), SMB (445).

### 3.3 SMB enumeration

```bash
smbclient -L //192.168.206.151 -N
```

### 3.4 Metasploit auxiliary AD scanning

SMB version:

```
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.206.151
run
```

NetBIOS name enumeration:

```
use auxiliary/scanner/netbios/nbname
set RHOSTS 192.168.206.151
run
```

---

## Part 4 — Scanning Windows 10 (WS01)

**Powered on:** ATTACK01 + WS01

### 4.1 Identify WS01

```bash
nmap -sn 192.168.206.0/24
```

### 4.2 Service scan (common Windows ports)

```bash
nmap -sS -sV -Pn -p 135,139,445,3389,5985,5986 192.168.206.163
```

Ports of interest: SMB (445), RDP (3389), WinRM (5985/5986).

### 4.3 Metasploit auxiliary scan

SMB version:

```
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.206.163
run
```

RDP scanner:

```
use auxiliary/scanner/rdp/rdp_scanner
set RHOSTS 192.168.206.163
run
```

---

## Quick flag reference

| Flag            | Meaning                                              |
| --------------- | ---------------------------------------------------- |
| `-sn`           | Ping/host-discovery scan only (no port scan)         |
| `-sS`           | TCP SYN ("stealth") scan                             |
| `-sV`           | Probe open ports to determine service/version        |
| `-Pn`           | Skip host discovery; treat host as up                |
| `-p`            | Scan only the specified ports                        |
| `--script vuln` | Run the NSE vulnerability script category            |
