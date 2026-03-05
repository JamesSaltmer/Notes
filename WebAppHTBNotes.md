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

