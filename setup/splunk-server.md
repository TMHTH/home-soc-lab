# Splunk Server Setup

## Overview

The SOC server runs Ubuntu Server and hosts Splunk Enterprise as the central log collection and analysis platform.

## System Role

* Operating System: Ubuntu Server
* Role: SOC / Log Server
* SOC-LAB IP: `192.168.56.20`
* Splunk Web Port: `8000`
* Splunk Receiving Port: `9997`

## Network Configuration

The Splunk server uses a static IP address on the isolated SOC-LAB network:

```text
192.168.56.20/24
```

The SOC-LAB interface is used for communication between the monitored Windows endpoint and the Splunk server.

A separate management interface can be used for administrative access.

## Base System Preparation

The Ubuntu server was updated and basic administration tools were installed:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl wget git unzip net-tools htop
```

## Splunk Enterprise Installation

Splunk Enterprise was installed using the Linux `.deb` package.

Example installation:

```bash
sudo dpkg -i splunk-*.deb
```

Splunk is installed under:

```text
/opt/splunk
```

## Starting Splunk

Splunk can be started manually with:

```bash
sudo /opt/splunk/bin/splunk start
```

The current status can be checked with:

```bash
sudo /opt/splunk/bin/splunk status
```

Automatic startup was enabled so that Splunk starts when the Ubuntu server boots.

## Splunk Web

Splunk Web is available on port `8000`.

Example:

```text
http://<management-ip>:8000
```

## Receiving Forwarded Data

Splunk Enterprise was configured to receive forwarded data on TCP port:

```text
9997
```

The listening port can be verified on Ubuntu with:

```bash
sudo ss -tulpn | grep :9997
```

## Current Data Flow

```text
Windows SOC Client
        |
        | Splunk Universal Forwarder
        | TCP 9997
        v
Ubuntu SOC Server
        |
        v
Splunk Enterprise
```

## Result

The Splunk server is operational and successfully receives Windows Event Log data from the monitored Windows endpoint.
