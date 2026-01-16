# Technical Implementation Guide: Azure Sentinel RDP Honeypot

**Author:** Afan Nishad
**Project:** SIEM Deployment & Attack Visualization

## Overview
This guide outlines the exact steps required to deploy a Windows Virtual Machine in Azure, expose it to the internet, and configure Microsoft Sentinel to visualize brute-force attacks in real-time.

---

## Phase 1: Infrastructure Setup (The Honeypot)
**Goal:** Deploy a Windows VM and expose it to the public internet.

1. **Create a Virtual Machine:**
    * Go to the Azure Portal and search for "Virtual Machines".
    * **Resource Group:** Create new (e.g., `Honeypot-Lab`).
    * **Image:** Windows 10 Pro (or Server 2019).
    * **Size:** Standard_B2s (or D2s_v3) — *sufficient for lab testing*.
    * **Networking:** Create a new Virtual Network (VNet).

2. **Configure the Firewall (NSG):**
    * Navigate to the VM's **Networking** tab.
    * Add an **Inbound Security Rule**:
        * **Destination Port Ranges:** `*` (All ports)
        * **Protocol:** Any
        * **Action:** Allow
        * **Priority:** 100 (High priority)
        * **Name:** `DANGER_Allow_Any`
    * *Result:* The VM is now completely exposed to the internet, specifically on Port 3389 (RDP).

---

## Phase 2: Log Analytics Configuration
**Goal:** Create a repository to ingest the security logs.

1. **Create Log Analytics Workspace:**
    * Search for "Log Analytics Workspaces" in Azure.
    * Create a new workspace in the same Resource Group (`Honeypot-Lab`).

2. **Enable Microsoft Sentinel:**
    * Search for "Microsoft Sentinel".
    * Click **Create** and select the Log Analytics Workspace you just created.

3. **Connect VM to Workspace:**
    * Go back to Log Analytics Workspaces > Select your workspace.
    * Navigate to **"Virtual Machines"** (sidebar).
    * Click on your Honeypot VM and click **Connect**.
    * *Result:* The VM is now sending its default logs to the workspace.

---

## Phase 3: Custom Log Enrichment (PowerShell)
**Goal:** Extract IP addresses from Windows Event Logs and add Geo-Location data.

1. **Prepare the API:**
    * Sign up for a free Geolocation API key (e.g., *ipgeolocation.io*).
    * Note the API Key for the script.

2. **Deploy the Script:**
    * Log into the VM via Remote Desktop (RDP).
    * Open **PowerShell ISE**.
    * Paste the custom script (refer to `Custom_Security_Log_Exporter.ps1` in repository).
    * **Function:** The script scans **Event ID 4625** (Failed Login), grabs the IP, sends it to the API, and creates a custom log file (`failed_rdp.log`) with the coordinates.

3. **Create Custom Log in Azure:**
    * In Azure Log Analytics, go to **Settings** > **Custom Logs**.
    * Add a new custom log.
    * **Sample File:** Copy the `failed_rdp.log` content from the VM to your local machine and upload it as a sample.
    * **Collection Path:** Set the path to `C:\ProgramData\failed_rdp.log` (or wherever your script saves it on the VM).
    * **Name:** `FAILED_RDP_WITH_GEO_CL` (The `_CL` is added automatically).

---

## Phase 4: Data Visualization (The Map)
**Goal:** Visualizing the attacks using KQL and Workbooks.

1. **Verify Data Ingestion:**
    * Go to **Sentinel** > **Logs**.
    * Run the query: `FAILED_RDP_WITH_GEO_CL | take 10`
    * Ensure data is showing up with fields like `latitude`, `longitude`, and `country`.

2. **Create the Workbook:**
    * Go to **Sentinel** > **Workbooks** > **Add New**.
    * Click **Edit** > **Add Query**.

3. **The KQL Query:**
    Use the following query to extract the raw XML data and plot it:

    ```kusto
    FAILED_RDP_WITH_GEO_CL
    | extend username = extract(@"username:([^,]+)", 1, RawData),
             timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
             latitude = extract(@"latitude:([^,]+)", 1, RawData),
             longitude = extract(@"longitude:([^,]+)", 1, RawData),
             sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
             state = extract(@"state:([^,]+)", 1, RawData),
             label = extract(@"label:([^,]+)", 1, RawData),
             destination = extract(@"destinationhost:([^,]+)", 1, RawData)
    | where destination != "samplehost"
    | where sourcehost != ""
    | summarize event_count=count() by sourcehost, latitude, longitude, label, destination
    ```

4. **Configure Map Settings:**
    * **Visualization:** Select "Map".
    * **Map Settings:**
        * **Location Info:** Latitude/Longitude.
        * **Size by:** `event_count`.
        * **Color by:** `event_count` (Heatmap style).
    * **Apply:** Save and close.

---

## Phase 5: Verification
* Wait 1–2 hours for the honeypot to be discovered.
* Refresh the Workbook.
* You should see bubbles appearing over different countries, indicating active brute-force attempts.
