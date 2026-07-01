# Evidence Walkthrough

This file embeds the screenshots in the same order as the lab timeline. Each image is tied to the behavior it proves and the MITRE ATT&CK mapping it supports.

## 1. Nmap service discovery against Ubuntu victim

**Phase:** Reconnaissance / Discovery  
**MITRE:** T1046 - Network Service Discovery

![Nmap service discovery against Ubuntu victim](screenshots/01_nmap_service_discovery.png)

## 2. Manual SSH failed login loop

**Phase:** Credential Access  
**MITRE:** T1110.001 - Password Guessing; T1021.004 - SSH

![Manual SSH failed login loop](screenshots/02_ssh_failed_login_loop.png)

## 3. Hydra SSH password attack success

**Phase:** Credential Access / Initial Access  
**MITRE:** T1110.001 - Password Guessing; T1021.004 - SSH; T1078 - Valid Accounts

![Hydra SSH password attack success](screenshots/03_hydra_password_attack_success.png)

## 4. Valid SSH login and host enumeration

**Phase:** Initial Access / Discovery  
**MITRE:** T1078 - Valid Accounts; T1021.004 - SSH; T1082 - System Information Discovery; T1033 - System Owner/User Discovery

![Valid SSH login and host enumeration](screenshots/04_valid_ssh_login_host_enumeration.png)

## 5. Account discovery through /etc/passwd

**Phase:** Discovery  
**MITRE:** T1087.001 - Local Account Discovery

![Account discovery through /etc/passwd](screenshots/05_passwd_account_discovery.png)

## 6. Confirmed socuser account entry in /etc/passwd

**Phase:** Discovery  
**MITRE:** T1087.001 - Local Account Discovery

![Confirmed socuser account entry in /etc/passwd](screenshots/06_passwd_socuser_entry.png)

## 7. Sudo privilege check

**Phase:** Discovery / Privilege Context  
**MITRE:** T1069.003 - Permission Groups Discovery: Local Groups; T1033 - System Owner/User Discovery

![Sudo privilege check](screenshots/07_sudo_privilege_check.png)

## 8. Harmless payload script prepared on Kali

**Phase:** Staging  
**MITRE:** T1105 - Ingress Tool Transfer preparation

![Harmless payload script prepared on Kali](screenshots/08_payload_script_created.png)

## 9. Payload transfer and execution via curl

**Phase:** Command and Control / Execution  
**MITRE:** T1105 - Ingress Tool Transfer; T1059.004 - Unix Shell

![Payload transfer and execution via curl](screenshots/09_payload_transfer_execution.png)

## 10. Cron persistence simulation

**Phase:** Persistence  
**MITRE:** T1053.003 - Scheduled Task/Job: Cron

![Cron persistence simulation](screenshots/10_cron_persistence_simulation.png)

## 11. User creation, password lock, and deletion

**Phase:** Persistence / Account Manipulation  
**MITRE:** T1136.001 - Create Account: Local Account; T1098 - Account Manipulation

![User creation, password lock, and deletion](screenshots/11_useradd_account_manipulation.png)

## 12. Wazuh threat hunting overview for UbuntuUser

**Phase:** SIEM Triage  
**MITRE:** Multiple detections across the intrusion chain

![Wazuh threat hunting overview for UbuntuUser](screenshots/12_wazuh_threat_hunting_overview.png)

## 13. Wazuh SSH search results

**Phase:** SIEM Triage  
**MITRE:** T1110.001; T1021.004; T1078

![Wazuh SSH search results](screenshots/13_wazuh_ssh_search_results.png)

## 14. Wazuh SSH authentication success rule details

**Phase:** Detection Engineering Context  
**MITRE:** T1078 - Valid Accounts; T1021 - Remote Services

![Wazuh SSH authentication success rule details](screenshots/14_wazuh_ssh_auth_success_rule.png)

## 15. Wazuh full_log for successful SSH login

**Phase:** Alert Investigation  
**MITRE:** T1078 - Valid Accounts; T1021.004 - SSH

![Wazuh full_log for successful SSH login](screenshots/15_wazuh_successful_ssh_full_log.png)

## 16. Wazuh failed authentication rows

**Phase:** Alert Investigation  
**MITRE:** T1110.001 - Password Guessing; T1021.004 - SSH

![Wazuh failed authentication rows](screenshots/16_wazuh_failed_authentication_rows.png)

## 17. Wazuh failed SSH document details with MITRE mapping

**Phase:** Alert Investigation  
**MITRE:** T1110.001 - Password Guessing; T1021.004 - SSH

![Wazuh failed SSH document details with MITRE mapping](screenshots/17_wazuh_failed_ssh_mitre_details.png)

## 18. Wazuh successful SSH document details with MITRE mapping

**Phase:** Alert Investigation  
**MITRE:** T1078 - Valid Accounts; T1021 - Remote Services

![Wazuh successful SSH document details with MITRE mapping](screenshots/18_wazuh_successful_ssh_mitre_details.png)

## 19. Wazuh related rule reference review

**Phase:** Rule Validation  
**MITRE:** Analyst validation: rule context review, not primary evidence

> Analyst note: this screenshot is included as a rule-context review. It is not used as primary evidence for the attack chain; it shows why analysts validate rule context instead of blindly trusting titles.

![Wazuh related rule reference review](screenshots/19_wazuh_rule_reference_review.png)

## 20. Wazuh useradd alert details

**Phase:** Persistence Detection  
**MITRE:** T1136 - Create Account

![Wazuh useradd alert details](screenshots/20_wazuh_useradd_create_account_details.png)

## 21. Wazuh useradd alert details - clean rerun with attacker-lab2

**Phase:** Persistence Detection / Account Creation  
**MITRE:** T1136.001 - Create Account: Local Account

This screenshot was added after rerunning the user creation phase with a new account name, `attacker-lab2`, so the Wazuh evidence was easy to isolate. The alert shows the original `useradd` log content, including the destination user, UID/GID, home directory, shell, agent, and manager.

![Wazuh useradd alert details for attacker-lab2](screenshots/21_wazuh_useradd_attacker_lab2_details.png)

## 22. Wazuh password change alert details for attacker-lab2

**Phase:** Account Manipulation  
**MITRE:** T1098 - Account Manipulation

After the local account was created, the account state was modified with `passwd -l attacker-lab2`. Wazuh recorded this as `PAM: User changed password.` Even though this was a controlled lock operation in the lab, the detection is useful because real intrusions often include account state changes, credential resets, account enabling/disabling, or persistence-related identity manipulation.

![Wazuh password change alert details for attacker-lab2](screenshots/22_wazuh_password_change_attacker_lab2_details.png)

## 23. Wazuh account manipulation event rows

**Phase:** Alert Correlation / Timeline Validation  
**MITRE:** T1098 - Account Manipulation; T1548.003 - Abuse Elevation Control Mechanism: Sudo and Sudo Caching

This screenshot shows the Wazuh event rows around the account manipulation activity, including `PAM: User changed password` and `Successful sudo to ROOT executed`. This is useful for analyst correlation because it connects the privileged action to the identity change. In a real investigation, these rows would help answer: who ran the command, when did it happen, and what happened immediately before or after?

![Wazuh account manipulation event rows](screenshots/23_wazuh_account_manipulation_event_rows.png)


## SOC Analyst Evidence Priority

For a SOC Analyst portfolio review, prioritize these screenshots:

1. `15_wazuh_successful_ssh_full_log.png` - best proof of successful access.
2. `17_wazuh_failed_ssh_mitre_details.png` - best proof of password guessing detection and MITRE mapping.
3. `20_wazuh_useradd_create_account_details.png` - best proof of local account creation.
4. `21_wazuh_useradd_attacker_lab2_details.png` - clean rerun of account creation with easy-to-isolate evidence.
5. `22_wazuh_password_change_attacker_lab2_details.png` - account state/password manipulation evidence.
6. `23_wazuh_account_manipulation_event_rows.png` - correlation of PAM and sudo activity.

These screenshots demonstrate core SOC skills: alert review, evidence validation, timeline correlation, and escalation judgment.
