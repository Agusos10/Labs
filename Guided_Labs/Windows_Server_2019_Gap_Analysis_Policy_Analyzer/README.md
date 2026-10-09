# Windows Server 2019 Gap Analysis with Microsoft Policy Analyzer

![Windows Server](https://img.shields.io/badge/Windows_Server_2019-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white)
**Microsoft Security Compliance Toolkit** | **Policy Analyzer** | **Security Baselines**

## Scenario & Objective
Compared the effective configuration of a Windows Server 2019 host (PC10) against the Microsoft Security Baseline for Windows 10 v1809 / Windows Server 2019 to identify configuration drift.

## Execution Methodology

### Phase 1: Identify OS Version and Build
- Ran `winver` to record **Version 1809, OS Build 17763.4377**.
- Matched version and build to the corresponding baseline package. Baseline templates are version-specific, so a mismatch invalidates the comparison.

### Phase 2: Stage Toolkit Artifacts
- Opened an elevated Windows PowerShell session (not ISE, not x86).
- Copied the Microsoft Security Compliance Toolkit packages from removable media:
  ```powershell
  copy D:* C:\LABFILES
  cd C:\LABFILES
  Expand-Archive -Path PolicyAnalyzer.zip
  Expand-Archive -Path "Windows 10 Version 1809 and Windows Server 2019 Security Baseline.zip"
  C:\LABFILES\PolicyAnalyzer\PolicyAnalyzer_40\PolicyAnalyzer.exe
  ```

### Phase 3: Review the Baseline (View / Compare)
- Pointed Policy Analyzer at the baseline `Documentation` folder and loaded rule set `MSFT-Win10-v1809-RS5-WS2019-FINAL`.
- Ran **View / Compare** to inspect the template's intended settings.

### Phase 4: Gap Analysis (Compare to Effective State)
- Ran **Compare to Effective State** to diff the baseline against the live system. Yellow-highlighted rows mark settings that differ.

| Policy Setting | Baseline Value | Effective Value | Status |
|---|---|---|---|
| `LockoutBadCount` | 10 | 0 | Non-compliant |
| `MinimumPasswordLength` | 14 | 7 | Non-compliant |

**Result:** PC10 does not comply with the baseline. Account lockout is disabled (0 = never lock) and the minimum password length is half the baseline value.

## Enterprise Remediation
1. **Enforce account lockout:** set `LockoutBadCount` to 10 via GPO (*Computer Configuration > Windows Settings > Security Settings > Account Policies > Account Lockout Policy*) to throttle online brute-force and password-spray attempts.
2. **Raise password length:** set `MinimumPasswordLength` to 14 under *Account Policies > Password Policy*.
3. **Deploy at scale:** import the baseline GPO backups from the toolkit, link them to the target OU, and validate with `gpresult /r`.
4. **Re-run Policy Analyzer** after deployment to confirm zero unexpected deltas, and schedule recurring comparisons to detect drift.
5. **Tailor the baseline:** the Microsoft template reflects general best practice, not organizational risk. Document and approve each deviation before enforcement.

## Lessons Learned
- Match the baseline to the exact OS version and build before comparing; the results are only valid for the matching release.
- Third-party baselines require scoping and tailoring to the organization's risk profile and business goals.
- Policy Analyzer's View / Compare shows intent; Compare to Effective State shows reality. Both are required to quantify the gap.

## Skills & Technologies
Gap Analysis, Security Baselines, Configuration Management, Configuration Drift Detection, Microsoft Security Compliance Toolkit, Policy Analyzer, Group Policy, Windows Server 2019, Account Lockout Policy, Password Policy, PowerShell, Hardening, Compliance Auditing
