# Detection: Suspicious PowerShell Discovery Commands

## Overview

This detection identifies selected discovery commands launched from Windows PowerShell.

The goal is to detect PowerShell activity that may indicate host or account discovery during an investigation or attack.

The detection uses Windows Security Event ID `4688`, which records process creation events.

---

## Data Source

**Log Source:** Windows Security Event Log

**Event ID:** `4688`

Event ID `4688` is generated when a new process is created.

Windows Process Creation Auditing was enabled on the monitored endpoint so that process creation events are recorded in the Windows Security log.

Command-line logging was also enabled so that Splunk can display the command line associated with newly created processes.

---

## Detection Objective

The detection looks for selected discovery commands launched by PowerShell.

The test focused on:

```text
whoami
ipconfig
net user
```

These commands can be used legitimately by administrators, but they are also commonly useful during system and account discovery.

For this reason, the detection should be treated as an investigation trigger rather than proof of malicious activity.

---

## Process Creation Logging

The Windows endpoint was configured to include process command-line information in Event ID `4688`.

This made it possible to identify:

* The parent process
* The newly created process
* The process command line
* The affected Windows host
* The event timestamp

During testing, PowerShell appeared as the creator process.

Example:

```text
Creator Process:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

The generated child-process command line could then be inspected in Splunk.

---

## Detection Query

The final SPL query searches for process creation events where PowerShell is the parent process and the created process matches selected discovery activity.

```spl
index=* EventCode=4688
Erstellerprozessname="*powershell.exe"
(
  Prozessbefehlszeile="*whoami*" OR
  Prozessbefehlszeile="*ipconfig*" OR
  Prozessbefehlszeile="*net.exe*" OR
  Prozessbefehlszeile="*net1.exe*"
)
| table _time host Erstellerprozessname Prozessbefehlszeile
| sort - _time
```

---

## Detection Logic

The search performs the following steps:

1. Searches Windows Security Event ID `4688`.
2. Filters for events where PowerShell is the creator process.
3. Searches the process command line for selected discovery commands.
4. Displays the timestamp, affected host, parent process, and command line.

---

## Test Scenario

The following commands were executed manually from PowerShell inside the isolated Windows lab endpoint:

```powershell
whoami
ipconfig
net user
```

The activity generated process creation events that were forwarded to Splunk using the Splunk Universal Forwarder.

Splunk successfully identified the corresponding process command lines.

During testing, `net user` appeared through Windows processes such as:

```text
net.exe
net1.exe
```

The detection was adjusted to account for this behavior.

---

## Splunk Alert

The detection query was saved as a scheduled Splunk alert.

**Alert Name:**

```text
PowerShell Discovery Commands Detected
```

**Severity:**

```text
Medium
```

The alert triggers when the scheduled search returns at least one matching event.

The alert was successfully triggered during testing.

---

## Investigation Guidance

When this detection triggers, an analyst should review:

* Which endpoint generated the event?
* Which command was executed?
* Was PowerShell the expected parent process?
* Which user was active at the time?
* Were several discovery commands executed within a short period?
* Did additional suspicious processes run before or after the event?
* Is the activity consistent with expected administrative behavior?

---

## Potential False Positives

Possible legitimate causes include:

* System administration
* Troubleshooting
* Helpdesk activity
* Network diagnostics
* User or administrator account checks
* Security testing

Because the individual commands are not inherently malicious, surrounding context is important.

---

## Recommended Response

If the activity is unexpected:

1. Identify the user associated with the PowerShell session.
2. Review surrounding Event ID `4688` events.
3. Check for additional PowerShell activity.
4. Review authentication activity on the affected endpoint.
5. Look for unusual child processes or command sequences.
6. Validate whether the activity was authorized.
7. Escalate if additional suspicious behavior is identified.

---

## Evidence

Relevant screenshots are stored under:

```text
screenshots/suspicious-powershell/
```

Evidence:

```text
01-discovery-commands-detected.png
02-powershell-alert-triggered.png
```

---

## Result

The lab successfully demonstrated the following workflow:

```text
PowerShell Activity
        ↓
Windows Event ID 4688
        ↓
Process Command-Line Logging
        ↓
Splunk Universal Forwarder
        ↓
Splunk Enterprise
        ↓
Discovery Command Detection
        ↓
Medium-Severity Alert
```

This validates the ability of the Home SOC Lab to monitor PowerShell-launched process activity and detect selected discovery commands using Windows Security telemetry.
