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

## 2) Web Login Bruteforce

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
