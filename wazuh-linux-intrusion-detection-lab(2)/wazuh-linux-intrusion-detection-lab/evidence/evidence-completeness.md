# Evidence Completeness Notes

The final three screenshots have been added to the repository and placed in the account manipulation / persistence detection section.

| Screenshot | Purpose | MITRE Context |
|---|---|---|
| `21_wazuh_useradd_attacker_lab2_details.png` | Clean Wazuh `useradd` evidence for `attacker-lab2` | `T1136.001 - Create Account: Local Account` |
| `22_wazuh_password_change_attacker_lab2_details.png` | PAM password/account state change evidence for `attacker-lab2` | `T1098 - Account Manipulation` |
| `23_wazuh_account_manipulation_event_rows.png` | Event-row correlation showing password change and sudo activity | `T1098`, `T1548.003` |

No evidence placeholders remain for this version of the lab write-up.
