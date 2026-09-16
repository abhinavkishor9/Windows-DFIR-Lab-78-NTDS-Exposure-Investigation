# Lab 78 — NTDS.dit Exposure and Access Investigation

## Overview

This lab investigates the potential presence or access of an `NTDS.dit` database on a Windows endpoint.

`NTDS.dit` is the Active Directory database used by Domain Controllers. Because the investigation system is a standalone Windows 11 workstation rather than a Domain Controller, the investigation begins by validating the endpoint role before interpreting any NTDS-related artifact.

The objective was not to assume credential access because of an NTDS reference. Instead, the investigation followed the evidence across filesystem artifacts, Windows event logs, Sysmon process creation telemetry, Volume Shadow Copy state, and Wazuh.

The investigation ultimately demonstrated an important DFIR principle:

> **An artifact name or reference is not, by itself, proof of malicious activity or credential access.**

---

## Investigation Question

> Is there evidence that an `NTDS.dit` database exists on the endpoint, was copied or staged, or was accessed through processes, backup activity, or other supporting telemetry?

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

## DFIR Lessons

### 1. Validate the Host Before Interpreting the Artifact

Before investigating an NTDS database, determine whether the endpoint is actually a Domain Controller.

A workstation and a Domain Controller have very different expectations for NTDS artifacts.

### 2. Artifact Names Are Not Conclusions

A directory named:

~~~text
NTDSLab
~~~

does not automatically represent:

~~~text
NTDS.dit
~~~

Likewise, Windows files such as:

~~~text
ntdsapi.dll
~~~

are not equivalent to an Active Directory database.

### 3. Correlation Matters

A stronger investigation would require multiple supporting signals such as:

~~~text
NTDS artifact
      +
relevant file timestamp
      +
process activity
      +
backup/VSS activity
      +
supporting security telemetry
~~~

The investigation did not establish that complete chain.

### 4. Negative Findings Are Still Findings

Not finding `ntds.dit` on a standalone workstation is meaningful when the search was performed and documented.

It is important to record:

- What was searched
- Where it was searched
- What was found
- What was not found
- What telemetry was available
- What telemetry was missing

### 5. Telemetry Limitations Matter

The absence of an event does not automatically prove that an activity never occurred.

Filesystem searches, Sysmon, Windows logs, VSS state, and Wazuh each provide only part of the investigative picture.

---

## Investigation Philosophy

This lab follows the principle:

> **Follow the evidence, not the assumption.**

The investigation did not attempt to create an NTDS database or artificially generate credential-access activity simply to produce a positive result.

The focus remained on identifying what the endpoint actually contained and what the available telemetry could support.

---

## Evidence Summary

| Evidence Source | Result | Interpretation |
|---|---|---|
| Endpoint role | `DomainRole 0` | Standalone workstation |
| Standard NTDS path | Not present | Expected on this endpoint |
| Targeted `ntds.dit` search | No result | No `ntds.dit` identified |
| Broad filesystem search | `C:\NTDSLab` and unrelated files | Requires context; not proof of NTDS |
| VSS | No shadows found | No existing shadow copy identified |
| Application log | VSS Event 8224 | Idle-timeout event; not proof of NTDS access |
| System log | Time-change events | Not NTDS activity |
| Sysmon | Process creation telemetry available | No NTDS access established from displayed output |
| Wazuh | Software Protection event | Unrelated to NTDS |

---

## Final Takeaway

This investigation demonstrates how a SOC or DFIR analyst should approach a potentially sensitive artifact without jumping directly to a compromise conclusion.

The correct sequence was:

~~~text
Validate host
    ↓
Determine whether NTDS is expected
    ↓
Search for the actual artifact
    ↓
Examine supporting filesystem evidence
    ↓
Review process activity
    ↓
Review VSS/backup activity
    ↓
Review Windows telemetry
    ↓
Review SIEM telemetry
    ↓
Correlate evidence
    ↓
State only what the evidence supports
~~~

The investigation did not confirm `NTDS.dit` exposure or access on the endpoint.
