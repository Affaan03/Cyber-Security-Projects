# Azure Cloud Honeypot: SIEM & Threat Intelligence

## Introduction
This project demonstrates a functional Cloud-Based SIEM (Security Information and Event Management) system built in **Microsoft Azure**. I deployed a vulnerable "Honeypot" Virtual Machine exposed to the internet, aggregated real-time attack logs into a centralized **Log Analytics Workspace**, and utilized **Microsoft Sentinel** to visualize the geolocation of attackers on a live World Map.

The project simulates a typical threat scenario (RDP Brute Force) and showcases the entire lifecycle of a security analyst's workflow: **Infrastructure Setup -> Log Ingestion -> Threat Analysis -> Visualization.**

---

## Technical Skills Demonstrated
- **Cloud Infrastructure (Azure):** Configuring Virtual Machines, Virtual Networks, and Network Security Groups (NSGs).
- **SIEM Configuration:** Deploying Microsoft Sentinel and connecting data sources via Data Collection Rules (DCR).
- **Log Analytics:** Creating workspaces and managing log retention.
- **KQL (Kusto Query Language):** Writing complex queries to parse XML data, filter specific Event IDs, and extract custom fields.
- **Network Security:** Implementing firewall rules (Windows Defender Firewall) and cloud-based access controls.
- **Troubleshooting:** Diagnosing log ingestion latency and agent configuration issues (AMA vs. Legacy Agents).

---

## Architecture
1. **The Honeypot:** A Windows 10 Virtual Machine hosted in Azure, acting as the target.
2. **The Listener:** Azure Log Analytics Workspace effectively absorbing all system events.
3. **The Brain:** Microsoft Sentinel parsing the data for threats.
4. **The Dashboard:** A custom Workbook visualizing attack origins.

---

## Step-by-Step Implementation

### 1. Infrastructure Setup (The Target)
I deployed a Windows 10 Enterprise (Version 22H2, Gen2) Virtual Machine in Azure to serve as our honeypot.
- **VM Configuration:**
    - *Size:* `Standard_D2s_v3` (2 vCPUs, 8GB RAM). This size was selected to ensure the VM could handle the processing load of the monitoring agent without lagging.
    - *Region:* (Selected Region, e.g., West Europe) to match the resource group location.
- **Network Security Group (NSG) Configuration:**
    - To make the VM discoverable by attackers, I modified the Azure Network Security Group (the cloud firewall).
    - I deleted the default restricted rules and created a new inbound rule named `DANGER_ALLOW_ANY` with Priority 100.
    - *Settings:* Protocol: Any, Port: Any (*), Source: Any, Action: Allow.
    - *Why:* This removed the Azure-level protections, exposing the VM's public IP address directly to the open internet.

### 2. Security Configuration (The Trap)
Inside the Windows OS, I configured the internal firewall to simulate a specific, realistic security misconfiguration.
- **Windows Defender Firewall State:** **Enabled**.
    - Unlike typical honeypots that disable the firewall entirely, I kept the Domain, Private, and Public profiles **ON** to mimic a production server that *thinks* it is secure.
- **The Vulnerability (Port 3389):**
    - I manually created a new Inbound Rule in Windows Defender Firewall specifically for **Remote Desktop Protocol (RDP)**.
    - *Rule Name:* `Allow_RDP_Any`.
    - *Action:* Allow connection on TCP Port 3389 from Any IP Address.
    - *Why:* This precise configuration simulates a careless administrator who leaves the "front door" (RDP) open for convenience, while keeping the rest of the firewall active. This ensures attackers can reach the login screen to generate failure logs, but other ports remain blocked.

### 3. Log Ingestion Pipeline
To analyze the attacks, I needed to extract the Event Logs from the VM and send them to the cloud.
- **Log Analytics Workspace:** Created a custom workspace (`honeypot-log-workspace`) to store the raw data.
- **Microsoft Sentinel:** Enabled Sentinel on top of the workspace to provide SIEM capabilities.
- **Data Collection Rule (DCR):**
    - I connected the VM to Sentinel using the **Azure Monitor Agent (AMA)**.
    - I configured the DCR to collect **Windows Security Events**.
    - *Crucial Setting:* I specifically selected **"Audit Failure"** and **"Information"** log levels.
    - *Reason:* A failed login attempt (Event ID 4625) is technically classified as an "Audit Failure," not an "Error." Default settings often miss these critical security events.

### 4. Threat Analysis & Troubleshooting (KQL)
Once the pipeline was active, I used **Kusto Query Language (KQL)** to hunt for the attack data.

**The Challenge:**
Initially, I attempted to query the standard `SecurityEvent` table, but it returned zero results despite the VM being active. After troubleshooting, I discovered that the new Azure Monitor Agent (AMA) writes logs to a different, generic table called `Event`.

**The Solution:**
I pivoted my strategy to query the `Event` table. However, the data in this table is stored in a complex XML format inside the `EventData` column. I wrote a custom KQL parser to extract the hidden IP addresses from the XML structure.

**The Custom KQL Query:**
```kusto
// Map Query for "Event" Table (XML Parser)
Event
| where EventID == 4625
| extend IpAddress = extract("IpAddress\">([0-9.]+)<", 1, EventData)
| where isnotempty(IpAddress) and IpAddress != "-"
| extend Country = tostring(geo_info_from_ip_address(IpAddress).country)
| summarize AttackCount = count() by Country, IpAddress
| project Country, IpAddress, AttackCount
| sort by AttackCount desc
```

### 5. Visualization (The World Map)
To transform the raw log data into actionable threat intelligence, I utilized **Azure Workbooks** within Microsoft Sentinel. This allowed me to plot the attackers' locations on a dynamic, interactive map.

- **Workbook Configuration:**
    - I created a new custom Workbook and removed the default widgets to start with a clean slate.
    - I injected the custom KQL query (from Step 4) into the workbook's query editor.
- **Map Settings:**
    - **Visualization Type:** Set to `Map` to display geospatial data.
    - **Location Info:** Mapped the `Country` column (generated by the `geo_info_from_ip_address` function) to the map's region setting.
    - **Metric Value:** Mapped the `AttackCount` column to the "Bubble Size." This ensures that countries with higher attack volumes appear as larger, more urgent bubbles on the dashboard.
    - **Palette:** Configured a "Heatmap" style color scale (Red/Orange) to visually indicate the intensity of threats.
- **Live Monitoring:**
    - Enabled the **"Auto-Refresh"** feature (set to 5 minutes). This turned the static chart into a live "Threat Monitor" that updated automatically as new attackers were discovered in real-time.

---

## Results
The experiment successfully demonstrated how quickly internet-facing assets are discovered and compromised by automated threats.

### 1. Attack Velocity
Within **minutes** of creating the `Allow_RDP_Any` firewall rule, the honeypot began receiving connection attempts. This confirms that attackers use automated scanners to constantly sweep the entire IPv4 address space for vulnerable targets.

### 2. The Threat Map
The screenshot below represents a 24-hour snapshot of global RDP brute-force attempts against the honeypot.

![Live Attack Map](path/to/your/screenshot.png)
*(Above: Real-time visualization of attacker geolocation constructed in Microsoft Sentinel)*

### 3. Key Metrics
- **Total Attacks Captured:** Over **[INSERT NUMBER, e.g., 2,500+]** failed login attempts were recorded in the first 24 hours.
- **Top Attacking Sources:** The majority of attacks originated from:
    1. **[INSERT COUNTRY 1, e.g., China]**
    2. **[INSERT COUNTRY 2, e.g., Russia]**
    3. **[INSERT COUNTRY 3, e.g., Brazil]**
- **Credential Analysis:** The logs revealed that attackers primarily used "Dictionary Attacks," cycling through common administrative usernames such as:
    - `Administrator`
    - `Admin`
    - `User`
    - `Test`

---

## Conclusion
This project served as a practical simulation of the cyber threat landscape and the necessary defensive countermeasures.

### Key Takeaways:
1.  **The Internet is Noisy:** "Security through obscurity" is a fallacy. Any public IP address is constantly being scanned. Leaving a port open without strict filtering guarantees it will be attacked.
2.  **The Importance of SIEM:** Without a SIEM like Microsoft Sentinel, these thousands of attacks would be hidden inside local Windows Event Logs, likely unnoticed by an administrator. Centralizing logs allows for pattern detection and rapid response.
3.  **Data Normalization is Critical:** One of the major technical hurdles was parsing the XML data from the new Azure Monitor Agent (AMA). This highlighted the real-world need for Security Engineers to understand data structures and query languages (KQL) to extract meaningful insights from raw logs.

By building this lab, I gained hands-on experience in **Azure Cloud Security, Log Analytics, KQL Scripting, and Threat Intelligence Visualization**—core competencies for a modern Security Operations Center (SOC) Analyst.
