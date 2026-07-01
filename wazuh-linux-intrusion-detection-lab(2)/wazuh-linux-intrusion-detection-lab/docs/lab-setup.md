# Lab Setup

## Environment

The lab used three virtual machines:

| VM | Role | Notes |
|---|---|---|
| Wazuh Server | SIEM / Manager | Wazuh manager, dashboard, indexer |
| Ubuntu Victim | Monitored endpoint | Wazuh agent installed and active as `UbuntuUser` |
| Kali Attacker | Attacker simulation box | Used for Nmap, SSH attempts, Hydra, and payload hosting |

## Network Design

The lab used a private host-only network for VM-to-VM traffic. The screenshots show the victim endpoint at `192.168.56.102` and the Kali attacker at `192.168.56.103` during the attack activity.

The important design principle was isolation: the lab traffic stayed inside the controlled virtual network.

## Wazuh Agent Enrollment

The Ubuntu victim was enrolled into Wazuh as:

```text
Agent name: UbuntuUser
Agent ID: 001
Status: Active
```

This matters because every alert in the investigation needed to be tied back to the monitored endpoint.
