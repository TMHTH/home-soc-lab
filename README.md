# Home SOC Lab

A virtualized Security Operations Center lab for hands-on practice with Windows event monitoring, Splunk, detection engineering, and incident investigation.

## Project Status

**In Progress**

### Completed

* VirtualBox lab environment
* Windows SOC client
* Ubuntu SOC server
* Isolated SOC network
* Splunk Enterprise installation
* Splunk Universal Forwarder configuration
* Centralized Windows Event Log collection
* First detection use case: repeated failed Windows logins
* Automated Splunk alerting

### Planned

* Suspicious PowerShell activity detection
* Network reconnaissance detection
* Additional Windows telemetry
* Incident investigation documentation
* Kali Linux attacker VM
* Sysmon integration

## Lab Architecture

| System             | Role               | IP Address      |
| ------------------ | ------------------ | --------------- |
| Windows SOC Client | Monitored endpoint | `192.168.56.10` |
| Ubuntu SOC Server  | Splunk Enterprise  | `192.168.56.20` |

Windows event logs are collected with the Splunk Universal Forwarder and sent to Splunk Enterprise over TCP port `9997`.

```text
Windows SOC Client
192.168.56.10
        |
        | Windows Event Logs
        | Splunk Universal Forwarder
        | TCP 9997
        v
Ubuntu SOC Server
192.168.56.20
        |
        v
Splunk Enterprise
        |
        v
Detection & Investigation
```

More details are available in:

[Lab Architecture](architecture/lab-overview.md)

## Detection Use Cases

### 1. Multiple Failed Windows Logins

The first detection use case monitors Windows Security Event ID `4625` for repeated failed authentication attempts.

The current rule detects at least five failed logins within a five-minute window and generates a Splunk alert when the threshold is exceeded.

[View Detection Documentation](detections/failed-logins-5min.md)

## Repository Structure

```text
home-soc-lab/
├── README.md
├── architecture/
│   └── lab-overview.md
├── setup/
├── detections/
│   └── failed-logins-5min.md
├── incidents/
└── screenshots/
```

## Current Skills Practiced

* Splunk Enterprise
* Splunk Universal Forwarder
* SPL
* Windows Event Logs
* Windows Security Event Analysis
* Detection Engineering
* Alert Configuration
* Security Monitoring
* Linux Server Administration
* TCP/IP Networking
* VirtualBox

## Screenshots

Current evidence includes:

* Raw Windows Event ID 4625 events
* Aggregated failed-login activity
* Five-minute detection results
* Successfully triggered Splunk alert

Screenshots are stored in the `screenshots/` directory.

## Project Goal

The goal of this lab is to build practical experience with the complete SOC workflow:

```text
Telemetry
   ↓
Log Collection
   ↓
Detection
   ↓
Alerting
   ↓
Investigation
   ↓
Incident Response
```

The environment will gradually be expanded with additional detection and investigation scenarios.

## Disclaimer

This project was created in an isolated home lab environment for educational and defensive security purposes.

All testing is performed against systems owned and controlled by the lab operator.
