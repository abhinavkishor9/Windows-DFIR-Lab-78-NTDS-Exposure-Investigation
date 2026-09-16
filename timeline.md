# Lab 78 — Investigation Timeline

## Timeline Overview

This timeline records the major investigative actions and evidence observed during the NTDS.dit investigation.

The timeline distinguishes between:

- Investigation actions
- Host artifacts
- Security telemetry
- Supporting evidence
- Unrelated telemetry
- Evidence limitations

---

## Investigation Timeline

| Time | Evidence / Action | Source | Interpretation |
|---|---|---|---|
| 16-09-2026 07:24:39 | `C:\NTDSLab` creation timestamp observed | Filesystem search | NTDS-related name observed; significance not established |
| 16-09-2026 07:36 | Investigation workspace created | PowerShell | Evidence collection structure established |
| 16-09-2026 07:36 | `Evidence`, `Logs`, `Notes`, and `Timeline` directories created | PowerShell | Investigation workspace initialized |
| 16-09-2026 | Investigation start timestamp recorded | PowerShell | Establishes investigation start point |
| 16-09-2026 | Hostname recorded | PowerShell | Endpoint identified as `DESKTOP-9MMM37V` |
| 16-09-2026 | Endpoint role identified | CIM / PowerShell | `WORKGROUP`, `DomainRole 0`; standalone workstation |
| 16-09-2026 | Operating system identified | CIM / PowerShell | Windows 11 Pro, build 26200 |
| 16-09-2026 | Standard NTDS path tested | PowerShell | `C:\Windows\NTDS\ntds.dit` not present |
| 16-09-2026 | Targeted `ntds.dit` search completed | Filesystem | No `ntds.dit` identified |
| 16-09-2026 | Broad NTDS/backup search completed | Filesystem | Multiple unrelated results plus `C:\NTDSLab` |
| 16-09-2026 | VSS state inspected | VSS / PowerShell | No shadow copies found |
| 14-09-2026 07:21:44 | VSS Event ID 8224 | Application / VSS | VSS service idle-timeout shutdown; not evidence of NTDS access |
| 16-09-2026 07:05:23 | Kernel-General Event ID 1 | System | System time-change events; relevant to timestamp interpretation |
| 16-09-2026 07:43–07:45 | Sysmon Event ID 1 activity observed | Sysmon | Process creation telemetry available; NTDS access not established from displayed output |
| 16-09-2026 | Wazuh Event ID 16384 reviewed | Wazuh / Application | Software Protection Platform event; unrelated to NTDS |
| 16-09-2026 | Evidence correlation performed | Analyst notes | No confirmed NTDS exposure or access established |

---

## Key Artifact Timeline

### `C:\NTDSLab`

~~~text
CreationTime: 16-09-2026 07:24:39
LastWriteTime: 16-09-2026 07:24:39
~~~

### Interpretation

The filesystem search identified an object named:

~~~text
C:\NTDSLab
~~~

The available evidence does not establish its contents or purpose.

It should therefore remain a lead rather than being classified as an NTDS database or credential-staging location.

---

## VSS Timeline

### 14-09-2026 07:21:44

~~~text
Event ID: 8224
Provider: VSS
~~~

Description:

~~~text
The VSS service is shutting down due to idle timeout.
~~~

### Interpretation

This indicates VSS service activity.

No connection to NTDS access was established.

---

## System Time Events

### 16-09-2026 07:05:23

Two `Microsoft-Windows-Kernel-General` Event ID 1 records were returned by the keyword search.

The messages described system time changes.

### Investigation Relevance

These events are not NTDS evidence.

However, system time changes should be considered when correlating timestamps from multiple forensic sources.

---

## Sysmon Timeline

### 16-09-2026 07:43–07:45

Multiple Sysmon Event ID 1 process creation events were observed.

The available console extraction displayed:

~~~text
Process Create:...
~~~

without the complete process details.

### Assessment

Process creation telemetry was available, but the captured evidence did not establish an NTDS-related process.

---

## Wazuh Timeline

### 16-09-2026

Wazuh telemetry showed:

~~~text
Event ID: 16384
Provider: Software Protection Platform Service
Channel: Application
~~~

### Assessment

This event was not related to NTDS activity and was excluded from the NTDS evidence chain.

---

## Correlation View

The investigation produced the following evidence sequence:

~~~text
Standalone Windows workstation
        ↓
NTDS not normally expected
        ↓
Standard NTDS path absent
        ↓
No ntds.dit identified in targeted search
        ↓
Broad search produced NTDS-related filenames
        ↓
C:\NTDSLab identified as an unexplained filesystem object
        ↓
No VSS snapshots found
        ↓
VSS event identified but not linked to NTDS
        ↓
Sysmon process telemetry available
        ↓
No NTDS process activity established from captured output
        ↓
Wazuh event reviewed but unrelated
        ↓
No confirmed NTDS exposure or access
~~~

---

## Evidence Classification

| Evidence | Classification |
|---|---|
| `DomainRole 0` | Confirmed |
| `WORKGROUP` membership | Confirmed |
| Standard `ntds.dit` absent | Confirmed |
| Targeted `ntds.dit` search returned no result | Confirmed |
| `C:\NTDSLab` exists | Confirmed |
| `C:\NTDSLab` is an NTDS database | Not established |
| VSS shadow copies present | Not identified |
| VSS service activity | Confirmed |
| VSS activity related to NTDS | Not established |
| Sysmon process creation telemetry | Confirmed |
| NTDS-related process execution | Not established |
| Wazuh NTDS alert | Not identified in supplied evidence |
| Credential access | Not established |

---

## Final Timeline Assessment

The timeline does not establish a supported sequence of:

~~~text
NTDS database
    →
access
    →
copy/staging
    →
credential access
~~~

Instead, it establishes that the endpoint was a standalone Windows workstation, the expected NTDS database was absent, no `ntds.dit` was identified by the targeted search, and the supporting telemetry reviewed did not establish NTDS access.

The unexplained `C:\NTDSLab` object remains a potential lead, but the available evidence is insufficient to classify it as an NTDS database or malicious staging location.
