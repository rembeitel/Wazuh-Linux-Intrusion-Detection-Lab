# Attack Timeline

This timeline summarizes the controlled intrusion simulation from attacker activity to SIEM investigation.

| Order | Activity | Evidence | ATT&CK Mapping |
|---:|---|---|---|
| 1 | Ran Nmap service discovery against the Ubuntu victim | `01_nmap_service_discovery.png` | `T1046 - Network Service Discovery` |
| 2 | Attempted SSH logins with incorrect credentials | `02_ssh_failed_login_loop.png` | `T1110.001 - Password Guessing`, `T1021.004 - SSH` |
| 3 | Ran Hydra against SSH using a small lab wordlist | `03_hydra_password_attack_success.png` | `T1110.001`, `T1021.004`, `T1078` |
| 4 | Logged in as `socuser` over SSH | `04_valid_ssh_login_host_enumeration.png` | `T1078 - Valid Accounts`, `T1021.004 - SSH` |
| 5 | Ran basic host and identity enumeration | `04_valid_ssh_login_host_enumeration.png` | `T1033 - System Owner/User Discovery`, `T1082 - System Information Discovery` |
| 6 | Reviewed `/etc/passwd` for local accounts | `05_passwd_account_discovery.png`, `06_passwd_socuser_entry.png` | `T1087.001 - Local Account Discovery` |
| 7 | Checked sudo privileges | `07_sudo_privilege_check.png` | `T1069.003 - Permission Groups Discovery: Local Groups` |
| 8 | Prepared and transferred a harmless shell script | `08_payload_script_created.png`, `09_payload_transfer_execution.png` | `T1105 - Ingress Tool Transfer`, `T1059.004 - Unix Shell` |
| 9 | Simulated cron persistence | `10_cron_persistence_simulation.png` | `T1053.003 - Scheduled Task/Job: Cron` |
| 10 | Created, locked, validated, and deleted a local account | `11_useradd_account_manipulation.png`, `20_wazuh_useradd_create_account_details.png` | `T1136.001 - Local Account`, `T1098 - Account Manipulation` |
| 11 | Reran account creation with `attacker-lab2` for cleaner Wazuh evidence | `21_wazuh_useradd_attacker_lab2_details.png` | `T1136.001 - Create Account: Local Account` |
| 12 | Confirmed password/account state change telemetry for `attacker-lab2` | `22_wazuh_password_change_attacker_lab2_details.png`, `23_wazuh_account_manipulation_event_rows.png` | `T1098 - Account Manipulation`, `T1548.003 - Sudo and Sudo Caching` |
| 13 | Investigated alerts in Wazuh | `12` through `23` | Multiple rule-level mappings shown in Wazuh |

## Analyst Narrative

The activity began with service discovery and quickly moved into SSH credential attacks. After a successful password guess, the attacker logged in as `socuser`, enumerated the host, reviewed local accounts, staged files, transferred a harmless script, simulated persistence with cron, and created a local user account.

From a SOC perspective, the most important pivot was this sequence:

```text
SSH failures → SSH success → post-login enumeration → sudo usage → user creation → account state change
```

That chain is much more suspicious than any single event by itself.


## SOC Analyst Correlation View

This is how I would compress the timeline for a SOC handoff:

| Stage | SOC interpretation | Escalation value |
|---|---|---|
| Nmap scan | External host identified SSH on the victim | Context; not enough alone |
| SSH failures | Possible password guessing | Medium; depends on volume and source |
| Hydra success / valid SSH login | Credential attack appears to have succeeded | High when tied to earlier failures |
| Enumeration | User is interacting with the host after login | Supports hands-on-keyboard activity |
| Sudo usage | Account has or is using elevated privileges | Raises severity |
| Payload transfer | Remote file download from attacker host | Suspicious post-access behavior |
| Cron simulation | Persistence-style behavior | Raises severity |
| Local user creation | New account could preserve access | High-value escalation trigger |

## Final SOC Assessment

The activity should not be treated as isolated SSH noise. The evidence shows a progression from attempted access to successful login and post-compromise behavior. The local account creation event is the strongest indicator that the activity moved beyond basic scanning or failed login attempts.
