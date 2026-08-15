# Lab Architecture

## Overview

The Home SOC Lab is a virtualized security monitoring environment designed to simulate a small Security Operations Center workflow.

The architecture combines endpoint telemetry, network telemetry, centralized log collection, detection logic, and alerting inside an isolated VirtualBox lab network.

The environment currently includes:

- A Windows endpoint monitored with Windows Event Logs and Sysmon
- An Ubuntu SOC server running Splunk Enterprise and Suricata
- A Kali Linux attacker VM used to generate controlled reconnaissance traffic

---

## Systems

| System | Operating System | Role | IP Address |
| --- | --- | --- | --- |
| SOC-CLIENT | Windows 10 | Monitored endpoint / Sysmon telemetry source | `192.168.56.10` |
| SOC-SERVER | Ubuntu Server | Splunk Enterprise / Suricata IDS / Log Server | `192.168.56.20` |
| KALI-ATTACKER | Kali Linux | Controlled attacker / reconnaissance system | `192.168.56.30` |

---

## Network Layout

```text
                         Windows Host
                             |
                         VirtualBox
                             |
                  SOC-LAB 192.168.56.0/24
                             |
             +---------------+---------------+
             |               |               |
             v               v               v
        SOC-CLIENT       SOC-SERVER      KALI-ATTACKER
        Windows 10       Ubuntu Server      Kali Linux
      192.168.56.10     192.168.56.20    192.168.56.30
             |               |               |
             |               |               |
      Windows Events         |          Nmap / Recon
          + Sysmon           |               |
             |               |<--------------+
             |               |
             | TCP 9997      | Suricata
             +-------------->| Network IDS
                             |
                             v
                     Splunk Enterprise
                             |
                             v
                    Detection & Alerting
```

---

## Windows Endpoint

The Windows endpoint acts as the monitored host in the lab.

Telemetry is collected through:

- Windows Security Event Log
- Windows System Event Log
- Windows Application Event Log
- Microsoft Sysmon Operational Log

The Splunk Universal Forwarder sends collected events to the Splunk server over TCP port `9997`.

Sysmon provides additional endpoint visibility, including:

- Process creation
- Command-line execution
- Parent-child process relationships
- Network connections
- File creation
- Registry activity

---

## Splunk Server

The Ubuntu server hosts Splunk Enterprise and acts as the central log collection and analysis platform.

Its responsibilities include:

- Receiving Windows Event Logs
- Receiving Sysmon telemetry
- Monitoring Suricata `eve.json`
- Indexing endpoint and network security events
- Running SPL searches
- Detecting suspicious activity
- Generating scheduled alerts
- Supporting security investigation

---

## Suricata Network Monitoring

Suricata runs on the Ubuntu SOC server and monitors the internal `SOC-LAB` interface.

Network telemetry is written to:

```text
/var/log/suricata/eve.json
```

Splunk monitors this file directly using:

```ini
[monitor:///var/log/suricata/eve.json]
disabled = 0
sourcetype = suricata:eve
index = suricata
```

This provides network-flow telemetry for reconnaissance and port-scan detection.

---

## Kali Linux Attacker VM

The Kali Linux VM is used to generate controlled security-testing activity inside the isolated lab.

Current use cases include:

- Nmap TCP SYN scanning
- Network reconnaissance
- Controlled attack simulation

Example:

```bash
sudo nmap -sS -T4 192.168.56.20
```

The resulting traffic is observed by Suricata and analyzed in Splunk.

---

## Endpoint Log Flow

```text
Windows Activity
      |
      v
Windows Event Logs / Sysmon
      |
      v
Splunk Universal Forwarder
      |
      | TCP 9997
      v
Splunk Enterprise
      |
      v
SPL Detection Logic
      |
      v
Scheduled Alert / Investigation
```

---

## Network Log Flow

```text
Kali Linux
      |
      | Reconnaissance Traffic
      v
SOC-LAB Network
      |
      v
Suricata
      |
      | eve.json
      v
Splunk Enterprise
      |
      v
SPL Detection Logic
      |
      v
Scheduled Alert / Investigation
```

---

## Implemented Detection Use Cases

The lab currently contains four implemented security monitoring use cases:

1. Multiple Failed Windows Logins
2. Suspicious PowerShell Discovery Commands
3. Network Reconnaissance / Port Scan
4. Sysmon Network Connection Monitoring

These use cases demonstrate both endpoint and network-based detection workflows.

---

## Network Separation

The virtual machines use a dedicated internal VirtualBox network named `SOC-LAB`.

```text
192.168.56.0/24
```

This provides an isolated environment for controlled security testing without exposing simulated attacks directly to the normal home network.

Additional NAT or management interfaces can be used when Internet access or administrative access is required.

---

## Current Architecture Summary

```text
Kali Linux
    |
    | Recon / Nmap
    v
Suricata --------+
                 |
                 v
            Splunk Enterprise
                 ^
                 |
                 | TCP 9997
                 |
Windows 10 + Sysmon
```

The architecture now provides:

- Endpoint telemetry
- Network telemetry
- Centralized log collection
- SPL-based detection logic
- Scheduled alerting
- Controlled attack simulation
- Investigation-ready security data
