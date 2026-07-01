# Incident Report Template

## Incident Title

Suspicious SSH Activity and Potential Linux Host Compromise

## Executive Summary

A controlled intrusion simulation was performed against an Ubuntu endpoint monitored by Wazuh. The activity included service discovery, SSH password guessing, a successful SSH login, host enumeration, file transfer, cron persistence simulation, and local account creation. Wazuh generated alerts for SSH authentication activity and local account creation, which were reviewed through the dashboard using rule details and `full_log` evidence.

## Environment

| Asset | Value |
|---|---|
| SIEM | Wazuh |
| Victim Agent | UbuntuUser / Agent ID 001 |
| Victim IP | 192.168.56.102 |
| Attacker IP | 192.168.56.103 |
| Attack Platform | Kali Linux |

## Timeline

| Time | Event | Evidence |
|---|---|---|
| 14:09 | Nmap service discovery | `01_nmap_service_discovery.png` |
| 14:10-14:17 | SSH failed authentication and Hydra attack | `02`, `03`, `16`, `17` |
| 14:45 / 19:45 dashboard time | Successful SSH login | `15`, `18` |
| 14:22 | Host enumeration | `04`, `05`, `06`, `07` |
| 14:48 | Payload transfer and execution | `08`, `09` |
| 14:49 | Cron persistence test | `10` |
| 14:51 | Local user account created | `11`, `20` |

## Impact

In a real environment, this pattern would indicate possible compromise of a Linux host through SSH credential guessing followed by valid account access and persistence-oriented behavior.

## SOC Analyst Triage Fields

| Field | Value |
|---|---|
| Alert source | Wazuh Threat Hunting / Events |
| Affected endpoint | UbuntuUser / 192.168.56.102 |
| Source IP | 192.168.56.103 |
| Targeted user | socuser |
| Suspicious account created | attacker-test / attacker-lab2 |
| Key Wazuh rules | SSH auth failed/success, successful sudo, new user added, PAM password change |
| Highest confidence behavior | Local account creation after suspicious SSH activity |
| Escalation recommendation | Escalate to IR / senior analyst for containment validation |

## Evidence Summary

- Failed SSH authentication from attacker IP.
- Successful SSH authentication for `socuser`.
- Host and local account enumeration.
- Tool transfer via HTTP from Kali to victim.
- Cron persistence simulation.
- New user creation detected by Wazuh.

## MITRE ATT&CK Summary

- `T1046 - Network Service Discovery`
- `T1110.001 - Password Guessing`
- `T1021.004 - Remote Services: SSH`
- `T1078 - Valid Accounts`
- `T1033 - System Owner/User Discovery`
- `T1082 - System Information Discovery`
- `T1087.001 - Local Account Discovery`
- `T1105 - Ingress Tool Transfer`
- `T1059.004 - Unix Shell`
- `T1053.003 - Cron`
- `T1136.001 - Create Account: Local Account`
- `T1098 - Account Manipulation`

## Recommended Response

1. Confirm whether the SSH login was authorized.
2. Disable or rotate credentials for the affected account.
3. Review sudo activity and shell history.
4. Remove unauthorized accounts and persistence mechanisms.
5. Review network logs for additional source IP activity.
6. Harden SSH with key-based authentication, MFA where possible, rate limiting, and account lockout policy.
7. Add additional Linux audit rules for command execution, cron changes, and sensitive file access.

## Lessons Learned

The most useful evidence came from correlating multiple low/medium alerts into a single sequence. Individually, an SSH success may look routine. After repeated failures and followed by user creation, it becomes suspicious.


## Analyst Escalation Note

```text
I recommend escalation because the activity progressed beyond failed SSH attempts. Wazuh shows SSH authentication failures, a successful SSH login for socuser from the attacker IP, sudo activity, and local account creation. The local account creation alert maps to MITRE ATT&CK T1136 and may indicate persistence.
```

## Questions for Follow-Up

- Was `socuser` expected to log in from `192.168.56.103`?
- Was the new local account authorized?
- Were any additional cron jobs, SSH keys, or startup scripts created?
- Was the same source IP seen authenticating to other endpoints?
- Should the `socuser` password be rotated or disabled?

## Containment Checklist

- Disable suspicious local accounts.
- Rotate or reset impacted credentials.
- Review sudoers and group membership.
- Remove unauthorized cron entries and payload files.
- Block or monitor the source IP where appropriate.
- Preserve Wazuh alerts and endpoint logs for investigation.
