# DISA STIG Implementation with PowerShell

A hands-on series implementing DISA Security Technical Implementation Guide (STIG) findings using PowerShell as the remediation engine. Each entry takes a real Tenable scan failure on a stock Azure Windows 11 VM and walks through hardening it end to end: identify, remediate, rescan, document.

**Author:** Mohamed Yagoub
**Status:** In progress 

---

## Why PowerShell

Most STIG remediation in Microsoft environments is either clicked through manually or wrapped in a GPO. Both work, but neither scales well and neither leaves an artifact you can version-control or audit. PowerShell does both, and it maps cleanly to the rule-by-rule structure of a STIG checklist. Every remediation in this series is:

* **Idempotent** (safe to re-run)
* **Auditable** (a clear function per Rule ID)
* **Self-documenting** (each function references the V-ID, plugin ID, CCI, and severity)

---

## What is a STIG?

The Defense Information Systems Agency (DISA) publishes STIGs as configuration standards for DoD systems. Each STIG is a catalog of security requirements (Group IDs, Rule IDs, CCIs, severities) covering a specific product or platform. Tenable maps these requirements to compliance plugins that can be run against a target via the DISA STIG scan template.

Reference: [DISA STIG Library](https://public.cyber.mil/stigs/)

---

## STIG Index

| #  | STIG                     | Repo Link                                                                              | Status    |
|----|--------------------------|----------------------------------------------------------------------------------------|-----------|
| 01 | STIG 01: WN11-AU-000500  | [link](https://github.com/goubx/STIG-Implementation-WN11-AU-000500)                    | Completed |
| 02 | STIG 02: WN11-CC-000150  | [link](https://github.com/goubx/STIG-Implementation-WN11-CC-000150)                    | Completed |
| 03 | STIG 03: WN11-AC-000010  | [link](https://github.com/goubx/STIG-Implementation-WN11-AU-000585)                    | Completed |
| 04 | STIG 04: WN11-CC-000100  | [link](https://github.com/goubx/STIG-Implementation-WN11-CC-000100)                    | Completed |
| 05 | STIG 05: WN11-CC-000280  | [link](https://github.com/goubx/STIG-Implementation-WN11-CC-000280)                    | Completed |
| 06 | STIG 06: WN11-CC-000005  | [link](https://github.com/goubx/STIG-Implementation-WN11-CC-000005)                    | Completed |
| 07 | STIG 07: WN11-CC-000326  | [link](https://github.com/goubx/STIG-Implementation-WN11-CC-000326)                    | Completed |
| 08 | STIG 08: WN11-00-000210  | [link](https://github.com/goubx/STIG-Implementation-WN11-00-000210)                    | Completed |

Status values: Planned, In Progress, Completed

---

## Methodology

Every implementation in this series follows the same workflow:

1. **Stand up the lab.** Deploy a stock Azure Windows 11 Pro VM. No pre-hardening, no custom image. The point is to scan a real-world default build.
2. **Baseline scan.** Run a Tenable Nessus scan using Plugin ID 21156 (Windows Compliance Checks) against the DISA Microsoft Windows 11 STIG v2r7 audit file. Stock Azure images return a long list of failures out of the box.
3. **Pick a finding.** In the Tenable scan results, open the **Audit** tab and select a failed check to remediate. Capture the V-ID, plugin ID, CCI, severity, and check name.
4. **Remediate with PowerShell.** Write a function that addresses the finding. The function references the V-ID, CCI, and Tenable plugin ID in its header. One function per V-ID keeps the artifact mappable to the scan.
5. **Rescan.** Re-run the same Tenable scan. Confirm the finding moves from failed to passed in the Audit tab.
6. **Document.** Update the per-STIG README with before and after screenshots, the remediation logic, and any operational impact.

### Alternative: Tenable Plugin Database

For non-STIG vulnerabilities, the same workflow applies using the [Tenable plugin database](https://www.tenable.com/plugins/search):

1. Search the plugin database for a platform-specific finding.
2. Reproduce the vulnerable condition on a lab VM if it isn't already present.
3. Remediate with PowerShell.
4. Rescan to confirm.

This series focuses on STIG findings since Azure default images surface dozens of them out of the box, but the structure of each repo supports either path.

---

## Tooling Stack

| Tool                          | Purpose                                                             |
|-------------------------------|---------------------------------------------------------------------|
| Tenable Nessus                | Primary scanner, Plugin ID 21156 against DISA Windows 11 STIG v2r7  |
| Tenable Plugin Database       | Reference for individual plugin IDs and fixes                       |
| Windows PowerShell 5.1        | Primary remediation engine                                          |
| secedit                       | Applies .inf security templates for policy-based findings           |
| Azure                         | Lab VM hosting                                                      |

---

## Lab Environment

All implementations run against a fresh stock Azure Windows 11 Pro VM. No pre-hardening, no custom image, no domain join.

* Hostname: `WIN-TARGET-01`
* Firewall: disabled for scan visibility
* Domain: none (standalone)
* No production credentials or PII present in any artifact

---

## Per-STIG Repo Layout

Each STIG implementation lives in its own repository, linked from the index above. Individual repos follow this structure:

```
STIG-XX-V-XXXXXX/
├── README.md         Full walkthrough: finding, remediation, verification
└── screenshots/      Numbered evidence, slots 01 through 12
```

Each per-STIG README follows a fixed 12-section template covering the target platform, tools used, lab setup, scan configuration, initial scan, finding details, remediation steps, result, and references. Screenshots are numbered and reused across the series where the setup is constant (firewall state, Tenable compliance config, plugin selection).

---

## Script Conventions

Every remediation function in this series follows the same shape so the artifacts stay consistent across the series:

```powershell
function Set-StigRule-V123456 {
    <#
    .SYNOPSIS
        V-123456: Short rule title from the STIG.
    .DESCRIPTION
        Severity:        CAT II
        STIG ID:         WN11-AU-000005
        CCI:             CCI-000000
        Tenable Plugin:  21156
        Reference:       DISA Microsoft Windows 11 STIG v2r7
    #>
    [CmdletBinding(SupportsShouldProcess)]
    param()

    # Implementation
}
```

Every script is run from an elevated PowerShell session on the target. Registry-based findings use `Set-ItemProperty` with `Test-Path` and `New-Item` guards for keys that do not exist on a stock image. Policy-based findings use `secedit` with an exported `.inf` template. Functions are idempotent and emit a clear pass or fail line per V-ID.

---

## Disclaimer

These implementations are for educational and lab use. STIG content is publicly released by DISA, but specific deployments should be validated against your organization's authorization boundary and accreditation requirements. Nothing here is official DoD guidance.

---

## Connect

* LinkedIn: [Mohamed Yagoub](https://www.linkedin.com/in/mohamedyagoub)
* Blog: [BuggedOut Academy](https://buggedout.blog/)
