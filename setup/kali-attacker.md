# Kali Linux Attacker VM Setup

## Overview

The Kali Linux VM is used as a controlled attacker and reconnaissance system inside the isolated Home SOC Lab.

Its purpose is to generate repeatable security-testing activity that can be observed by Suricata and Splunk.

---

## System

- **Operating System:** Kali Linux
- **Hostname:** `kali-attacker`
- **SOC-LAB IP:** `192.168.56.30`
- **Internal Network:** `SOC-LAB`
- **Lab Subnet:** `192.168.56.0/24`

---

## VirtualBox Resources

The VM was configured with approximately:

- **CPU:** 2 vCPUs
- **Memory:** 4 GB RAM
- **Disk:** 40 GB dynamically allocated

---

## Network Configuration

The Kali VM uses two VirtualBox network adapters.

### Adapter 1

```text
NAT
```

Used for Internet access and system updates.

### Adapter 2

```text
Internal Network: SOC-LAB
```

Used for isolated security testing against the lab systems.

The SOC-LAB interface uses:

```text
192.168.56.30/24
```

---

## Lab Connectivity

The Kali VM can communicate with the SOC server:

```text
192.168.56.20
```

Connectivity was validated using:

```bash
ping 192.168.56.20
```

The Windows endpoint is located at:

```text
192.168.56.10
```

---

## Reconnaissance Tooling

Kali Linux provides Nmap for controlled network reconnaissance.

A TCP SYN scan against the Ubuntu SOC server was performed with:

```bash
sudo nmap -sS -T4 192.168.56.20
```

The scan generated network flow activity across a large number of destination ports.

---

## Detection Workflow

The Kali VM is used to generate controlled activity through the following workflow:

```text
Kali Linux
192.168.56.30
        |
        | Nmap / Reconnaissance
        v
SOC-LAB Network
        |
        v
Ubuntu SOC Server
192.168.56.20
        |
        v
Suricata
        |
        v
Splunk Enterprise
```

This provides a reproducible attacker source for network-based detection testing without exposing external systems.
