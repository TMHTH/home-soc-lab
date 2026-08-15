# Suricata Setup

## Overview

Suricata is used as the network intrusion detection and telemetry source for the Home SOC Lab.

It runs on the Ubuntu SOC server and monitors the internal `SOC-LAB` network.

The generated network telemetry is written to `eve.json` and ingested directly by Splunk Enterprise.

---

## System

- **Host:** SOC-SERVER
- **Operating System:** Ubuntu Server
- **SOC-LAB IP:** `192.168.56.20`
- **Monitored Interface:** `enp0s8`
- **Log File:** `/var/log/suricata/eve.json`

---

## Installation

Suricata was installed using the Ubuntu package manager:

```bash
sudo apt update
sudo apt install suricata -y
```

The installed version can be checked with:

```bash
suricata --build-info
```

---

## Rule Update

Suricata rules were downloaded and updated using:

```bash
sudo suricata-update
```

The resulting rule file is stored under:

```text
/var/lib/suricata/rules/suricata.rules
```

---

## Interface Configuration

The default Suricata configuration referenced an interface named:

```text
eth0
```

The SOC-LAB network uses:

```text
enp0s8
```

The `af-packet` section in:

```text
/etc/suricata/suricata.yaml
```

was therefore updated to:

```yaml
af-packet:
  - interface: enp0s8
```

---

## Configuration Validation

The Suricata configuration was validated before restarting the service:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

After successful validation:

```bash
sudo systemctl restart suricata
```

Service status can be checked with:

```bash
sudo systemctl status suricata --no-pager -l
```

---

## Event Output

Suricata writes structured JSON events to:

```text
/var/log/suricata/eve.json
```

The file can be monitored directly with:

```bash
sudo tail -f /var/log/suricata/eve.json
```

Events associated with the Kali attacker VM can be filtered with:

```bash
sudo tail -f /var/log/suricata/eve.json | grep '192.168.56.30'
```

---

## Splunk Integration

Splunk Enterprise monitors the Suricata event file directly.

The following input was configured on the Ubuntu Splunk server:

```ini
[monitor:///var/log/suricata/eve.json]
disabled = 0
sourcetype = suricata:eve
index = suricata
```

A dedicated Splunk index named:

```text
suricata
```

was created for the network telemetry.

After changing the input configuration, Splunk was restarted:

```bash
sudo /opt/splunk/bin/splunk restart
```

---

## Validation

Suricata events can be validated in Splunk with:

```spl
index=suricata sourcetype="suricata:eve"
```

Traffic generated from Kali can be filtered with:

```spl
index=suricata sourcetype="suricata:eve" src_ip="192.168.56.30"
```

This confirms successful network telemetry collection from Suricata into Splunk Enterprise.
