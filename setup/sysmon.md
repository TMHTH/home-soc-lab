# Microsoft Sysmon Setup

## Overview

Microsoft Sysmon is installed on the Windows SOC client to provide additional endpoint telemetry beyond the default Windows Event Logs.

The Sysmon Operational log is collected by the Splunk Universal Forwarder and forwarded to Splunk Enterprise.

---

## System

- **Host:** SOC-CLIENT
- **Operating System:** Windows 10
- **SOC-LAB IP:** `192.168.56.10`
- **Sysmon Event Log:** `Microsoft-Windows-Sysmon/Operational`

---

## Installation Files

Sysmon was extracted to:

```text
C:\Tools\Sysmon
```

The executable used was:

```text
Sysmon64.exe
```

The active Sysmon schema version was:

```text
4.91
```

---

## Sysmon Configuration

A custom configuration file was created at:

```text
C:\Tools\Sysmon\sysmonconfig.xml
```

The configuration enabled telemetry for:

- Process creation
- Network connections
- File creation
- Registry activity

The configuration uses SHA-256 hashing.

Example structure:

```xml
<Sysmon schemaversion="4.91">
  <HashAlgorithms>sha256</HashAlgorithms>
  <EventFiltering>

    <ProcessCreate onmatch="exclude">
    </ProcessCreate>

    <NetworkConnect onmatch="exclude">
    </NetworkConnect>

    <FileCreate onmatch="exclude">
    </FileCreate>

    <RegistryEvent onmatch="exclude">
    </RegistryEvent>

  </EventFiltering>
</Sysmon>
```

---

## Sysmon Installation

PowerShell was opened as Administrator.

The installation was performed with:

```powershell
cd C:\Tools\Sysmon
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

The Sysmon service was validated with:

```powershell
Get-Service Sysmon64
```

---

## Local Event Validation

Sysmon events were verified locally using:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
```

Relevant events used in the lab include:

```text
Event ID 1 - Process Creation
Event ID 3 - Network Connection
```

---

## Splunk Universal Forwarder Configuration

The following input was added to:

```text
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
```

The Splunk Universal Forwarder was then restarted:

```powershell
Restart-Service SplunkForwarder
```

---

## Forwarder Permission Issue

The Universal Forwarder initially ran under:

```text
NT SERVICE\SplunkForwarder
```

Although the Sysmon log existed and contained events, the Forwarder was unable to subscribe to the channel.

The Splunk log reported errors indicating that it could not subscribe to:

```text
Microsoft-Windows-Sysmon/Operational
```

For the isolated lab environment, the Splunk Forwarder service account was changed to:

```text
LocalSystem
```

The service was stopped:

```powershell
Stop-Service SplunkForwarder
```

The account was changed with:

```powershell
sc.exe config SplunkForwarder obj= LocalSystem
```

The Forwarder was then started again:

```powershell
Start-Service SplunkForwarder
```

The active service account was verified using:

```powershell
Get-CimInstance Win32_Service -Filter "Name='SplunkForwarder'" |
Select-Object Name, StartName, State
```

After this change, Sysmon events were successfully forwarded to Splunk.

---

## Splunk Validation

Process creation telemetry can be validated with:

```spl
index=* Sysmon EventCode=1
| table _time host Image CommandLine ParentImage User
| sort - _time
```

Network connection telemetry can be validated with:

```spl
index=* Sysmon EventCode=3
| table _time host Image DestinationIp DestinationPort Protocol User
| sort - _time
```

---

## Result

The Sysmon integration provides additional endpoint visibility including:

- Detailed process creation
- Full command-line information
- Parent process relationships
- Process-aware network connections
- File activity
- Registry activity

The resulting telemetry is forwarded to Splunk Enterprise and used for endpoint detection and investigation.
