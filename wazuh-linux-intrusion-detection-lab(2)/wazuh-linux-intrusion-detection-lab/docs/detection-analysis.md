# Detection Analysis

## SOC Analyst Triage Workflow

For each alert, I treated Wazuh like a real SOC queue and asked:

1. **What triggered?** Review `rule.description`, `rule.id`, and `rule.level`.
2. **Where did it happen?** Confirm `agent.name`, `agent.id`, and `agent.ip`.
3. **Who was involved?** Identify `data.dstuser`, `data.srcuser`, and local account names.
4. **Where did it come from?** Check `data.srcip` and compare it to the Kali attacker IP.
5. **What is the original evidence?** Open `full_log`, not just the alert title.
6. **Is this isolated or correlated?** Compare timestamps before and after the alert.
7. **Should this be escalated?** Decide based on sequence, impact, and persistence indicators.

## What Wazuh Saw

The Wazuh dashboard captured events from the Ubuntu victim agent. The strongest detections were:

- SSH authentication failures.
- SSH authentication success.
- Successful sudo execution.
- File integrity changes.
- New local user creation.

## Most Important Alert Fields

When investigating each event, I focused on these fields:

| Field | Why it matters |
|---|---|
| `timestamp` | Places the event in the timeline |
| `agent.name` | Identifies the affected endpoint |
| `agent.ip` | Confirms the victim endpoint IP |
| `data.srcip` | Shows the source IP of the remote SSH activity |
| `data.dstuser` | Shows the targeted or successful username |
| `full_log` | Preserves the original Linux log line |
| `rule.description` | Summarizes Wazuh's interpretation |
| `rule.level` | Indicates alert severity |
| `rule.id` | Identifies the Wazuh rule |
| `rule.mitre.id` | Maps the alert to attacker behavior |

## SSH Authentication Failure

Wazuh captured failed SSH authentication events. One example `full_log` showed:

```text
Failed password for socuser from 192.168.56.103 ... ssh2
```

This maps well to:

- `T1110.001 - Password Guessing`
- `T1021.004 - Remote Services: SSH`

The key point is that these failures came from the Kali attacker IP and targeted the victim endpoint.

## SSH Authentication Success

Wazuh also captured successful authentication:

```text
Accepted password for socuser from 192.168.56.103 ... ssh2
```

This is important because the activity moved from attempted access to successful access. Wazuh mapped this behavior to:

- `T1078 - Valid Accounts`
- `T1021 - Remote Services`

## New User Added

The strongest post-compromise alert was the `useradd` event. The original run created `attacker-test`, and the clean rerun created `attacker-lab2` so the evidence was easier to isolate in Wazuh.

```text
new user: name=attacker-test, UID=1002, GID=1002, home=/home/attacker-test, shell=/bin/sh
new user: name=attacker-lab2, UID=1002, GID=1002, home=/home/attacker-lab2, shell=/bin/bash
```

Wazuh identified this as:

```text
Rule description: New user added to the system.
Rule ID: 5902
Rule level: 8
MITRE: T1136 - Create Account
```

This is high-value evidence because suspicious account creation is a classic persistence behavior.

## Password / Account State Change

The final screenshots also show the account manipulation step after `attacker-lab2` was created:

```text
PAM: User changed password.
full_log: pam_unix(passwd:chauthtok): password changed for attacker-lab2
Rule ID: 5555
Rule level: 3
```

In the lab, this came from locking the account with `passwd -l attacker-lab2`. In a real incident, similar telemetry could represent password changes, account state changes, or identity manipulation after compromise.

Analyst mapping:

- `T1098 - Account Manipulation`
- `T1548.003 - Sudo and Sudo Caching` when correlated with the nearby `Successful sudo to ROOT executed` event

## Detection Gaps and Notes

Some actions, like reading `/etc/passwd`, running basic shell commands, or using `curl`, may not always generate high-quality alerts by default. That is expected. A production environment could improve visibility with:

- Linux audit rules.
- Command-line logging.
- File Integrity Monitoring expansion.
- Additional Wazuh decoders/rules.
- EDR telemetry.

A good SOC analyst does not assume "no alert" means "no activity." It may simply mean the telemetry is not configured to capture that behavior.


## SOC Analyst Priority Findings

### Finding 1: SSH password guessing reached a successful login

This is the most important authentication finding. Failed SSH attempts are common, but the investigation becomes more serious when a successful SSH login follows the failures. The SOC analyst task is to correlate the failed and successful events by source IP, destination user, and time window.

**Escalation logic:**

```text
Same source IP + failed SSH attempts + successful login + post-login activity = suspicious valid-account access
```

### Finding 2: Local account creation occurred after access

The `useradd` alert is the highest-value post-compromise signal in this lab. Wazuh rule `5902` identified `New user added to the system`, and the alert mapped to `T1136 - Create Account`.

**SOC relevance:** unexpected local account creation on a Linux server should be reviewed immediately because it can indicate persistence.

### Finding 3: Account state/password change followed user creation

The PAM alert for `attacker-lab2` showed account/password state change telemetry. In the lab this came from locking the account, but in production the same class of event could indicate unauthorized password resets, account enabling/disabling, or persistence cleanup.

### Finding 4: Some actions need better telemetry

Wazuh did not necessarily produce clean high-fidelity alerts for every terminal-side action. That is a realistic SOC lesson. Reading `/etc/passwd`, running `curl`, or staging files may require additional audit logging, FIM tuning, or EDR telemetry to detect consistently.

## What I Would Put in a SOC Escalation

```text
Affected host: UbuntuUser / 192.168.56.102
Source IP: 192.168.56.103
Impacted account: socuser
Observed behavior: SSH authentication failures followed by successful SSH login, host enumeration, sudo usage, payload transfer, cron persistence simulation, and local account creation.
Highest-confidence alert: New user added to the system / useradd / T1136
Recommended action: confirm authorization, rotate credentials, remove unauthorized accounts, review persistence mechanisms, and harden SSH.
```
