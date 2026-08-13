# Lab Architecture

## Overview

The Home SOC Lab is a virtualized security monitoring environment designed to simulate a small Security Operations Center workflow.

The current architecture separates the monitored Windows endpoint from the Splunk server while keeping both systems inside an isolated internal lab network.

## Systems

| System     | Operating System | Role                           | IP Address      |
| ---------- | ---------------- | ------------------------------ | --------------- |
| SOC-CLIENT | Windows 10       | Monitored endpoint             | `192.168.56.10` |
| SOC-SERVER | Ubuntu Server    | Splunk Enterprise / Log Server | `192.168.56.20` |

## Network Layout

```text
                    Windows Host
                        |
                    VirtualBox
                        |
               SOC-LAB 192.168.56.0/24
                        |
          +-------------+-------------+
          |                           |
          v                           v
     SOC-CLIENT                  SOC-SERVER
     Windows 10                  Ubuntu Server
   192.168.56.10                192.168.56.20
          |                           |
          | Splunk Universal          |
          | Forwarder                 |
          |                           |
          +------- TCP 9997 --------->|
                                      |
                               Splunk Enterprise
                                      |
                                      v
                           Detection & Investigation
```

## Windows Endpoint

The Windows endpoint acts as the monitored system in the lab.

Relevant Windows Event Logs are collected using the Splunk Universal Forwarder.

Currently monitored logs include:

* Security
* System
* Application

The endpoint forwards collected events to the Splunk server over TCP port `9997`.

## Splunk Server

The Ubuntu server hosts Splunk Enterprise and acts as the central log collection and analysis platform.

Its responsibilities include:

* Receiving Windows Event Logs
* Indexing security events
* Running SPL searches
* Detecting suspicious activity
* Generating alerts
* Supporting incident investigation

## Log Flow

```text
Windows Activity
      |
      v
Windows Event Log
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
Alert / Investigation
```

## Network Separation

The virtual machines use a dedicated internal VirtualBox network for SOC traffic.

This provides an isolated environment for controlled security testing without exposing simulated attacks directly to the normal home network.

A separate management interface can be used for administrative access to the Ubuntu server.

## Planned Expansion

Future components may include:

* Kali Linux attacker VM
* Network reconnaissance simulations
* Suspicious PowerShell detection
* Additional Windows telemetry
* Sysmon
* Further incident response scenarios
