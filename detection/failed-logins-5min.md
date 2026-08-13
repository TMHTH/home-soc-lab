# Detection: Multiple Failed Windows Logins

## Overview

This detection identifies repeated failed Windows authentication attempts within a short time period.

Multiple failed logins can indicate:

* Password guessing
* Brute-force activity
* Incorrectly configured credentials
* User error
* Potential account compromise attempts

The detection is based on Windows Security Event ID `4625`.

---

## Data Source

**Log Source:** Windows Security Event Log

**Event ID:** `4625`

Event ID `4625` is generated when a Windows logon attempt fails.

The Windows Security logs are collected using the Splunk Universal Forwarder and forwarded to Splunk Enterprise over TCP port `9997`.

---

## Detection Objective

The goal is to detect at least:

```text
5 failed login attempts within 5 minutes
```

on the monitored Windows endpoint.

This threshold is intentionally simple and designed for the lab environment.

In a production environment, the threshold would need to be tuned based on normal authentication behavior and potential false positives.

---

## Initial Search

The raw failed-login events can be identified with:

```spl
index=* EventCode=4625
```

This confirms that failed authentication attempts are successfully collected by Splunk.

---

## Aggregated Detection

The events are grouped into 5-minute time windows and counted per host and source network address.

```spl
index=* EventCode=4625
| bin _time span=5m
| stats count by _time host Quellnetzwerkadresse
| where count >= 5
| sort - _time
```

### Detection Logic

The search performs the following steps:

1. Searches for Windows Event ID `4625`.
2. Groups events into 5-minute time buckets.
3. Counts failed authentication events by host and source network address.
4. Returns only results where at least 5 failed logins occurred.

---

## Test Scenario

The detection was tested by intentionally entering an incorrect password multiple times on the Windows lab endpoint.

This generated several Windows Security Event ID `4625` events.

The events were successfully:

```text
Generated on Windows
        ↓
Collected by the Universal Forwarder
        ↓
Sent to Splunk Enterprise
        ↓
Grouped into a 5-minute window
        ↓
Matched against the threshold
        ↓
Detected by the SPL query
```

During the local test, the source network address appeared as:

```text
127.0.0.1
```

This is expected because the failed authentication attempts were generated locally on the monitored Windows endpoint.

---

## Splunk Alert

The detection query was saved as a Splunk alert.

**Alert Name:**

```text
Multiple Failed Logins - 5 in 5 Minutes
```

**Trigger Condition:**

The alert triggers when the search returns at least one result matching the detection threshold.

The alert was successfully triggered during testing.

---

## Investigation Guidance

When this alert triggers, a SOC analyst should investigate:

* Which endpoint generated the event?
* What source network address is associated with the failed attempts?
* How many authentication failures occurred?
* Over what time period did the failures occur?
* Which user account was targeted?
* Did a successful login occur shortly after the failed attempts?
* Are there additional suspicious authentication events?
* Is the activity expected or potentially malicious?

---

## Potential False Positives

Possible benign causes include:

* User repeatedly entering an incorrect password
* Old credentials stored in an application or service
* Misconfigured scheduled tasks
* Authentication problems after a password change
* Automated systems using outdated credentials

For this reason, the detection should be treated as an investigation trigger rather than proof of malicious activity.

---

## Recommended Response

If the activity appears suspicious:

1. Identify the affected user account.
2. Review successful logins following the failed attempts.
3. Investigate the source system or IP address.
4. Validate the activity with the affected user.
5. Reset credentials if account compromise is suspected.
6. Lock or disable the account if necessary.
7. Review surrounding endpoint activity for additional indicators.
8. Increase monitoring of the affected account or endpoint.

---

## Evidence

The following screenshots document the detection workflow:

* `01-failed-login-events.png`
* `02-bruteforce-summary.png`
* `03-bruteforce-summary-details.png`
* `04-bruteforce-5min-detection.png`
* `05-bruteforce-alert-triggered.png`

---

## Result

The lab successfully demonstrated an end-to-end authentication monitoring workflow:

```text
Failed Windows Login
        ↓
Event ID 4625
        ↓
Splunk Universal Forwarder
        ↓
Splunk Enterprise
        ↓
5-Minute Aggregation
        ↓
Threshold >= 5 Events
        ↓
Splunk Alert
```

This confirms that the Home SOC Lab can collect Windows authentication telemetry, apply detection logic, and generate alerts for suspicious activity.
