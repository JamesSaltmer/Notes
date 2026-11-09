# Transfer Tool Locations

Kali paths for files from these notes that are useful to upload, transfer, serve over HTTP/SMB, or copy into a target session.
## Linux Uploads

| File | Kali location | Typical use |
|---|---|---|
| `linpeas.sh` | `/usr/share/peass/linpeas/linpeas.sh` | Linux privilege escalation enumeration |

## Windows Uploads

| File | Kali location | Typical use |
|---|---|---|
| `winPEASx64.exe` | `/usr/share/peass/winpeas/winPEASx64.exe` | Windows privilege escalation enumeration, 64-bit |
| `winPEASx86.exe` | `/usr/share/peass/winpeas/winPEASx86.exe` | Windows privilege escalation enumeration, 32-bit |
| `winPEASany.exe` | `/usr/share/peass/winpeas/winPEASany.exe` | Windows privilege escalation enumeration, AnyCPU build |
| `winPEAS.bat` | `/usr/share/peass/winpeas/winPEAS.bat` | Windows privilege escalation enumeration via batch |
| `nc.exe` | `/usr/share/windows-resources/binaries/nc.exe` | Windows reverse shells and file transfer |
| `plink.exe` | `/usr/share/windows-resources/binaries/plink.exe` | Windows SSH tunnelling/pivoting |
| `PowerView.ps1` | `/usr/share/windows-resources/powersploit/Recon/PowerView.ps1` | AD enumeration from PowerShell |
| `PowerUp.ps1` | `/usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1` | Windows privilege escalation checks |
| `mimikatz.exe` x64 | `/usr/share/windows-resources/mimikatz/x64/mimikatz.exe` | Credential and Kerberos operations, 64-bit |
| `mimikatz.exe` x86 | `/usr/share/windows-resources/mimikatz/Win32/mimikatz.exe` | Credential and Kerberos operations, 32-bit |
| `SharpHound.ps1` | `/usr/lib/bloodhound/resources/app/Collectors/SharpHound.ps1` | BloodHound collection from PowerShell |
| `SharpHound.exe` | `/usr/share/metasploit-framework/data/post/SharpHound.exe` | BloodHound collection from Windows |

## Web Shells

| Resource | Kali location | Typical use |
|---|---|---|
| SecLists web shells | `/usr/share/seclists/Web-Shells/` | PHP/JSP/ASPX/CFM shell templates |
| FuzzDB `nc.exe` copy | `/usr/share/seclists/Web-Shells/FuzzDB/nc.exe` | Alternative Windows `nc.exe` copy |
| FuzzDB PHP command shell | `/usr/share/seclists/Web-Shells/FuzzDB/cmd.php` | PHP command shell |
| FuzzDB ASPX command shell | `/usr/share/seclists/Web-Shells/FuzzDB/cmd.aspx` | ASP.NET command shell |
| FuzzDB JSP command shell | `/usr/share/seclists/Web-Shells/FuzzDB/cmd.jsp` | JSP command shell |
| Laudanum shells | `/usr/share/seclists/Web-Shells/laudanum-1.0/` | Web shell templates |

## Wordlists

All paths are Kali defaults (`/usr/share/wordlists` and `/usr/share/seclists`) — nothing needs downloading. **If the exam supplies a wordlist, use that instead.**

`rockyou.txt` only exists as `.gz` on a fresh Kali — unpack once:

```bash
[ -f /usr/share/wordlists/rockyou.txt ] || sudo gunzip -k /usr/share/wordlists/rockyou.txt.gz
```

### Usernames

| List | Path | Use |
|---|---|---|
| Top 17 usernames | `/usr/share/seclists/Usernames/top-usernames-shortlist.txt` | Quick online brute force (SSH, FTP, SMB, web) |
| Unix service accounts | `/usr/share/wordlists/metasploit/unix_users.txt` | Linux user guessing |
| Default usernames | `/usr/share/seclists/Usernames/cirt-default-usernames.txt` | Devices / appliances |
| First names (10k) | `/usr/share/seclists/Usernames/Names/names.txt` | SMTP VRFY/EXPN, spraying, o365 |
| Surnames (top 1000) | `/usr/share/seclists/Usernames/Names/familynames-usa-top1000.txt` | Build `jsmith`-style AD names |
| Usernames (624k) | `/usr/share/seclists/Usernames/xato-net-10-million-usernames-dup.txt` | Kerbrute userenum (large) |
| nmap usernames | `/usr/share/nmap/nselib/data/usernames.lst` | Tiny default set |

`jsmith.txt` (statistically-likely-usernames) is **not** on Kali. Build one offline from names lists:

```bash
for f in a b c d e f g h i j k l m n o p q r s t u v w x y z; do sed "s/^/$f/" /usr/share/seclists/Usernames/Names/familynames-usa-top1000.txt; done | tr 'A-Z' 'a-z' > /tmp/jsmith.txt
```

### Passwords — online attacks (hydra / crackmapexec)

| List | Path | Size |
|---|---|---|
| Top 25 | `/usr/share/seclists/Passwords/Common-Credentials/top-passwords-shortlist.txt` | 25 |
| 2023 top 200 | `/usr/share/seclists/Passwords/2023-200_most_used_passwords.txt` | 200 |
| Worst 500 | `/usr/share/seclists/Passwords/Common-Credentials/500-worst-passwords.txt` | 500 |
| best1050 | `/usr/share/seclists/Passwords/Common-Credentials/best1050.txt` | 1k (default in notes) |
| Unix passwords | `/usr/share/wordlists/metasploit/unix_passwords.txt` | 1k |
| fasttrack | `/usr/share/wordlists/fasttrack.txt` | 262 |
| john default | `/usr/share/wordlists/john.lst` | 3.5k |
| 10k most common | `/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt` | 10k |
| rockyou top 9k | `/usr/share/seclists/Passwords/Leaked-Databases/rockyou-50.txt` | 9k |
| rockyou top 59k | `/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt` | 59k |

### Passwords — offline cracking (john / hashcat)

| List / rule | Path |
|---|---|
| rockyou | `/usr/share/wordlists/rockyou.txt` |
| rockyou (compressed) | `/usr/share/wordlists/rockyou.txt.gz` |
| NCSC 100k | `/usr/share/seclists/Passwords/Common-Credentials/100k-most-used-passwords-NCSC.txt` |
| metasploit password.lst | `/usr/share/wordlists/metasploit/password.lst` |
| hashcat best64 (newer Kali: best66) | `/usr/share/hashcat/rules/best64.rule` |
| hashcat rockyou-30000 | `/usr/share/hashcat/rules/rockyou-30000.rule` |
| hashcat d3ad0ne / dive | `/usr/share/hashcat/rules/d3ad0ne.rule`, `/usr/share/hashcat/rules/dive.rule` |
| john best64 | `/usr/share/john/rules/best64.rule` |

### Default credentials (`user:pass` format → `hydra -C`)

| Service | Path |
|---|---|
| SSH | `/usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt` |
| FTP | `/usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt` |
| Telnet | `/usr/share/seclists/Passwords/Default-Credentials/telnet-betterdefaultpasslist.txt` |
| Windows | `/usr/share/seclists/Passwords/Default-Credentials/windows-betterdefaultpasslist.txt` |
| MySQL | `/usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt` |
| MSSQL | `/usr/share/seclists/Passwords/Default-Credentials/mssql-betterdefaultpasslist.txt` |
| PostgreSQL | `/usr/share/seclists/Passwords/Default-Credentials/postgres-betterdefaultpasslist.txt` |
| Oracle | `/usr/share/seclists/Passwords/Default-Credentials/oracle-betterdefaultpasslist.txt` |
| VNC | `/usr/share/seclists/Passwords/Default-Credentials/vnc-betterdefaultpasslist.txt` |
| Tomcat | `/usr/share/seclists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist.txt` |
| Tomcat (metasploit) | `/usr/share/wordlists/metasploit/tomcat_mgr_default_userpass.txt` |
| HTTP basic auth | `/usr/share/wordlists/metasploit/http_default_userpass.txt` |
| All services | `/usr/share/wordlists/metasploit/default_userpass_for_services_unhash.txt` |
| Vendor table (CSV) | `/usr/share/seclists/Passwords/Default-Credentials/default-passwords.csv` |
| IPMI users / passwords | `/usr/share/wordlists/metasploit/ipmi_users.txt`, `/usr/share/wordlists/metasploit/ipmi_passwords.txt` |
| Oracle SIDs | `/usr/share/wordlists/metasploit/sid.txt` |
| Oracle user:pass | `/usr/share/wordlists/metasploit/oracle_default_userpass.txt` |

### SNMP community strings

| List | Path | Size |
|---|---|---|
| Common (fast, default in notes) | `/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt` | 118 |
| nmap | `/usr/share/nmap/nselib/data/snmpcommunities.lst` | 27 |
| metasploit | `/usr/share/wordlists/metasploit/snmp_default_pass.txt` | 120 |
| Full | `/usr/share/seclists/Discovery/SNMP/snmp.txt` | 3.2k |
| onesixtyone format | `/usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt` | 3.2k |

### DNS / subdomains / vhosts

| List | Path | Size |
|---|---|---|
| Top 5k | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` | 5k |
| Top 20k | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt` | 20k |
| Top 110k | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt` | 114k |
| fierce | `/usr/share/seclists/Discovery/DNS/fierce-hostlist.txt` | 2k |
| namelist | `/usr/share/seclists/Discovery/DNS/namelist.txt` | 151k |
| bitquark | `/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt` | 100k |
| dnsmap | `/usr/share/wordlists/dnsmap.txt` | small |

### Web — directories and files

| List | Path | Size |
|---|---|---|
| SecLists common (default in notes) | `/usr/share/seclists/Discovery/Web-Content/common.txt` | 4.7k |
| dirb common | `/usr/share/wordlists/dirb/common.txt` | 4.6k |
| quickhits (sensitive files) | `/usr/share/seclists/Discovery/Web-Content/quickhits.txt` | 2.5k |
| big | `/usr/share/seclists/Discovery/Web-Content/big.txt` | 20k |
| raft medium directories | `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt` | 30k |
| raft medium files | `/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt` | 17k |
| raft small words | `/usr/share/seclists/Discovery/Web-Content/raft-small-words.txt` | 43k |
| DirBuster small | `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt` | 87k |
| DirBuster medium | `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` | 220k |
| DB backups | `/usr/share/seclists/Discovery/Web-Content/Common-DB-Backups.txt` | 336 |
| Logins / admin pages | `/usr/share/seclists/Discovery/Web-Content/Logins.fuzz.txt` | 89 |
| API endpoints | `/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt` | 269 |
| API objects | `/usr/share/seclists/Discovery/Web-Content/api/objects.txt` | 3k |
| CGI scripts (Shellshock) | `/usr/share/seclists/Discovery/Web-Content/CGIs.txt` | |
| Apache | `/usr/share/seclists/Discovery/Web-Content/apache.txt` | |
| Tomcat | `/usr/share/seclists/Discovery/Web-Content/tomcat.txt` | |
| IIS | `/usr/share/seclists/Discovery/Web-Content/IIS.fuzz.txt` | |
| nginx | `/usr/share/seclists/Discovery/Web-Content/nginx.txt` | |
| WordPress | `/usr/share/seclists/Discovery/Web-Content/CMS/wordpress.fuzz.txt` | |
| dirb vuln lists | `/usr/share/wordlists/dirb/vulns/` | per-product |

### Web — parameters, extensions and payloads

| List | Path |
|---|---|
| Parameter names | `/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt` |
| Web extensions | `/usr/share/seclists/Discovery/Web-Content/web-extensions.txt` |
| Common extensions | `/usr/share/seclists/Fuzzing/extensions-most-common.fuzz.txt` |
| dirb extensions | `/usr/share/wordlists/dirb/extensions_common.txt` |
| Upload extensions | `/usr/share/seclists/Discovery/Web-Content/raft-small-extensions.txt` |
| Content-Types (upload bypass) | `/usr/share/seclists/Discovery/Web-Content/web-all-content-types.txt` |
| LFI Linux | `/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt` |
| LFI Linux (files) | `/usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt` |
| LFI Windows | `/usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-windows.txt` |
| Traversal | `/usr/share/wordlists/wfuzz/Injections/Traversal.txt` |
| SQLi quick | `/usr/share/seclists/Fuzzing/SQLi/quick-SQLi.txt` |
| SQLi generic | `/usr/share/seclists/Fuzzing/SQLi/Generic-SQLi.txt` |
| SQLi login bypass | `/usr/share/seclists/Fuzzing/Databases/sqli.auth.bypass.txt` |
| NoSQL | `/usr/share/seclists/Fuzzing/Databases/NoSQL.txt` |
| XSS | `/usr/share/seclists/Fuzzing/XSS/human-friendly/XSS-Jhaddix.txt` |
| Command injection | `/usr/share/seclists/Fuzzing/command-injection-commix.txt` |
| Special characters | `/usr/share/seclists/Fuzzing/special-chars.txt` |
| HTTP methods | `/usr/share/seclists/Fuzzing/http-request-methods.txt` |

### Numbers (IDs, PINs, ports)

| List | Path |
|---|---|
| 000–999 | `/usr/share/seclists/Fuzzing/3-digits-000-999.txt` |
| 0000–9999 | `/usr/share/seclists/Fuzzing/4-digits-0000-9999.txt` |
| Common HTTP ports | `/usr/share/seclists/Discovery/Infrastructure/common-http-ports.txt` |
| Any range | `seq 1 1000 > /tmp/ids.txt` |

### Quick picks

| Task | Use |
|---|---|
| Directory brute force | `/usr/share/seclists/Discovery/Web-Content/common.txt`, then `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` |
| Backup / config files | `/usr/share/seclists/Discovery/Web-Content/quickhits.txt` + `-x bak,old,zip` |
| Subdomain / vhost | `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` |
| SNMP community | `/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt` |
| SSH / FTP brute force | `/usr/share/seclists/Usernames/top-usernames-shortlist.txt` + `/usr/share/seclists/Passwords/Common-Credentials/best1050.txt` |
| Default creds | `hydra -C /usr/share/seclists/Passwords/Default-Credentials/<service>-betterdefaultpasslist.txt` |
| SMTP user enum | `/usr/share/seclists/Usernames/Names/names.txt` |
| Hash cracking | `/usr/share/wordlists/rockyou.txt` + `-r /usr/share/hashcat/rules/best64.rule` |
| Parameter fuzzing | `/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt` |
