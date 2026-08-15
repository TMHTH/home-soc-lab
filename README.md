# Home SOC Lab

A virtualized Security Operations Center lab for hands-on practice with Windows event monitoring, Sysmon, Suricata, Splunk, detection engineering, alerting, and security investigation.

## Project Status

**Core Lab Completed**

### Implemented

* VirtualBox lab environment
* Windows SOC client
* Ubuntu SOC server
* Kali Linux attacker VM
* Isolated `SOC-LAB` network
* Splunk Enterprise
* Splunk Universal Forwarder
* Centralized Windows Event Log collection
* Microsoft Sysmon integration
* Suricata network monitoring
* Custom Suricata index in Splunk
* SPL-based detection logic
* Scheduled Splunk alerting
* Detection documentation and screenshots

### Detection Use Cases

* Multiple failed Windows logins
* Suspicious PowerShell discovery commands
* Network reconnaissance / port scanning
* Sysmon network connection monitoring

---

## Lab Architecture

| System | Role | IP Address |
| --- | --- | --- |
| Windows SOC Client | Monitored endpoint / Sysmon telemetry | `192.168.56.10` |
| Ubuntu SOC Server | Splunk Enterprise / Suricata IDS | `192.168.56.20` |
| Kali Linux Attacker | Controlled reconnaissance / attack simulation | `192.168.56.30` |

The virtual machines communicate through an isolated VirtualBox internal network:

```text
SOC-LAB
192.168.56.0/24
```

### Endpoint Telemetry

```text
Windows SOC Client
192.168.56.10
        |
        | Windows Event Logs
        | Microsoft Sysmon
        v
Splunk Universal Forwarder
        |
        | TCP 9997
        v
Ubuntu SOC Server
192.168.56.20
        |
        v
Splunk Enterprise
        |
        v
Detection & Alerting
```

### Network Telemetry

```text
Kali Linux
192.168.56.30
        |
        | Nmap / Reconnaissance
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
Detection & Alerting
```

More details:

[View Lab Architecture](architecture/lab-overview.md)

---

## Detection Use Cases

### 1. Multiple Failed Windows Logins

Monitors Windows Security Event ID `4625` for repeated failed authentication attempts.

The detection identifies at least five failed logins within a five-minute window and generates a Splunk alert when the threshold is exceeded.

[View Detection Documentation](detection/failed-logins-5min.md)

---

### 2. Suspicious PowerShell Discovery Commands

Monitors Windows process creation events using Event ID `4688`.

The detection identifies selected discovery commands launched from PowerShell, including:

```text
whoami
ipconfig
net user
```

Command-line logging provides visibility into the processes started from PowerShell.

[View Detection Documentation](detection/suspicious-powershell.md)

---

### 3. Network Reconnaissance / Port Scan

Uses Suricata network flow telemetry generated during controlled Nmap scanning from the Kali Linux attacker VM.

The SPL detection identifies a source host contacting at least 20 unique destination ports on the same target within a five-minute window.

During testing, the Nmap scan contacted approximately 1000 unique ports.

[View Detection Documentation](detection/network-reconnaissance.md)

---

### 4. Sysmon Network Connection Monitoring

Microsoft Sysmon provides additional endpoint telemetry beyond standard Windows Event Logs.

Sysmon Event ID `3` is used to identify process-aware network connections.

The lab detection monitors a controlled outbound connection from the Windows endpoint to the SOC server on TCP port `8000`.

[View Detection Documentation](detection/sysmon-network-connection.md)

---

## Repository Structure

```text
home-soc-lab/
├── architecture/
│   └── lab-overview.md
│
├── detection/
│   ├── failed-logins-5min.md
│   ├── network-reconnaissance.md
│   ├── suspicious-powershell.md
│   └── sysmon-network-connection.md
│
├── screenshots/
│   ├── failed-logins/
│   │   ├── 01-failed-login-events.PNG
│   │   ├── 04-bruteforce-5min-detection.PNG
│   │   └── 05-bruteforce-alert-triggered.PNG
│   │
│   ├── network-reconnaissance/
│   │   ├── 01-suricata-nmap-flow.PNG
│   │   ├── 02-portscan-detection-splunk.PNG
│   │   └── 03-portscan-alert-triggered.PNG
│   │
│   ├── suspicious-powershell/
│   │   ├── 01-discovery-commands-detected.png
│   │   └── 02-powershell-alert-triggered.PNG
│   │
│   └── sysmon/
│       ├── 01-sysmon-process-events.PNG
│       ├── 02-sysmon-network-connection.PNG
│       └── 03-sysmon-network-alert-triggered.PNG
│
├── setup/
        ├── kali-attacker.md
        ├── splunk-server.md
        ├── suricata.md
        ├── sysmon.md
        └── windows-forwarder.md
│
└── README.md
```

---

## Technologies Used

* Splunk Enterprise
* Splunk Universal Forwarder
* SPL
* Microsoft Sysmon
* Suricata IDS
* Windows Event Logs
* Kali Linux
* Nmap
* Ubuntu Server
* Windows 10
* VirtualBox
* TCP/IP Networking

---

## Skills Practiced

* Security monitoring
* SIEM administration
* Log collection and ingestion
* Windows event analysis
* Endpoint telemetry analysis
* Network telemetry analysis
* Detection engineering
* SPL query development
* Threshold-based detection
* Scheduled alert configuration
* Process analysis
* Network reconnaissance detection
* False-positive analysis
* Investigation guidance
* Linux server administration
* Virtual network configuration

---

## Detection Workflow

The lab demonstrates an end-to-end SOC monitoring workflow:

```text
Controlled Activity
        |
        v
Endpoint / Network Telemetry
        |
        v
Log Collection
        |
        v
Splunk Enterprise
        |
        v
SPL Detection Logic
        |
        v
Scheduled Alert
        |
        v
Investigation
```

---

## Evidence

Each detection use case includes screenshots showing relevant stages such as:

* Raw telemetry
* SPL detection results
* Triggered Splunk alerts

Screenshots are stored under the corresponding subdirectories in:

```text
screenshots/
```

---

## Project Goal

The goal of this project is to build practical experience with technologies and workflows commonly used in Security Operations Center environments.

The project focuses on understanding the complete monitoring lifecycle rather than simply deploying security tools:

```text
Telemetry
   ↓
Collection
   ↓
Analysis
   ↓
Detection
   ↓
Alerting
   ↓
Investigation
```

The lab combines endpoint and network telemetry with controlled security testing to provide reproducible detection scenarios.

---

## Possible Future Improvements

Potential future extensions include:

* MITRE ATT&CK mapping
* Splunk dashboards
* Additional Sysmon-based detections
* Multi-event incident investigations
* Additional Suricata detection scenarios
* Detection tuning and allowlisting

---

## Disclaimer

This project was created in an isolated home lab environment for educational and defensive security purposes.

All testing is performed against systems owned and controlled by the lab operator.
