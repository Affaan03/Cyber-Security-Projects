# Vulnerability Management Lab: Tenable Nessus Essentials

**Author:** Affaan Irshad
**Role:** Cybersecurity Analyst
**Tools:** Tenable Nessus Essentials, VMware/VirtualBox, Windows 10
**Focus:** Vulnerability Assessment, System Hardening, Risk Remediation

---

## 1. Executive Summary
**Objective:** To perform a comprehensive vulnerability assessment on a networked asset, identify critical security flaws, and execute a remediation plan to harden the system.
**Methodology:** Deployed Tenable Nessus Essentials to perform a **Credentialed Scan** against a Windows 10 target.
**Outcome:** Successfully identified critical vulnerabilities (including outdated software and missing OS patches), performed remediation, and verified the security posture improvement through a rescan, achieving a clean bill of health.

---

## 2. Business Justification (Why This Matters)
In a corporate environment, new vulnerabilities are discovered daily (zero-days, software updates). A "set it and forget it" approach leaves organizations exposed.
* **Proactive vs. Reactive:** This project demonstrates a **proactive** security stance. Instead of waiting for an incident (Incident Response), we identify and close the gap before an attacker can exploit it.
* **Attack Surface Reduction:** By removing unnecessary services and patching software, we reduce the number of entry points available to an adversary.
* **Compliance:** Regular scanning is mandatory for compliance standards like **PCI-DSS** (Credit Cards), **HIPAA** (Health), and **ISO 27001**.

---

## 3. Project Lifecycle

### Phase 1: Environment Setup & Configuration
To simulate a real corporate environment where security tools have administrative visibility, I configured the target VM for a **Credentialed Scan**. This allows Nessus to see "inside" the machine (Registry, File System) rather than just scanning the outside firewall.
* **Registry Modification:** Added the `LocalAccountTokenFilterPolicy` key to bypass UAC remote restrictions, ensuring the scanner had full administrative rights to audit the file system.
* **Services Configured:** Enabled Remote Registry and File & Printer Sharing to allow the scanner to inspect installed software versions.

### Phase 2: Vulnerability Discovery
* **Tool:** Tenable Nessus Essentials.
* **Scan Type:** Basic Network Scan (Credentialed).
* **Findings:**
    * **Critical (CVSS 9.8):** Outdated Web Browser (Firefox) susceptible to **Remote Code Execution (RCE)**.
    * **High:** Missing cumulative Windows Security Updates.
    * **Medium:** SMB Signing not required.

### Phase 3: Remediation & Verification
* **Action:** Applied patches and updated third-party software based on CVSS severity.
* **Verification:** Re-ran the scan, confirming the "Critical" alerts were removed and the system was clean.

---

## 4. Key Cyber Security Concepts Applied

* **Credentialed vs. Non-Credentialed Scanning:**
    * *Concept:* A non-credentialed scan only sees what is exposed to the outside network (like a locked door). A credentialed scan logs in and checks the software version numbers inside (like inspecting the wiring inside the house).
    * *Application:* I configured Windows Registry keys to allow Nessus to authenticate, providing a deeper and more accurate audit.

* **CVSS (Common Vulnerability Scoring System):**
    * *Concept:* An industry standard for rating the severity of security vulnerabilities.
    * *Application:* I used CVSS scores to prioritize my work, fixing the "Critical" (Score 9.0+) browser vulnerability first because it posed the highest risk.

* **False Positives:**
    * *Concept:* When a scanner thinks there is a bug, but there isn't.
    * *Application:* I manually verified the scan results to ensure the alerts were genuine before applying fixes.

---

## 5. Real-World Scenarios & Consequences

**Scenario A: The "Drive-By" Download (Browser Vulnerability)**
* **The Vulnerability:** The unpatched Firefox browser identified in this lab.
* **The Attack:** An employee visits a compromised website. Because the browser is outdated, malicious code is automatically downloaded and executed without the user clicking anything.
* **The Consequence:** This is a common entry point for **Ransomware**. The attacker gains a foothold, encrypts company data, and demands payment.
* **Business Impact:** Operational downtime, potential millions in ransom/recovery costs.

**Scenario B: Lateral Movement (Missing OS Patches)**
* **The Vulnerability:** Missing Windows Security Updates (e.g., SMB vulnerabilities).
* **The Attack:** An attacker gets onto one minor laptop. Because the internal network is unpatched, they use an exploit (like EternalBlue) to spread to the Domain Controller.
* **The Consequence:** Full domain compromise. The attacker owns the entire network.
* **Business Impact:** Complete loss of intellectual property and massive reputation damage.

---

## 6. Visuals

### Initial Scan Results (Before Fix)
*The screenshot below highlights the initial exposure, showing critical vulnerabilities.*
![Nessus Initial Scan Results](PLACEHOLDER_FOR_BEFORE_IMAGE)

### Remediation Verification (After Fix)
*The screenshot below demonstrates the improved security posture after patching.*
![Nessus Clean Scan](PLACEHOLDER_FOR_AFTER_IMAGE)
