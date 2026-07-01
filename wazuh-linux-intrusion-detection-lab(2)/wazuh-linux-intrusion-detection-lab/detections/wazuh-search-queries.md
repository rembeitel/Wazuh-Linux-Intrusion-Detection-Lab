# Wazuh Search Queries

These searches were used in **Wazuh → Threat Hunting → Events**.

## General Review

```text
agent.name: UbuntuUser
```

```text
manager.name: vboxuser-VirtualBox
```

## SSH Activity

```text
sshd
```

```text
failed password
```

```text
Failed password
```

```text
Accepted password
```

```text
socuser
```

## Brute Force / Password Guessing

```text
brute force
```

```text
authentication failed
```

```text
invalid user
```

## Privilege and Account Activity

```text
sudo
```

```text
useradd
```

```text
attacker-test
```

```text
passwd
```

```text
userdel
```

## Persistence / File Activity

```text
cron
```

```text
crontab
```

```text
payload
```

```text
exfil
```

## Fields Worth Adding as Columns

```text
agent.name
agent.ip
data.srcip
data.dstuser
full_log
location
rule.description
rule.id
rule.level
rule.mitre.id
rule.mitre.tactic
rule.mitre.technique
```


## SOC Analyst Pivot Searches

Use these when moving from one alert to the next during investigation.

### Pivot from failed SSH to successful SSH

```text
data.srcip: 192.168.56.103
```

```text
data.dstuser: socuser
```

```text
rule.groups: authentication_failed OR rule.groups: authentication_success
```

### Pivot from login to post-login activity

```text
socuser AND sudo
```

```text
socuser AND useradd
```

```text
agent.name: UbuntuUser AND rule.description: "Successful sudo to ROOT executed."
```

### Pivot to persistence/account manipulation

```text
attacker-test OR attacker-lab2
```

```text
rule.mitre.id: T1136
```

```text
rule.description: "New user added to the system."
```

## Recommended Screenshot Columns for SOC Portfolio

Add these columns before taking screenshots:

```text
timestamp
agent.name
data.srcip
data.dstuser
rule.description
rule.level
rule.id
full_log
rule.mitre.id
rule.mitre.technique
```
