# SOC Home Lab — Threat Detection with Splunk
 
## Architecture
Kali Linux (Attacker) → Windows 11 (Victim + Sysmon) → Ubuntu/Splunk (SIEM)
 
## What I built
- Deployed Splunk SIEM on Ubuntu, configured to ingest Windows 11 logs
- Installed Sysmon with SwiftOnSecurity config for deep endpoint telemetry
- Simulated 4 attack scenarios: port scan, brute force, RDP breach, persistence
- Wrote SPL detection rules for each attack type
- Built real-time alerts and a 5-panel SOC dashboard
- Documented findings in a formal incident report (MITRE ATT&CK mapped)
 
## Dashboard
![SOC Dashboard](screenshots/splunk-dashboard.png)
 
## Attack Chain Detected
![Attack Chain](screenshots/attack-chain-timechart.png)
