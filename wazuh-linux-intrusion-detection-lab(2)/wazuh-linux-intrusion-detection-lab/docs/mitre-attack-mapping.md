# MITRE ATT&CK Mapping

This lab mapped each major action to MITRE ATT&CK to show the behavior behind the logs.

## ATT&CK Table

| Lab Phase | Activity | Tactic | Technique |
|---|---|---|---|
| Nmap scan | Service discovery against victim | Discovery | `T1046 - Network Service Discovery` |
| Failed SSH attempts | Bad password attempts against SSH | Credential Access | `T1110.001 - Password Guessing` |
| SSH remote access | Remote login attempt over SSH | Lateral Movement / Initial Access context | `T1021.004 - Remote Services: SSH` |
| Successful login | Valid login as `socuser` | Defense Evasion / Persistence / Privilege Escalation / Initial Access / Lateral Movement context | `T1078 - Valid Accounts` |
| `whoami`, `id` | Identify current user and groups | Discovery | `T1033 - System Owner/User Discovery` |
| `hostname`, `uname`, `ip a` | System and network discovery | Discovery | `T1082 - System Information Discovery` |
| `/etc/passwd` | Local account discovery | Discovery | `T1087.001 - Account Discovery: Local Account` |
| `sudo -l` | Privilege and group context review | Discovery | `T1069.003 - Permission Groups Discovery: Local Groups` |
| `curl` payload download | File transfer from attacker host | Command and Control | `T1105 - Ingress Tool Transfer` |
| `bash /tmp/payload.sh` | Shell script execution | Execution | `T1059.004 - Command and Scripting Interpreter: Unix Shell` |
| Crontab entry | Scheduled execution test | Persistence | `T1053.003 - Scheduled Task/Job: Cron` |
| `useradd attacker-test` / `attacker-lab2` | New local account creation | Persistence | `T1136.001 - Create Account: Local Account` |
| `passwd -l attacker-lab2`, `userdel` | Account state change and cleanup | Persistence / Defense Evasion context | `T1098 - Account Manipulation` |
| `sudo` command execution | Privileged command execution used to modify local account state | Privilege Escalation context | `T1548.003 - Abuse Elevation Control Mechanism: Sudo and Sudo Caching` |

## Notes on MITRE Context

Some actions can map to more than one tactic depending on attacker intent. For example, a successful SSH login can represent initial access, lateral movement, persistence, or valid-account abuse depending on how the account was obtained and used.

For this lab, I used the Wazuh-provided MITRE mappings when available and supplemented them with analyst interpretation for terminal-side activity that did not always generate a direct alert.

## Account Manipulation Evidence Added Later

The final screenshots strengthen the persistence portion of the story:

- `21_wazuh_useradd_attacker_lab2_details.png` shows the clean `useradd` detection for `attacker-lab2`. This directly supports `T1136.001 - Create Account: Local Account`.
- `22_wazuh_password_change_attacker_lab2_details.png` shows the PAM password/account state change telemetry. This is mapped by analyst context to `T1098 - Account Manipulation`.
- `23_wazuh_account_manipulation_event_rows.png` shows the surrounding Wazuh events, including password change and successful sudo execution. This supports timeline correlation and privileged activity review, with `T1548.003` used as context for sudo-based privileged command execution.
