# SIEM Home Lab: Threat Detection & Incident Response

## Objective
The goal of this project was to build a functional Security Information and Event Management (SIEM) home lab to simulate real-world cyber attacks and practice Incident Response (IR). 

I deployed **Wazuh** as the primary SIEM solution to monitor a vulnerable Windows 10 endpoint. I then used **Kali Linux** to act as the attacker, executing Brute Force attacks and Malware simulation to test the defensive configuration.

### Skills Learned
- SIEM Configuration & Log Analysis (Wazuh).
- Endpoint Detection & Response (EDR) agents.
- Network Security & Firewall Troubleshooting.
- Attack Simulation (RDP Brute Force, Ransomware behavior).
- Troubleshooting Network Level Authentication (NLA) protocols.

---

## Network Diagram
- **Wazuh Server:** Hosted on VirtualBox (Linux-based).
- **Victim Endpoint:** Windows 10 Enterprise (Bridged Network).
- **Attacker:** Kali Linux (Bridged Network).

---

## Phase 1: File Integrity Monitoring (FIM) & Ransomware Simulation
To demonstrate the importance of integrity monitoring, I configured Wazuh to watch specific directories in real-time for unauthorized changes.

**The Attack:**
I wrote a custom PowerShell script to simulate a ransomware attack. The script:
1. Created 20 dummy "financial" files.
2. Rapidly encrypted (renamed) them with a `.locked` extension to mimic ransomware behavior.

**The Defense (Wazuh):**
Wazuh's FIM module detected the rapid file modifications and generated alerts immediately.

![FIM Alert Screenshot](images/fim-alert.png)
*(Above: Screenshot showing the spike in file modification alerts on the Wazuh dashboard)*

---

## Phase 2: RDP Brute Force Attack
I simulated an external attacker trying to gain unauthorized access via Remote Desktop Protocol (RDP).

**The Attack (Kali Linux):**
I used **Hydra** and **xfreerdp** from a Kali Linux machine to execute a dictionary attack against the Windows target.
- *Command used:* `xfreerdp /v:<Target_IP> /u:Administrator /p:WrongPass /cert:ignore`

**The Challenge (NLA Troubleshooting):**
During the attack, I encountered an `ERRCONNECT_LOGON_FAILURE` caused by Windows **Network Level Authentication (NLA)** blocking the connection.
- **Solution:** I researched the issue and re-configured the Windows Registry/Remote Settings to allow non-NLA connections for the testing phase, and validated that modern tools like `NetExec` can bypass this in production environments.

**The Defense (Wazuh):**
Wazuh ingested the Windows Security Event logs (Event ID 4625) and correlated them into a "Brute Force" alert.

![Brute Force Screenshot](images/brute-force-spike.png)
*(Above: Wazuh capturing multiple failed logon attempts from the Kali IP address)*

---

## Conclusion
This lab successfully demonstrated the "Cat and Mouse" relationship between Red Team attacks and Blue Team defense. I learned that default configurations (like NLA) often block simple attacks, requiring a deeper understanding of Windows protocols to properly simulate and detect threats.
