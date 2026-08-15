# Detection: Network Reconnaissance / Port Scan

## Overview

This detection identifies possible network reconnaissance activity based on a single source host contacting a large number of destination ports on the same target within a short time window.

The lab uses Suricata to capture network telemetry and Splunk Enterprise to aggregate and detect port scanning behavior.

---

## Data Source

- **Network Sensor:** Suricata
- **Log File:** `/var/log/suricata/eve.json`
- **Splunk Sourcetype:** `suricata:eve`
- **Splunk Index:** `suricata`

Suricata monitors the internal `SOC-LAB` network and writes network events in JSON format to `eve.json`.

The file is monitored directly by Splunk Enterprise.

---

## Lab Systems

- **Kali Linux Attacker:** `192.168.56.30`
- **Ubuntu SOC Server:** `192.168.56.20`
- **Network:** `SOC-LAB`

The Kali Linux VM was used to generate controlled reconnaissance traffic against the Ubuntu server.

---

## Test Scenario

A TCP SYN scan was executed from Kali Linux:

```bash
sudo nmap -sS -T4 192.168.56.20
```

The scan contacted a large number of destination ports on the Ubuntu server.

Suricata captured the resulting network flows.

---

## Suricata Telemetry

Suricata recorded traffic with:

- Source IP: `192.168.56.30`
- Destination IP: `192.168.56.20`
- Protocol: TCP
- Event Type: `flow`
- Multiple destination ports

### Raw Network Telemetry

![Suricata Nmap Flow](../screenshots/network-reconnaissance/01-suricata-nmap-flow.PNG)

---

## Detection Objective

The goal is to identify a source host contacting many unique destination ports on the same target within a five-minute period.

This behavior can indicate:

- Port scanning
- Service discovery
- Network reconnaissance
- Security testing

The activity is not automatically malicious and requires context.

---

## Detection Query

```spl
index=suricata sourcetype="suricata:eve" event_type="flow"
| bin _time span=5m
| stats dc(dest_port) as unique_ports by _time src_ip dest_ip
| where unique_ports >= 20
| table _time src_ip dest_ip unique_ports
| sort - _time
```

### Detection Logic

The search:

1. Filters for Suricata network flow events.
2. Groups the activity into five-minute time windows.
3. Counts the number of unique destination ports contacted by each source IP against each destination IP.
4. Returns results where at least 20 unique destination ports were contacted.

During the lab test, the Nmap scan contacted approximately 1000 unique destination ports.

---

## Detection Result

Splunk successfully aggregated the Suricata flow data and identified the scanning behavior.

![Splunk Port Scan Detection](../screenshots/network-reconnaissance/02-portscan-detection-splunk.PNG)

---

## Splunk Alert

The detection query was saved as a Splunk alert.

- **Alert Name:** `Network Reconnaissance - Port Scan Detected`
- **Severity:** `Medium`
- **Trigger Condition:** At least one matching result
- **Detection Threshold:** 20 or more unique destination ports within five minutes

The alert was successfully triggered during controlled testing.

![Network Reconnaissance Alert Triggered](../screenshots/network-reconnaissance/03-portscan-alert-triggered.PNG)

---

## Investigation Guidance

When this detection triggers, an analyst should review:

- Which source IP initiated the activity?
- Which system was targeted?
- How many unique destination ports were contacted?
- Was the activity expected or authorized?
- Is the source system known to perform vulnerability scanning?
- Were multiple systems targeted?
- Did additional suspicious activity occur before or after the scan?
- Were any connections successfully established to exposed services?

---

## Potential False Positives

Possible legitimate causes include:

- Vulnerability scanners
- Network inventory tools
- Administrative troubleshooting
- Monitoring systems
- Security testing
- Authorized penetration testing

The detection should therefore be evaluated together with the source host, target system and surrounding activity.

---

## Recommended Response

If the activity is unexpected:

1. Identify and validate the source system.
2. Determine whether the scan was authorized.
3. Review other Suricata events associated with the source IP.
4. Identify which services and ports were targeted.
5. Review authentication and endpoint telemetry on the destination system.
6. Determine whether the scan was followed by exploitation or authentication attempts.
7. Escalate if the activity cannot be explained as legitimate.

---

## Result

The lab successfully demonstrated the following detection workflow:

```text
Kali Linux
192.168.56.30
        |
        | Nmap TCP SYN Scan
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
        | SPL Aggregation
        v
Port Scan Detection
        |
        v
Medium-Severity Alert
```

This use case demonstrates the ability to collect network telemetry, ingest Suricata events into Splunk, develop a behavioral port-scan detection and generate an alert from controlled reconnaissance activity.
