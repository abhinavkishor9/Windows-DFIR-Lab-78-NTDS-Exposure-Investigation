# Investigation Notes — Lab 78

## Investigation

**Lab:** 78 — NTDS.dit Exposure and Access Investigation

**Investigation Workspace:**

~~~text
C:\Users\Dell\Desktop\Lab78-NTDS-Investigation
~~~

Workspace structure:

~~~text
Lab78-NTDS-Investigation
├── Evidence
├── Logs
├── Notes
└── Timeline
~~~

---

## Investigation Objective

Determine whether the Windows endpoint contains an `NTDS.dit` database or supporting evidence indicating that an NTDS database was copied, staged, accessed, or otherwise targeted.

The investigation was performed without extracting, dumping, copying, or exposing credential material.

---

## Initial Host Validation

### Host

~~~text
DESKTOP-9MMM37V
~~~

### Operating System

~~~text
Microsoft Windows 11 Pro
Version: 10.0.26200
Build: 26200
~~~

### Domain

~~~text
WORKGROUP
~~~

### Domain Role

~~~text
0
~~~

The endpoint is therefore a standalone workstation rather than a Domain Controller.

### Investigation Significance

`NTDS.dit` is normally associated with Active Directory Domain Controllers.

Because this endpoint is not a Domain Controller, the expected baseline is that:

~~~text
C:\Windows\NTDS\ntds.dit
~~~

should not exist.

This baseline was established before interpreting any filesystem artifacts.

---

## Standard NTDS Location

Command:

~~~powershell
Test-Path "C:\Windows\NTDS\ntds.dit"
~~~

Result:

~~~text
False
~~~

The directory listing of:

~~~text
C:\Windows\NTDS
~~~

also returned no relevant contents.

### Finding

No `ntds.dit` database was identified at the standard Windows NTDS location.

---

## Filesystem Search

A targeted search was performed for:

~~~text
ntds.dit
~~~

No matching file was returned.

A broader filesystem search produced several results containing NTDS-related names or backup extensions.

Notable result:

~~~text
C:\NTDSLab
~~~

This result requires contextual interpretation.

The available evidence only establishes that a filesystem object named `NTDSLab` was present. It does not establish that it contains `ntds.dit`, an Active Directory database, credential material, or malicious tooling.

Other results included:

~~~text
C:\Windows\System32\ntdsapi.dll
C:\Windows\SysWOW64\ntdsapi.dll
~~~

These are Windows system components and should not be interpreted as an `NTDS.dit` database.

Multiple unrelated `.bak` files were also identified.

### Assessment

The broad search produced NTDS-related filenames, but no confirmed `ntds.dit` database.

---

## VSS Investigation

Commands:

~~~powershell
Get-CimInstance Win32_ShadowCopy |
Select-Object ID, VolumeName, InstallDate, State
~~~

and:

~~~powershell
vssadmin list shadows
~~~

Result:

~~~text
No items found that satisfy the query.
~~~

### Finding

No existing Volume Shadow Copy snapshots were identified through the checks performed.

### Investigation Significance

VSS can be relevant when investigating access to protected files and backup-related activity.

However, the absence of an existing shadow copy does not prove that VSS was never used historically.

---

## Application Event Log

The Application log search returned:

~~~text
TimeCreated: 14-09-2026 07:21:44
Event ID: 8224
Provider: VSS
~~~

Message:

~~~text
The VSS service is shutting down due to idle timeout.
~~~

### Interpretation

This demonstrates VSS service activity but does not establish that an NTDS database was accessed.

No direct NTDS reference was established from this event.

---

## System Event Log

The System log search returned Event ID 1 events from:

~~~text
Microsoft-Windows-Kernel-General
~~~

The messages concerned system time changes.

### Interpretation

These events do not provide evidence of NTDS access.

They are nevertheless relevant to timeline interpretation because timestamp changes can affect correlation between different telemetry sources.

---

## Sysmon Investigation

Sysmon Event ID 1 process creation telemetry was queried.

The returned results demonstrated repeated process creation events around:

~~~text
16-09-2026 07:43–07:45
~~~

The displayed output contained:

~~~text
Process Create:...
~~~

However, the extracted output did not expose the full process image or command-line information required to attribute an NTDS operation to a specific process.

### Finding

Sysmon process creation telemetry was available.

### Limitation

The displayed results did not establish a confirmed process accessing `ntds.dit`.

Therefore:

~~~text
Sysmon process telemetry available
≠
NTDS access confirmed
~~~

---

## Wazuh Investigation

A Wazuh event was examined from:

~~~text
wazuh-alerts-4.x-2026.09.16
~~~

Host:

~~~text
DESKTOP-9MMM37V
~~~

Event:

~~~text
Event ID: 16384
Provider: Software Protection Platform Service
Channel: Application
~~~

The event concerned scheduling the Software Protection service for restart.

### Interpretation

This event is unrelated to NTDS activity.

It was therefore excluded from the NTDS evidence chain.

### DFIR Lesson

A SIEM event associated with the investigated host does not automatically become evidence for the investigation.

The event must be relevant to the investigative question.

---

## Evidence Correlation

The evidence currently supports the following chain:

~~~text
Standalone Windows workstation
        ↓
NTDS database not expected
        ↓
Standard NTDS path absent
        ↓
Targeted ntds.dit search produced no result
        ↓
Broad search produced NTDS-related filenames
        ↓
No confirmed NTDS database identified
        ↓
No VSS snapshots present
        ↓
No confirmed NTDS process activity established
        ↓
Wazuh event examined was unrelated
~~~

There is no supported evidence chain showing:

~~~text
NTDS.dit
    ↓
access
    ↓
copy/staging
    ↓
credential access
~~~

---

## Confirmed

- The endpoint is Windows 11 Pro.
- The hostname is `DESKTOP-9MMM37V`.
- The system is in `WORKGROUP`.
- `DomainRole` is `0`.
- The standard `C:\Windows\NTDS\ntds.dit` path did not contain the database.
- The targeted filesystem search did not identify `ntds.dit`.
- No VSS shadow copies were reported.
- Sysmon process creation telemetry was available.
- The examined Wazuh event was a Software Protection Platform event.

---

## Potential Leads

### `C:\NTDSLab`

This object should be considered a lead rather than a conclusion.

Additional investigation would be required before assigning significance.

Questions would include:

- Is it a directory or file?
- What is inside it?
- When was it created?
- Which process created or modified it?
- Does it contain an NTDS database?
- Is it related to the lab itself?
- Is there supporting process or security telemetry?

The evidence currently provided does not answer those questions.

---

## Not Confirmed

The investigation did not confirm:

- Presence of `ntds.dit`
- NTDS database copying
- NTDS database staging
- NTDS database access
- Credential extraction
- VSS-based NTDS access
- Malicious NTDS tooling
- NTDS-related Wazuh detection

---

## Investigation Limitations

The investigation relied on:

- Current filesystem state
- Available Windows event logs
- Available Sysmon telemetry
- Current VSS state
- Available Wazuh telemetry

The absence of an artifact or event should therefore be interpreted within the visibility provided by those sources.

A historical activity may not be recoverable if the relevant artifact or telemetry has already been removed, overwritten, or was never collected.

---

## Analyst Conclusion

The available evidence does not establish `NTDS.dit` exposure or access on this endpoint.

The most important result of the investigation was not a malicious artifact, but the demonstration of evidence validation.

The endpoint role was established first, the expected NTDS location was checked, filesystem searches were performed, and supporting telemetry was reviewed before reaching an assessment.

**Assessment: No confirmed NTDS exposure or access identified from the collected evidence.**
