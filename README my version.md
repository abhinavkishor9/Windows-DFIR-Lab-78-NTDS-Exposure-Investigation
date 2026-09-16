# Windows-DFIR-Lab-78-NTDS-Exposure-Investigation
## Overview
NTDS.dit is the Active Directory database used by a Domain Controller to store directory information, including objects, attributes, and credential-related data.

From a DFIR/SOC perspective, the important question is not simply:

“Does ntds.dit exist?”

The more useful question is:

“Why would an NTDS-related artifact exist on this endpoint, and is there evidence that it was accessed, copied, backed up, or otherwise targeted?”

A proper investigation should first establish whether the endpoint is actually a Domain Controller.

If it is not, a native NTDS.dit database is not expected. Any NTDS-related artifact found on that system would therefore require additional investigation.

This lab investigates the potential presence or access of an `NTDS.dit` database on a Windows endpoint.

`NTDS.dit` is the Active Directory database used by Domain Controllers. Because the investigation system is a standalone Windows 11 workstation rather than a Domain Controller, the investigation begins by validating the endpoint role before interpreting any NTDS-related artifact.

The objective was not to assume credential access because of an NTDS reference. Instead, the investigation followed the evidence across filesystem artifacts, Windows event logs, Sysmon process creation telemetry, Volume Shadow Copy state, and Wazuh.

The investigation ultimately demonstrated an important DFIR principle:

> **An artifact name or reference is not, by itself, proof of malicious activity or credential access.**

---

## Investigation Objectives

By the end of this lab, the investigation should allow you to:

- Create and organize a dedicated forensic investigation workspace.
- Establish a baseline for the Windows endpoint before interpreting NTDS-related artifacts.
- Determine whether the system is operating as a Domain Controller or standalone workstation.
- Explain why `NTDS.dit` is significant during a Windows DFIR investigation.
- Verify whether the expected `C:\Windows\NTDS\ntds.dit` location exists on the endpoint.
- Perform a targeted search for `ntds.dit` across the filesystem.
- Identify NTDS-related filenames and distinguish relevant artifacts from normal Windows components and unrelated backup files.
- Examine the metadata of any potentially relevant artifact without accessing credential contents.
- Investigate Sysmon process creation telemetry for activity that could provide context around NTDS-related files.
- Examine Volume Shadow Copy state and identify whether existing snapshots provide additional investigative context.
- Review Windows Application and System event logs for supporting or unrelated activity.
- Review Wazuh telemetry and determine whether available events actually support the investigation.
- Correlate filesystem, process, VSS, Windows event, and SIEM evidence using timestamps.
- Separate confirmed evidence from potential leads, assumptions, and unknowns.
- Document telemetry limitations and explain what the available evidence cannot establish.
- Reach an evidence-based assessment without assuming that an NTDS-related filename or event automatically represents credential access.

---

## Investigation Scenario

A Windows endpoint is being reviewed after a potential indication of **NTDS-related activity**. The investigation team needs to determine whether the endpoint contains an `ntds.dit` database, evidence of a copied or staged NTDS database, or supporting telemetry that could indicate interaction with such an artifact.

The endpoint is not assumed to be a Domain Controller at the beginning of the investigation. This distinction is important because the expected presence and location of `ntds.dit` depend on the system's role. The investigation must therefore begin by establishing the host's configuration before interpreting NTDS-related filenames or events.

The analyst is asked to investigate the endpoint using available host and security telemetry, including:

- Windows filesystem artifacts and file metadata
- Sysmon process creation events
- Windows Application and System event logs
- Volume Shadow Copy information
- Wazuh telemetry
- Timestamps from the investigation workspace and collected evidence

The investigation should answer several core questions:

1. Is the endpoint actually a Domain Controller?
2. Does a legitimate `ntds.dit` database exist anywhere on the system?
3. Are there NTDS-related files or directories that require further examination?
4. Is there supporting telemetry showing access, copying, staging, or other interaction with an NTDS database?
5. Do any VSS or Windows event artifacts provide additional investigative context?
6. Which findings are confirmed, which are only potential leads, and which have no evidentiary connection to NTDS activity?

A key part of the scenario is avoiding premature conclusions. An NTDS-related filename, a VSS event, or a process event by itself does not establish credential access or database theft. Each artifact must be evaluated in context and correlated with other available evidence.

The investigation should remain focused on **artifact discovery, access context, and forensic interpretation**. No password hashes or credential material should be extracted, dumped, cracked, or exposed during the lab.

The final assessment should state what the collected evidence supports, what remains unconfirmed, and what telemetry or artifacts would be required for a stronger conclusion.

---

## Environment

- **Operating System:** Microsoft Windows 11 Pro
- **Version:** 10.0.26200
- **Hostname:** `DESKTOP-9MMM37V`
- **Domain:** `WORKGROUP`
- **Domain Role:** `0`
- **Sysmon:** Installed
- **Wazuh Agent:** Installed
- **Investigation Workspace:** `Lab78-NTDS-Investigation`

---

## Investigation Workflow

The investigation followed this sequence:

1. Create a dedicated investigation workspace
2. Record investigation start time and host information
3. Establish the endpoint baseline
4. Determine whether the endpoint is a Domain Controller
5. Check the standard `NTDS.dit` location
6. Search the filesystem for `ntds.dit`
7. Review related NTDS, `.dit`, and `.bak` artifacts
8. Inspect Volume Shadow Copy state
9. Review Windows Application and System logs
10. Review Sysmon process creation events
11. Review Wazuh telemetry
12. Correlate available evidence
13. Determine what was confirmed, what remained unknown, and what was not supported by evidence

---

## Key Findings

### Endpoint Role

The endpoint reported:

~~~text
Name       : DESKTOP-9MMM37V
Domain     : WORKGROUP
DomainRole : 0
~~~

This establishes that the system is a standalone workstation rather than a Domain Controller.

Therefore, a native:

~~~text
C:\Windows\NTDS\ntds.dit
~~~

database was not expected.

### Standard NTDS Location

The check for:

~~~text
C:\Windows\NTDS\ntds.dit
~~~

returned:

~~~text
False
~~~

The standard `C:\Windows\NTDS` directory also produced no relevant directory listing.

### Filesystem Search

A drive-wide search did not identify an `ntds.dit` file.

The broader search surfaced:

- `C:\NTDSLab`
- Multiple unrelated `.bak` files
- Windows `ntdsapi.dll` files
- Windows component files containing NTDS-related names

These results were treated according to context rather than assuming that every filename containing `ntds` represented an Active Directory database.

In particular:

~~~text
C:\NTDSLab
~~~

was observed, but the available evidence does not establish that it contains an NTDS database or represents credential-access activity.

### Volume Shadow Copy

No existing Volume Shadow Copy snapshots were reported:

~~~text
No items found that satisfy the query.
~~~

This means no active shadow-copy artifact was identified through the checks performed during the investigation.

### Windows Event Logs

The Application log returned one VSS-related event:

~~~text
Event ID: 8224
Provider: VSS
~~~

The message indicated that the VSS service was shutting down because of an idle timeout.

This is not, by itself, evidence of NTDS access.

The System log search did not produce a confirmed NTDS or backup operation. The returned events were related to system time changes from `Microsoft-Windows-Kernel-General`.

### Sysmon

Sysmon Event ID 1 process creation telemetry was available, demonstrating that process creation logging was functioning.

However, the captured output was only the high-level `Process Create` representation and did not establish an NTDS-related process from the evidence provided.

Therefore, no NTDS access operation was confirmed from the displayed Sysmon results.

### Wazuh

The Wazuh event examined during the investigation was:

~~~text
Event ID: 16384
Provider: Software Protection Platform Service
Channel: Application
~~~

The event concerned scheduling of the Software Protection service.

It was not an NTDS-related alert.

This was therefore treated as unrelated telemetry rather than evidence supporting NTDS access.

---

## Investigation Assessment

The investigation did **not** identify a confirmed `ntds.dit` file on the workstation.

It also did not establish a confirmed NTDS access operation through the available Sysmon, Windows event, VSS, or Wazuh evidence.

The investigation therefore remains:

**No confirmed NTDS exposure or access identified from the collected evidence.**

This conclusion is deliberately narrower than saying that NTDS activity was impossible. The available telemetry and filesystem searches establish what was observed during the investigation, not everything that may have occurred historically.

---

