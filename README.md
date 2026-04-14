# 🛡️ Adversary Emulation & Detection Validation (Purple Team Lab POC)

An end-to-end Purple Teaming and Detection Engineering lab designed to move beyond passive security monitoring by actively simulating real-world attacks and engineering custom SIEM rules to eliminate visibility blindspots.

## 📌 Project Overview
This Proof of Concept (POC) validates the defensive capabilities of an enterprise SIEM. Instead of assuming out-of-the-box rules are sufficient, this project utilizes **Adversary Emulation** to execute attack chains, analyze the resulting telemetry, and engineer custom detection logic to achieve a 100% detection rate for targeted techniques.

* **Author:** Fuzail Khan
* **Focus Areas:** Adversary Emulation, Detection Engineering, SOC Pipeline Optimization, Log Correlation.

---

## 🏗️ Lab Architecture & Tooling

The lab was constructed in an isolated, air-gapped Oracle VirtualBox environment to ensure the safe execution of atomic tests and malware payloads.

### **Infrastructure**
* **Attacker Node & SIEM Host:** Kali Linux
* **Target Endpoint:** Windows 11 Enterprise

### **Tech Stack**
* **SIEM:** Wazuh Manager (v4.7.5) & Elastic Stack
* **Adversary Emulation:** Atomic Red Team (PowerShell)
* **Endpoint Telemetry:** Sysmon, Windows Advanced Audit Policies, Wazuh Agent
* **Log Sources:** Windows Security Logs (Event ID 4688), File Integrity Monitoring (FIM), PowerShell Script Block Logging

---

## ⚔️ Techniques Emulated & Detection Engineering

This project mapped simulated attacks directly to the **MITRE ATT&CK® Framework**. Below are the core techniques tested and the resulting detection engineering actions:

### 1. Process Discovery (MITRE T1057)
* **Attack:** Simulated an adversary enumerating running processes (e.g., executing `tasklist.exe`) to identify active EDR solutions.
* **Gap Analysis:** The default Wazuh ruleset ingested the log but failed to trigger a high-severity alert.
* **The Fix:** Engineered a custom XML detection rule (**Rule ID 100002**) utilizing PCRE2 regular expressions (`(?i)tasklist`) to catch the silent attack.
* **Result:** Successfully triggered a **Level 10 Critical Alert** upon re-execution.

### 2. Boot or Logon Autostart Execution: Registry Run Keys (MITRE T1547)
* **Attack:** Deployed malicious batch scripts into the Windows Startup folder and manipulated Registry Run Keys to point to a simulated payload, mimicking persistent access.
* **The Fix:** Aggressively tuned Wazuh's **File Integrity Monitoring (FIM)** module to monitor critical system paths (System32, Startup folders).
* **Result:** Triggered **Rule 554** (File added to the system), automatically detecting the unauthorized file creation.

### 3. OS Credential Dumping: LSASS Memory (MITRE T1003)
* **Attack:** Utilized `rundll32.exe` and `comsvcs.dll` to attempt an LSASS memory dump for Kerberos ticket theft.
* **The Fix:** While actively blocked by Windows Defender Tamper Protection, custom SIEM correlation rules were engineered to capture the specific "Access Denied" and "Malicious Script Detected" logs.
* **Result:** Tuned severities to ensure blocked credential dumping attempts still generate high-priority SOC tickets for immediate investigation.

---

## 🔧 Pipeline Optimization
Beyond rule creation, this project addressed critical SIEM pipeline stability and technical debt:
* **Time-Sync Resolution:** Investigated and resolved a critical log-dropping issue caused by a 10-hour timezone desynchronization between the victim endpoint and the SIEM host.
* **Rule Refactoring:** Refactored overlapping custom XML decoders to prevent **Rule Shadowing**, ensuring the most specific and critical alerts always trigger first.

---

## 📂 Repository Structure

* `/docs/` - Contains the final Project Report, architecture diagrams, and presentation materials.
* `/scripts/` - Contains the custom `local_rules.xml` decoders and Regex logic.

---

*This project demonstrates a proactive approach to cybersecurity, proving that continuous tuning and active validation are required to defend against modern adversaries.*
