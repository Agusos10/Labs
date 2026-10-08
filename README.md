# Labs

Documented IT and security lab write-ups: guided certification labs and custom home lab builds. Each lab follows the same structure: objective, execution methodology, enterprise remediation, and certification mapping.

## Guided Labs

| Lab | Focus | Tools |
|---|---|---|
| [Windows Server 2019 Gap Analysis with Policy Analyzer](./Guided_Labs/Windows_Server_2019_Gap_Analysis_Policy_Analyzer) | Compared a Windows Server 2019 host to the Microsoft v1809 security baseline and documented remediation for non-compliant settings | Policy Analyzer, Microsoft Security Compliance Toolkit, PowerShell |

## Repository Structure

```
Labs/
├── Guided_Labs/          # Certification and course labs (WGU, CompTIA, Hack The Box)
│   └── <Lab_Name>/
│       ├── README.md
│       ├── CHANGELOG.md
│       └── assets/       # Diagrams and sanitized screenshots (when applicable)
```

## Documentation Standard

- Technical, active-voice write-ups with no filler.
- Troubleshooting documented as *Symptom -> Investigation -> Root Cause -> Fix*.
- Offensive security labs include enterprise remediation steps.
- Screenshots and configs are sanitized: no credentials, keys, or real internal addresses.

## Skills & Technologies

Security Baselines, Gap Analysis, Configuration Management, Group Policy, Windows Server, PowerShell, Compliance Auditing, Hardening
