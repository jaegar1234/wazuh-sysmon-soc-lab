# Enterprise SIEM & Threat Detection Lab (Wazuh & Sysmon)

A self-hosted SOC lab built from scratch to demonstrate end-to-end security monitoring — from raw endpoint telemetry to MITRE ATT&CK-mapped detections.

## Overview

This project deploys a full open-source SIEM stack (Wazuh) monitoring a Windows endpoint instrumented with Sysmon, then validates the detection pipeline using controlled adversary-emulation techniques. The goal was to build, break, troubleshoot, and validate a realistic detection environment — not just follow a tutorial.

**Full incident report:** [`SOC_Incident_Report.docx`](./SOC_Incident_Report.docx)

## Architecture

- **Hypervisor:** Oracle VirtualBox (Bridged networking)
- **SIEM Stack:** Wazuh Manager + Indexer + Dashboard (all-in-one), Ubuntu Server, v4.14.7
- **Monitored Endpoint:** Windows 11 Pro, Wazuh Agent + Sysmon (SwiftOnSecurity config)
- **Pipeline:** Sysmon / Windows Security Log → Wazuh Agent → Wazuh Manager (rule correlation) → Indexer → Dashboard

## What Was Tested

| Technique | MITRE ID | Tactic | Result |
|---|---|---|---|
| Ingress Tool Transfer | T1105 | Command and Control | Detected (false positive, triaged) |
| PowerShell Execution (custom rule) | T1059.001 | Execution | Detected |
| Hosts File Tampering (FIM) | T1565.001 | Impact | Detected |
| Credential Brute Force | T1110 | Credential Access | Detected |
| Account Lockout | T1531 | Impact | Detected |
| DLL Search Order Hijacking (alert triage) | T1574.001 / .002 | Persistence / Priv. Esc. / Defense Evasion | False positive, triaged |

## Highlights

- **Custom detection engineering:** authored a local Wazuh rule (`local_rules.xml`) to detect PowerShell-driven file creation, mapped to MITRE T1059.001.
- **File Integrity Monitoring:** real-time syscheck on `C:\Windows\System32\drivers\etc`, validated with full before/after hash comparison (MD5/SHA1/SHA256).
- **Alert triage:** investigated an unplanned cluster of 613 high-volume alerts, traced to legitimate Windows Update activity, and correctly closed as a false positive.
- **Infrastructure troubleshooting:** resolved a VM networking misconfiguration, an agent/manager version mismatch, a disk-exhaustion event that corrupted the OpenSearch security plugin (requiring a full stack reinstall), and a Sysmon event-forwarding gap that required pivoting detection logic to a different event source.

## Repo Contents

- `SOC_Incident_Report.docx` — full incident tickets, MITRE coverage table, and lessons learned
- `screenshots/` — Wazuh Dashboard views (MITRE ATT&CK matrix, alert details)
- `local_rules.xml` — custom Wazuh detection rule authored for this lab

## Tools Used

Wazuh, Sysmon (SwiftOnSecurity config), VirtualBox, Ubuntu Server, PowerShell
