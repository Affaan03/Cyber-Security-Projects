# Project: Automated SOC Response (SOAR) with Azure Sentinel

**Author:** Affaan Irshad
**Role:** Cybersecurity Analyst / Cloud Security Engineer
**Tools:** Microsoft Azure Sentinel, Logic Apps, KQL, NIST Incident Response

---

## 1. Executive Summary
**Objective:** To drastically reduce incident response time by automating the containment of brute-force attacks.
**Solution:** Configured Microsoft Sentinel to detect high-frequency login failures and deployed an Azure Logic App to automatically notify security teams with enriched incident details.
**Outcome:** Reduced response time from "detection-to-notification" to near real-time, demonstrating proficiency in **NIST Incident Response** and **Security Automation**.

---

## 2. The Problem (Business Context)
In a manual Security Operations Center (SOC), analysts are often flooded with thousands of logs. If an attacker is brute-forcing a server, a human analyst might not notice the alert for hours, leading to significant risk.
* **Business Risk:** Prolonged exposure to credential theft and potential ransomware entry.
* **Operational Bottleneck:** Wasted hours manually triaging low-level alerts instead of focusing on critical threats.

---

## 3. The Solution: Automated Workflow
I built a "Detect & Respond" pipeline using the cloud-native capabilities of Microsoft Azure, moving from passive monitoring to active automation.

### Phase 1: Detection (The "Eyes")
* **Technology:** Microsoft Sentinel (SIEM) & KQL (Kusto Query Language).
* **Logic:** I wrote a custom analytics rule to monitor Windows Security Events (`EventID 4625`).
* **Threshold:** The rule triggers a **High Severity Alert** immediately if a single IP address fails authentication more than 10 times within a 5-minute window.

**The KQL Query Used:**
```kusto
// Map Query for "Event" Table (XML Parser)
Event
| where EventID == 4625
| extend IpAddress = extract("IpAddress\">([0-9.]+)<", 1, EventData)
| where isnotempty(IpAddress) and IpAddress != "-"
| project TimeGenerated, IpAddress, Computer, EventID
```
## Phase 2: The Automated Playbook (Logic App)
**Goal:** Create the automated workflow that triggers when an attack is detected.

1. **Create the Logic App:**
    * Search for **Microsoft Sentinel** in the Azure Portal.
    * Navigate to **Configuration** > **Automation** > **Create** > **Playbook with Incident Trigger**.
    * **Resource Group:** Select `Honeypot-Lab`.
    * **Playbook Name:** `SOAR-Auto-Response-Email`.
    * Click **Next** > **Review and Create**.

2. **Design the Workflow:**
    * Once created, click **"Go to resource"** to open the Logic App Designer.
    * **Trigger:** The default trigger should be `Microsoft Sentinel incident`.
    * **Add Action:** Click **+ New Step**.
    * Search for **"Office 365 Outlook"** (for work/school mail) or **"Outlook.com"** (for personal mail).
    * Select Action: **Send an email (V2)**.

3. **Configure the Email:**
    * **To:** Enter your email address.
    * **Subject:** `ALERT: High Severity Incident Detected - @{triggerBody()?['object']?['properties']?['title']}`
    * **Body:**
      ```text
      Incident Severity: @{triggerBody()?['object']?['properties']?['severity']}
      Alert Name: @{triggerBody()?['object']?['properties']?['title']}
      Description: @{triggerBody()?['object']?['properties']?['description']}
      Time: @{triggerBody()?['object']?['properties']?['createdTimeUtc']}
      ```
    * **Save** the Logic App.

---

## Phase 3: The Detection Rule (KQL)
**Goal:** Configure Sentinel to detect the brute-force pattern and create an Incident.

1. **Create Analytics Rule:**
    * Go to **Sentinel** > **Analytics** > **Create** > **Scheduled query rule**.
    * **Name:** `Brute Force Attack Detected`
    * **Severity:** High
    * **Tactics:** Credential Access

2. **Set Rule Logic (The Code):**
    * Paste the following KQL query:
      ```kusto
      SecurityEvent
      | where EventID == 4625
      | summarize FailureCount = count() by IpAddress, Account
      | where FailureCount >= 10
      ```
    * **Schedule:** Run every **5 Minutes**; Lookup data from the last **1 Hour**.
    * **Threshold:** Generate alert when number of query results is **Greater than 0**.

3. **Enable Incident Creation:**
    * Under "Incident Settings", ensure "Enabled" is toggled **On**.

---

## Phase 4: Connection & Verification
**Goal:** Link the Detection (Phase 3) to the Automation (Phase 2).

1. **Automated Response:**
    * In the Analytics Rule wizard (under **Automated response** tab), click **+ Add new**.
    * **Trigger:** "When incident is created".
    * **Action:** Run Playbook.
    * **Select Playbook:** Choose `SOAR-Auto-Response-Email`.
    * Click **Apply** and then **Save** the rule.

2. **Validation:**
    * Simulate an attack by entering incorrect passwords on the VM (RDP).
    * Wait 5 minutes.
    * Verify that a **Red Alert** appears in the Sentinel Incidents/Alerts page.
    * Verify that an **Email** is delivered to your inbox with the incident details.
  
  ---

## Phase 5: Results
**Goal:** Confirm the system is working as intended.

1. **Incident Triggered:**
   * After generating the failed login attempts, Azure Sentinel successfully correlated the data and created an incident.
   * The status was correctly flagged as **"New"** with **"High"** severity.
   * The attack was mapped to the MITRE ATT&CK tactic: **Credential Access**.

2. **Automated Response:**
   * The Logic App successfully fired immediately after the incident creation.
   * An email was received within 60 seconds of the attack detection containing:
     * **Alert Name:** Brute Force Attack Detected
     * **Severity:** High
     * **Time of Attack:** [Timestamp]

---

## Phase 6: Visuals
*Evidence of the successful deployment.*

### 1. Detection (Sentinel Dashboard)
*The screenshot below shows the custom Analytics Rule triggering a "High Severity" alert in the Azure Portal.*

![Sentinel Alerts Page](PLACEHOLDER_FOR_ALERTS_IMAGE_LINK)
*(Instructions: Replace the link above with your `image_20b3aa.png`)*

### 2. Response (Logic App Success)
*The screenshot below verifies the Logic App run history and the automated email received by the SOC analyst.*

![Automated Email Notification](PLACEHOLDER_FOR_EMAIL_IMAGE_LINK)
*(Instructions: Replace the link above with your email screenshot)*

---

## Conclusion
In this project, I successfully constructed a **SOAR (Security Orchestration, Automation, and Response)** pipeline using Microsoft Azure.

* **Detection:** By shifting from standard logging to a custom KQL query, I reduced the detection window for brute-force attacks to 5 minutes.
* **Response:** By implementing a Logic App, I eliminated the need for manual monitoring, ensuring that critical alerts are delivered to the security team immediately.

This lab demonstrates practical application of the **NIST Incident Response Lifecycle** (specifically *Detection* and *Analysis*), proving the ability to automate high-volume security tasks in a cloud environment.
