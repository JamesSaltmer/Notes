# Transfer Tool Locations

Kali paths for files from these notes that are useful to upload, transfer, serve over HTTP/SMB, or copy into a target session.

This intentionally excludes tools already callable from `PATH`, such as `nmap`, `ffuf`, `netexec`, `evil-winrm`, `impacket-*`, `msfvenom`, `sqlmap`, and similar CLI tools.

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
| `winPEAS.ps1` | `/usr/share/peass/winpeas/winPEAS.ps1` | Windows privilege escalation enumeration via PowerShell |
| `nc.exe` | `/usr/share/windows-resources/binaries/nc.exe` | Windows reverse shells and file transfer |
| `plink.exe` | `/usr/share/windows-resources/binaries/plink.exe` | Windows SSH tunnelling/pivoting |
| `PowerView.ps1` | `/usr/share/windows-resources/powersploit/Recon/PowerView.ps1` | AD enumeration from PowerShell |
| `PowerUp.ps1` | `/usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1` | Windows privilege escalation checks |
| `mimikatz.exe` x64 | `/usr/share/windows-resources/mimikatz/x64/mimikatz.exe` | Credential and Kerberos operations, 64-bit |
| `mimikatz.exe` x86 | `/usr/share/windows-resources/mimikatz/Win32/mimikatz.exe` | Credential and Kerberos operations, 32-bit |
| `SharpHound.ps1` | `/usr/share/powershell-empire/empire/server/data/module_source/situational_awareness/network/SharpHound.ps1` | BloodHound collection from PowerShell |
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

## Wordlists to Transfer or Reference

| Resource | Kali location | Typical use |
|---|---|---|
| `rockyou.txt` | `/usr/share/wordlists/rockyou.txt` | Password cracking/password spray list |
| `rockyou.txt.gz` | `/usr/share/wordlists/rockyou.txt.gz` | Compressed `rockyou.txt` copy |
| SecLists web content lists | `/usr/share/seclists/Discovery/Web-Content/` | Directory and file fuzzing lists |
| SecLists usernames | `/usr/share/seclists/Usernames/` | Username enumeration/spraying lists |
| SecLists passwords | `/usr/share/seclists/Passwords/` | Password testing lists |
