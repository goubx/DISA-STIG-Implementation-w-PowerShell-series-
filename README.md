# DISA STIG Implementation with PowerShell

A hands-on series implementing DISA Security Technical Implementation Guide (STIG) findings using PowerShell as the remediation engine. Each entry takes a real Tenable scan failure on a stock Azure VM and walks through hardening it end-to-end: identify, remediate, rescan, document.

**Author:** Mohamed Yagoub
**Status:** In progress (0 of 10 published)

---

## Why PowerShell

Most STIG remediation in Microsoft environments is either clicked through manually or wrapped in a GPO. Both work, but neither scales well and neither leaves an artifact you can version-control or audit. PowerShell does both, and it maps cleanly to the rule-by-rule structure of a STIG checklist. Every remediation in this series is:

* **Idempotent** (safe to re-run)
* **Auditable** (a clear function per Rule ID)
* **Reversible** (paired rollback where it makes sense)
* **Self-documenting** (each function references the V-ID, plugin ID, and severity)

---

## What is a STIG?

The Defense Information Systems Agency (DISA) publishes STIGs as configuration standards for DoD systems. Each STIG is a catalog of security requirements (Group IDs, Rule IDs, CCIs, severities) covering a specific product or platform. Tenable maps these requirements to compliance plugins that can be run against a target via the DISA STIG scan template.

Reference: [DISA STIG Library](https://public.cyber.mil/stigs/)

---

## STIG Index

| #  | STIG                                           | Repo Link | Status   |
|----|-----------------------------------------------|------------|-----------|
| 01 | STIG 01: WN11-AU-000500                       | [link](https://github.com/goubx/STIG-Implementation-WN11-AU-000500)  | Completed  |
| 02 | STIG 02: WN11-CC-000150                       | [link](https://github.com/goubx/STIG-Implementation-WN11-CC-000150)  | Completed  |
| 03 | STIG 03: WN11-AC-000010                       | [link](https://github.com/goubx/STIG-Implementation-WN11-AU-000585)  | Completed  |
| 04 | STIG 04: WN11-CC-000100                       | [link](https://github.com/goubx/STIG-Implementation-WN11-CC-000100)  | Completed  |
| 05 | STIG 05:                                      | [link]()  | Planned  |
| 06 | STIG 06:                                      | [link]()  | Planned  |
| 07 | STIG 07:                                      | [link]()  | Planned  |
| 08 | STIG 08:                                      | [link]()  | Planned  |
| 09 | SITG 09:                                      | [link]()  | Planned  |
| 10 | STIG 10:                                      | [link]()  | Planned  |

Status values: Planned, In Progress, Published, Updated

---

## Methodology

Every implementation in this series follows the same workflow:

1. **Stand up the lab.** Deploy a stock Azure VM matching the target platform (e.g. Windows 11, Windows Server 2022). No pre-hardening, no custom image. The point is to scan a real-world default build.
2. **Baseline scan.** Run a Tenable Nessus scan using the **DISA STIG** scan template against the VM. Stock Azure images return a long list of failures out of the box.
3. **Pick a finding.** In the Tenable scan results, open the **Audit** tab and select a failed check to remediate. Capture the V-ID, plugin ID, severity, and check name.
4. **Remediate with PowerShell.** Write a function that addresses the finding. The function references the V-ID and Tenable plugin ID in its header. One function per V-ID keeps the artifact mappable to the scan.
5. **Rescan.** Re-run the same Tenable scan. Confirm the finding moves from failed to passed in the Audit tab.
6. **Document.** Update the per-STIG README with before/after screenshots, the remediation logic, and any operational impact. Track checklist state in a `.ckl` exported from STIG Viewer if applicable.

### Alternative: Tenable Plugin Database

For non-STIG vulnerabilities, the same workflow applies using the [Tenable plugin database](https://www.tenable.com/plugins/search):

1. Search the plugin database for a platform-specific finding.
2. Reproduce the vulnerable condition on a lab VM if it isn't already present.
3. Remediate with PowerShell.
4. Rescan to confirm.

This series focuses on STIG findings since Azure default images surface dozens of them out of the box, but the structure of each repo supports either path.

---

## Tooling Stack

| Tool                          | Purpose                                          |
|-------------------------------|--------------------------------------------------|
| Tenable Nessus                | Primary scanner, DISA STIG scan template         |
| Tenable Plugin Database       | Reference for individual plugin IDs and fixes    |
| Windows PowerShell 5.1 / 7.x  | Primary remediation engine                       |
| PowerShell DSC                | Declarative configuration where it fits          |
| `PowerSTIG` module            | Reference and cross-validation                   |
| DISA STIG Viewer              | Checklist tracking, `.ckl` output                |
| Azure                         | Lab VM hosting                                   |

---

## Lab Environment

All implementations run against fresh Azure VMs. No pre-hardening, no custom images, no shared state between STIGs. Hostnames, usernames, and network details are genericized in every artifact:

* Hostname pattern: `WIN-TARGET-01`, `WIN-DC-01`, `WIN-SQL-01`
* Domain (where used): `lab.local`
* No production credentials, IPs, or PII present in any artifact

---

## Per-STIG Repo Layout

Each STIG implementation lives in its own repository, linked from the index above. Individual repos follow this structure:

```
STIG-XX-Platform-Name/
├── README.md                       Walkthrough, findings, screenshots
├── scans/
│   ├── pre-remediation.nessus      Tenable export, baseline
│   └── post-remediation.nessus     Tenable export, post-fix
├── checklists/                     Optional, .ckl from STIG Viewer
├── remediation/
│   ├── Invoke-StigRemediation.ps1  Main entry point
│   ├── Functions/                  One file per V-ID
│   └── Rollback/                   Reversal scripts where applicable
├── evidence/                       Screenshots of Audit tab pre/post
└── notes/                          Risk acceptances, NA justifications
```

---

## Script Conventions

Every remediation function in this series follows the same shape so the artifacts stay consistent across all 10 STIGs:

```powershell
function Set-StigRule-V123456 {
    <#
    .SYNOPSIS
        V-123456: Short rule title from the STIG.
    .DESCRIPTION
        Severity:        CAT II
        STIG ID:         WN11-AU-000005
        Tenable Plugin:  123456
        Reference:       DISA Windows 11 STIG vYRZ
    #>
    [CmdletBinding(SupportsShouldProcess)]
    param()

    # Implementation
}
```

Every script is run from an elevated PowerShell session on the target. Functions are idempotent and emit a clear pass/fail line per V-ID.

---

## Disclaimer

These implementations are for educational and lab use. STIG content is publicly released by DISA but specific deployments should be validated against your organization's authorization boundary and accreditation requirements. Nothing here is official DoD guidance.

---

## Connect

* LinkedIn: [Mohamed Yagoub](https://www.linkedin.com/in/your-handle)
* Blog: [BuggedOut Academy](https://your-site-here.com)
