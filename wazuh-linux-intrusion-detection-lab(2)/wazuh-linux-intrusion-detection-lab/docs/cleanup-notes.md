# Cleanup Notes

After the lab, cleanup was performed on the Ubuntu victim to remove temporary users, cron entries, staged files, and harmless payload artifacts.

## Cleanup Commands Used

```bash
sudo userdel -r attacker-lab2 2>/dev/null
sudo userdel -r attacker-test 2>/dev/null
crontab -r
sudo rm -rf /tmp/exfil
sudo rm -f /tmp/payload.sh
sudo rm -f /tmp/soc-lab-payload.log
sudo rm -f /tmp/soc-lab-cron.log
```

## Verification Commands

```bash
getent passwd attacker-lab2
getent passwd attacker-test
crontab -l
ls -lh /tmp/exfil /tmp/payload.sh /tmp/soc-lab-payload.log /tmp/soc-lab-cron.log
```

Expected result:

- No output for removed attacker users.
- No active crontab for the lab user.
- No remaining `/tmp` lab artifacts.

## Important Note

Wazuh alert logs were intentionally preserved because they are evidence for the investigation and portfolio writeup.
