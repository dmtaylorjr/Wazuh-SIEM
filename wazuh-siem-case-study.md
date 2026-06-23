# Wazuh SIEM Home Lab — Threat Detection & Log Monitoring

**Type:** Home Lab Project  
**Role Target:** SOC Analyst, Cybersecurity  
**Tools:** Wazuh 4.7, VMware Workstation Pro, Ubuntu Server 22.04, Windows 10, Sysmon (SwiftOnSecurity config)  
**Status:** Complete

---

## Project Overview

This project involved deploying a fully functional Security Information and Event Management (SIEM) system using Wazuh in a home lab environment. The goal was to simulate a real enterprise SOC setup — a centralized log server collecting and analyzing endpoint telemetry, with custom detection rules for common attack patterns.

SIEM platforms are a core tool for SOC analysts. The ability to deploy, configure, and write detection logic for a SIEM is one of the most sought-after skills in entry-level and mid-level cybersecurity roles. This project was built specifically to demonstrate that capability hands-on.

---

## Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation Pro 26H1 (free via Broadcom) |
| Wazuh Server | Ubuntu Server 22.04 VM — 4 vCPU, 8GB RAM, 100GB disk |
| Wazuh Version | 4.7.5 (all-in-one install) |
| Monitored Endpoint | Windows 10 PC (agent: `windows-pc`) |
| Endpoint Telemetry | Sysmon v15 with SwiftOnSecurity configuration |
| Network | VMware NAT — VM IP: 192.168.77.129 |

---

## What I Built

- Deployed Wazuh all-in-one (manager + indexer + dashboard) on a Ubuntu Server VM
- Enrolled a Windows 10 machine as a monitored agent
- Installed Sysmon on the Windows endpoint using the SwiftOnSecurity ruleset for enhanced telemetry
- Configured the Wazuh agent to collect Sysmon logs from the Windows Event Log channel
- Built a custom SOC dashboard with a login activity bar chart (last 24 hours)
- Wrote two custom detection rules targeting high-priority security events

---

## Custom Detection Rules

Custom rules are written in XML and stored at `/var/ossec/etc/rules/local_rules.xml`. Wazuh evaluates incoming log events against these rules in real time and generates alerts when conditions are matched.

### Rule 100002 — Multiple Failed Login Attempts

```xml
<group name="local,authentication_failed,">
  <rule id="100002" level="10" frequency="5" timeframe="120">
    <if_matched_group>authentication_failed</if_matched_group>
    <description>Multiple failed login attempts detected</description>
    <group>authentication_failures,</group>
  </rule>
</group>
```

**Why this matters:** Five or more failed logins within 120 seconds is a classic indicator of a brute force attack. This rule fires a Level 10 alert (high severity) when that threshold is crossed — exactly the kind of detection logic a SOC analyst would be expected to implement and tune.

### Rule 100003 — New Admin Account Created

```xml
<group name="local,windows,">
  <rule id="100003" level="12">
    <if_sid>60144</if_sid>
    <description>New admin account created on Windows</description>
    <group>account_management,</group>
  </rule>
</group>
```

**Why this matters:** Unauthorized admin account creation is a common persistence technique used by attackers after gaining initial access. This rule fires a Level 12 alert (critical) and maps to the MITRE ATT&CK technique for privilege escalation. In a real SOC, this would trigger an immediate investigation.

---

## Dashboard

A custom OpenSearch dashboard was built inside Wazuh to visualize login activity over time. The visualization uses:

- **Data source:** `wazuh-alerts-*` index
- **Filter:** `rule.groups: authentication`
- **Chart type:** Vertical bar chart
- **X-axis:** Timestamp (date histogram, auto interval)
- **Y-axis:** Event count

This gives a SOC analyst a quick at-a-glance view of authentication spikes that may indicate a brute force attempt or unauthorized access.

---

## Challenges & How I Solved Them

### Ubuntu 24.04 Compatibility

Wazuh 4.7's install script officially supports Ubuntu up to 22.04. Running the standard install on the VM produced an OS compatibility error. The fix was to pass the `--ignore-check` flag to bypass the OS check:

```bash
sudo bash wazuh-install.sh -a --ignore-check
```

This is a documented workaround and the install completed successfully. Knowing how to read error messages and find the right flag is a practical skill — not every production environment runs a perfectly supported OS stack.

### Rules UI Bug

The Wazuh dashboard's Rules management page threw a JavaScript error due to the Ubuntu 24.04 compatibility gap. Rather than blocking progress, this was solved by writing the custom rules directly in the terminal using `nano` — which is actually how rules are managed in many real enterprise deployments. The Wazuh manager was then restarted to load the new rules:

```bash
sudo systemctl restart wazuh-manager
```

### curl vs wget

The initial Wazuh install script download using `curl -sO` failed to save the file to disk (it printed the script contents to the terminal instead). Switching to `wget` resolved the issue immediately. Small troubleshooting wins like this are worth documenting — they show adaptability.

---

## Results

After completing the setup, the Wazuh dashboard showed:

- **1 active agent** (Windows PC) reporting in real time
- **516+ alerts** generated within the first session, automatically categorized by MITRE ATT&CK technique
- **MITRE ATT&CK mappings** visible including T1059 (Command and Scripting Interpreter), T1105 (Ingress Tool Transfer), and T1087 (Account Discovery)
- **3 authentication successes** logged and visible in the login activity dashboard
- **Custom rules active** and loaded into the Wazuh manager

---

## What I Learned

- How SIEM architecture works end-to-end: agent → manager → indexer → dashboard
- How Sysmon enhances Windows endpoint visibility beyond default Windows Event Logs
- How to write custom Wazuh detection rules in XML targeting specific security events
- How MITRE ATT&CK techniques map to real log events in a SIEM
- How to troubleshoot OS compatibility issues in open-source security tooling
- The difference between high-severity alert levels and how thresholds affect detection sensitivity

---

## Files & Resources

- **GitHub Repo:** *(link to be added)*
- **Wazuh Documentation:** https://documentation.wazuh.com
- **Sysmon Config:** https://github.com/SwiftOnSecurity/sysmon-config
- **MITRE ATT&CK:** https://attack.mitre.org
