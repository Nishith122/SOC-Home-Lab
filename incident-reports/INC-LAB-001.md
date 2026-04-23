# Incident Report — INC-LAB-001

![Status](https://img.shields.io/badge/Status-Resolved-brightgreen)
![Severity](https://img.shields.io/badge/Severity-High-red)
![Type](https://img.shields.io/badge/Attack-Brute%20Force%20%7C%20RDP-blue)
![SIEM](https://img.shields.io/badge/SIEM-Splunk-orange)
![Platform](https://img.shields.io/badge/Platform-Windows%2011-lightgrey)
![Lab](https://img.shields.io/badge/Environment-SOC%20Lab-informational)

---

| Field    | Details                                         |
| -------- | ----------------------------------------------- |
| Date     | 23 April 2026                                   |
| Analyst  | Nishith Reddy                                   |
| Severity | High                                            |
| Category | Brute Force + Unauthorized Access + Persistence |
| Status   | Resolved                                        |

---

## Summary

A brute force attack was detected against Windows 11 host (`10.10.1.11`) originating from Kali Linux attacker machine (`10.10.1.59`). The attacker used Hydra to perform repeated RDP login attempts, generating multiple EventID 4625 (failed logon) events. A successful RDP session was subsequently established (EventID 4624, Logon_Type=10). Following the breach, a backdoor administrator account was created on the victim machine (EventID 4720) to maintain persistence. All activity was detected and alerted by Splunk SIEM running on Ubuntu (`10.10.1.9`).

---

## Environment

| Host       | IP         | Role          | OS           |
| ---------- | ---------- | ------------- | ------------ |
| kali-linux | 10.10.1.59 | Attacker      | Kali Linux   |
| Win11      | 10.10.1.11 | Victim        | Windows 11   |
| Ubuntu     | 10.10.1.9  | SIEM (Splunk) | Ubuntu 22.04 |

---

## Attack Timeline

| Time | Event                                                             | EventID         |
| ---- | ----------------------------------------------------------------- | --------------- |
| T+00 | First failed RDP login attempt detected                           | 4625            |
| T+02 | Brute force threshold breached — 5+ failures in 5 minutes         | 4625 ×5         |
| T+03 | Splunk alert fired: "Brute Force - Failed Login Threshold"        | Alert triggered |
| T+05 | Successful RDP login from Kali (Logon_Type=10)                    | 4624            |
| T+07 | Backdoor account 'hacker' created on Win11                        | 4720            |
| T+08 | Backdoor account added to Administrators group                    | 4732            |
| T+10 | Incident contained — backdoor account deleted, session terminated | Resolved        |

---

## Indicators of Compromise (IOCs)

| Type           | Value         | Notes                       |
| -------------- | ------------- | --------------------------- |
| Attacker IP    | 10.10.1.59    | Kali Linux VM               |
| Target IP      | 10.10.1.11    | Windows 11 victim           |
| Target account | Administrator | RDP brute forced            |
| Attack tool    | Hydra         | RDP brute force (-t 2 -W 3) |
| Backdoor user  | hacker        | Created post-breach         |
| Attack port    | 3389 (RDP)    | Remote Desktop Protocol     |

---

## Detection Details

### Alert 1 — Brute Force Threshold

* **Alert name:** Brute Force - Failed Login Threshold
* **Trigger:** 5+ failed logins (EventID 4625) from same source IP in 5 minutes
* **Fired at:** T+03 after attack began
* **Splunk index:** windows_logs

```spl
index=windows_logs EventCode=4625 earliest=-5m
| stats count as attempts by Account_Name, Source_Network_Address
| where attempts >= 5
```

---

### Alert 2 — New Account Created (Persistence)

* **Alert name:** Persistence - New User Account Created
* **Trigger:** Any EventID 4720 (account creation) in real-time
* **Splunk index:** windows_logs

```spl
index=windows_logs EventCode=4720 earliest=-5m
```

---

### Sysmon Detections

* **EventCode 1** — Process creation during RDP session (whoami, ipconfig, net user)
* **EventCode 3** — Network connections from Kali IP (`10.10.1.59`) port scan activity

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name                 | Observed Behavior                     |
| ------------ | ------------------------------ | ------------------------------------- |
| T1110.001    | Brute Force: Password Guessing | Hydra against RDP with wordlist       |
| T1021.001    | Remote Services: RDP           | xfreerdp used for unauthorized access |
| T1136.001    | Create Account: Local Account  | net user hacker /add after breach     |
| T1078        | Valid Accounts                 | Administrator account used post-brute |
| T1057        | Process Discovery              | whoami, ipconfig run post-compromise  |

---

## Actions Taken

1. Splunk alert fired automatically within 5 minutes of threshold breach
2. Identified attacker IP (`10.10.1.59`) from EventID 4625 logs
3. Confirmed successful RDP breach via EventID 4624 Logon_Type=10
4. Detected backdoor account creation via EventID 4720
5. Terminated RDP session
6. Deleted backdoor account: `net user hacker /delete`
7. Documented all IOCs and attack timeline

---

## Evidence Collected

* `screenshots/brute-force-alert-fired.png`
* `screenshots/attack-chain-timechart.png`
* `screenshots/rdp-login-logon-type-10.png`
* `screenshots/backdoor-account-creation.png`
* `screenshots/splunk-dashboard1.png`

---

## Recommendations

1. Implement account lockout policy — lock after 5 failed attempts in 5 minutes
2. Restrict RDP access to specific trusted IPs only via firewall rules
3. Enable Windows Defender Credential Guard to prevent credential theft
4. Monitor EventID 4720 in real-time
5. Use MFA for all RDP access

---

*Report prepared as part of SOC Home Lab project. All activity conducted in an isolated lab environment.*

