# Lab 78 

## Timeline

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

