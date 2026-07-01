# SOC Analyst Focus

This document explains the lab from the perspective of a SOC Analyst I / Junior Security Analyst role. The technical steps matter, but the bigger point is the investigation workflow: triage, validate, correlate, escalate, and document.

## Role-Relevant Goal

The lab was designed to answer a realistic SOC question:

> Did the Ubuntu endpoint experience suspicious SSH activity that progressed into post-login activity or persistence?

The evidence supports escalation because the activity followed a suspicious chain:

```text
Recon → SSH failures → password guessing success → valid SSH login → enumeration → sudo usage → payload transfer → cron persistence simulation → local account creation
```

A single alert can be ambiguous. A sequence of related alerts is an incident.

## SOC Triage Workflow Used

| Triage question | How I answered it | Evidence used |
|---|---|---|
| Which endpoint was affected? | Wazuh agent `UbuntuUser`, Agent ID `001` | Wazuh agent and event fields |
| What was the source? | Kali attacker host, `192.168.56.103` | SSH `data.srcip` and `full_log` |
| What account was targeted? | `socuser` during SSH activity | `data.dstuser`, SSH logs |
| Did access succeed? | Yes, SSH authentication success for `socuser` | Wazuh rule `5715`, `full_log` |
| What happened after access? | Enumeration, sudo, transfer, cron, user creation | Terminal evidence and Wazuh detections |
| Was persistence attempted? | Yes, cron simulation and local user creation | Cron screenshot and Wazuh `useradd` event |
| What is the severity? | Escalate due to successful login plus account creation | Correlated timeline |

## Alert Triage Notes

### SSH authentication failures

SOC interpretation: failed SSH logins from a source IP can indicate password guessing or brute force behavior. The key details are the source IP, username, count, and whether a later success occurs.

Important fields:

```text
data.srcip
data.dstuser
rule.description
rule.level
rule.mitre.id
full_log
```

Relevant ATT&CK:

- `T1110.001 - Password Guessing`
- `T1021.004 - Remote Services: SSH`

### Successful SSH authentication

SOC interpretation: a successful login is not always malicious, but it becomes suspicious when it follows password guessing or occurs from an unusual source. In this lab, the successful login came from the Kali attacker-side IP and was followed by post-login activity.

Relevant ATT&CK:

- `T1078 - Valid Accounts`
- `T1021.004 - Remote Services: SSH`

### Sudo activity

SOC interpretation: sudo activity after a suspicious SSH login matters because it may indicate privilege use or system modification. The event should be reviewed for command context and surrounding events.

Relevant ATT&CK:

- `T1548.003 - Sudo and Sudo Caching`

### Local user creation

SOC interpretation: local account creation is high-value evidence. In a real environment, an unexpected new user account can indicate persistence, account manipulation, or preparation for future access.

Relevant ATT&CK:

- `T1136.001 - Create Account: Local Account`
- `T1098 - Account Manipulation`

## Evidence That Matters Most for a SOC Analyst

| Priority | Evidence | Why it matters |
|---:|---|---|
| 1 | Wazuh `full_log` for successful SSH login | Proves user, source IP, service, and timestamp |
| 2 | Failed SSH authentication rows | Supports password guessing behavior |
| 3 | Hydra success screenshot | Shows how the valid password was obtained in the lab |
| 4 | `sudo -l` and sudo alert rows | Shows the account had elevated capability |
| 5 | Wazuh `useradd` alert | Strongest persistence/account-creation detection |
| 6 | PAM password/account state change | Shows account manipulation after creation |
| 7 | Timeline histogram in Wazuh | Shows clustered activity over time |

## Escalation Decision

I would escalate this incident because there was not just one suspicious event. The chain showed:

```text
Multiple SSH authentication failures
→ successful SSH login
→ host enumeration
→ sudo activity
→ account creation
→ password/account state change
```

That sequence indicates possible unauthorized access followed by persistence-oriented behavior.

## Example SOC Ticket Summary

```text
Title: Suspicious SSH Password Guessing Followed by Valid Login and Local Account Creation on UbuntuUser

Summary:
Wazuh generated multiple SSH authentication alerts for Ubuntu endpoint UbuntuUser. Failed SSH authentication attempts were observed from 192.168.56.103, followed by a successful SSH login for socuser from the same source. Subsequent activity included host enumeration, sudo usage, and local account creation for attacker-test / attacker-lab2. Wazuh detected the local account creation as rule 5902, New user added to the system, mapped to MITRE ATT&CK T1136 Create Account. PAM account state/password change telemetry was also observed for attacker-lab2.

Assessment:
The sequence is consistent with SSH password guessing followed by valid-account access and persistence-oriented account manipulation. Recommend escalation for containment and host review.
```

## Recommended SOC Response Actions

1. Confirm whether `socuser` login from `192.168.56.103` was authorized.
2. Disable or rotate the `socuser` password if unauthorized.
3. Review `/var/log/auth.log`, Wazuh `full_log`, and shell history for post-login commands.
4. Remove unauthorized local accounts.
5. Check crontab and common persistence paths.
6. Review additional network activity from the source IP.
7. Harden SSH with key-based authentication, lockout/rate limiting, and MFA where available.
8. Add or tune Linux audit rules for user creation, cron changes, file transfer, and command execution.

## What This Shows to a Hiring Manager

This lab demonstrates that I can:

- Build and troubleshoot a monitored endpoint environment.
- Generate controlled security telemetry.
- Search and filter alerts in a SIEM.
- Investigate `full_log` instead of relying only on alert titles.
- Identify source IP, target user, affected host, rule level, and MITRE mapping.
- Correlate separate events into an incident timeline.
- Explain containment, remediation, and visibility gaps.

## Interview Answer

A strong way to explain this project:

> I built a Wazuh lab to practice SOC alert triage. I used Kali to generate SSH password guessing and post-login Linux activity against an Ubuntu endpoint running the Wazuh agent. I then investigated the Wazuh alerts by reviewing full logs, source IPs, usernames, rule levels, and MITRE mappings. The biggest lesson was correlation: an SSH success alone may be normal, but SSH failures followed by a valid login, sudo usage, and local account creation should be escalated as a potential compromise.
