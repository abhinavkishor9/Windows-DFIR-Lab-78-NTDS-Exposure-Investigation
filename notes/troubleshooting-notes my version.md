# Troubleshooting Notes 

## 1. NTDS.dit Was Not Found

### Observation

The standard NTDS path check returned:

~~~text
False
~~~

The targeted filesystem search for:

~~~text
ntds.dit
~~~

also returned no results.

### Explanation

This was expected because the investigation system reported:

~~~text
Domain: WORKGROUP
DomainRole: 0
~~~

The machine is a standalone Windows workstation and not a Domain Controller.

### Lesson

Do not create, download, copy, or artificially place an `ntds.dit` file on the workstation simply to generate a positive finding.

The correct forensic approach is to validate whether the artifact should exist before interpreting its absence.

---

## 2. Broad Filesystem Search Returned NTDS-Related Files

### Observation

The broader search returned objects such as:

~~~text
C:\NTDSLab
C:\Windows\System32\ntdsapi.dll
C:\Windows\SysWOW64\ntdsapi.dll
~~~

It also returned numerous unrelated `.bak` files and Windows component files containing NTDS-related names.

### Explanation

A filename containing `ntds` does not automatically mean that it is an Active Directory database.

For example:

~~~text
ntdsapi.dll
~~~

is not:

~~~text
ntds.dit
~~~

Similarly:

~~~text
C:\NTDSLab
~~~

only establishes that an object with that name was found.

### Lesson

Treat broad searches as lead generation.

Every result requires contextual validation.

---

## 3. No VSS Snapshots Were Found

### Observation

`vssadmin list shadows` returned:

~~~text
No items found that satisfy the query.
~~~

### Explanation

No existing shadow copies were visible through the command used during the investigation.

This does not establish that VSS has never operated on the system.

### Lesson

Distinguish:

~~~text
No current shadow copies found
~~~

from:

~~~text
VSS was never used
~~~

The first statement is supported by the evidence. The second is not.

---

## 4. Application Log Returned a VSS Event

### Observation

The Application log returned:

~~~text
Event ID: 8224
Provider: VSS
~~~

with a message indicating that the VSS service was shutting down because of an idle timeout.

### Interpretation

This confirms VSS service activity at that time.

It does not establish:

- NTDS access
- NTDS copying
- Credential access
- Malicious activity

### Lesson

A VSS event should not automatically be converted into an NTDS finding.

Context and correlation are required.

---

## 5. System Log Search Returned Time-Change Events

### Observation

The System log search returned:

~~~text
Provider: Microsoft-Windows-Kernel-General
Event ID: 1
~~~

The events described system time changes.

### Interpretation

These events were not evidence of NTDS or backup activity.

They are still potentially relevant to forensic timeline interpretation because system time changes can affect timestamp correlation.

### Lesson

Not every event returned by a keyword search is relevant to the original investigative question.

---

## 6. Sysmon Process Events Were Available but Incomplete in the Captured Output

### Observation

Sysmon Event ID 1 returned multiple process creation events.

The displayed output was:

~~~text
Process Create:...
~~~

rather than the full process details.

### Limitation

The captured output did not provide enough process information to establish:

- A process interacting with `ntds.dit`
- A specific command line targeting NTDS
- A backup utility accessing NTDS
- A confirmed staging operation

### Lesson

The presence of process telemetry is not equivalent to having sufficient evidence for attribution.

For future investigations, preserve the full relevant Event ID 1 fields, particularly:

- Image
- CommandLine
- ParentImage
- ParentCommandLine
- User
- ProcessId
- ParentProcessId
- CurrentDirectory
- Hashes

---

## 7. Wazuh Event Was Not Relevant to NTDS

### Observation

The examined Wazuh event contained:

~~~text
Event ID: 16384
Provider: Software Protection Platform Service
Channel: Application
~~~

### Interpretation

This was a Windows Software Protection event rather than NTDS-related activity.

### Lesson

Do not include an unrelated SIEM alert merely because it came from the investigated host.

A useful SOC investigation asks:

> Does this event actually support the investigative hypothesis?

In this case, it did not.

---

## 8. Search Output Should Be Preserved Carefully

### Observation

The investigation created separate directories for:

~~~text
Evidence
Logs
Notes
Timeline
~~~

### Lesson

Separating evidence, logs, notes, and timeline information makes later correlation easier.

For future labs, preserve:

- Original command output
- Search results
- Relevant event details
- Analyst interpretation
- Timeline entries
- Investigation limitations

This prevents findings from becoming mixed with assumptions.

---

## 9. Broad Searches Can Produce Noise

### Observation

The filesystem search returned many unrelated `.bak` and Windows component files.

### Explanation

Searching the entire `C:\` drive using broad filename conditions creates significant noise.

### Lesson

A practical investigation should progress from:

~~~text
Known artifact
    ↓
Expected location
    ↓
Targeted search
    ↓
Broader search
    ↓
Contextual validation
~~~

Rather than immediately treating every matching filename as suspicious.

---

## 10. Evidence Versus Assumption

A recurring troubleshooting lesson from this investigation was the distinction between an observation and an interpretation.

### Observation

~~~text
C:\NTDSLab
~~~

### Unsupported Conclusion

~~~text
An NTDS database was staged.
~~~

### Evidence-Based Position

~~~text
An object named C:\NTDSLab was identified and requires contextual investigation.
~~~

The third statement accurately reflects the available evidence.

---

