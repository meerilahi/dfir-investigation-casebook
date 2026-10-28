# case-001-thm-boogeyman

DFIR investigation of the TryHackMe **Boogeyman 2** lab: a phishing email with a malicious Word macro, followed by second-stage execution on a Windows workstation.

| | |
|---|---|
| **Case ID** | case-001-thm-boogeyman |
| **Source** | TryHackMe, Boogeyman 2 room (lab scenario) |
| **Incident type** | Phishing, malicious macro document, second-stage executable |
| **Affected host** | WKSTN-2961 (user: maxine.beck) |
| **Verdict** | Confirmed malicious document-led execution; scheduled-task persistence plausible but unverified |
| **Confidence** | High (execution chain), medium (persistence) |

## Summary

An HR specialist opened a job-application email carrying `Resume_WesleyTaylor.doc`. Its VBA `AutoOpen` macro downloads a remote file, saves it as `C:\ProgramData\update.js`, and runs it with `wscript.exe`. Memory analysis shows the chain `OUTLOOK.EXE -> WINWORD.EXE -> wscript.exe -> updater.exe`, plus memory strings describing a daily `Updater` scheduled task that runs hidden PowerShell from a registry value.

Not established: what `updater.exe` did, whether persistence was actually registered, and any data theft or lateral movement.

## Contents

| File | Description |
|---|---|
| `report.pdf` / `report.docx` | Full investigation report (export of the case report) |
| `README.md` | This file |

Suggested layout for supporting material (evidence itself should not be committed):

```
case-001-thm-boogeyman/
├── README.md
├── report.pdf
├── iocs.csv          # indicators from report section 5
└── notes/            # command output, Volatility plugin logs
```

## Evidence

Hashes are SHA-256. Original files are not stored in this repo.

| File | SHA-256 |
|---|---|
| Application for IT Analyst Role.eml | `88d7090d0a9f7115ce9f76a2c82f6d4843cdf655fba09e0f0c50e4eddd554fce` |
| Resume_WesleyTaylor.doc | `4db25ee3c46be38aa219fe2192711af65d55d5d7e25a889bb9990beb19f9b8b0` |
| WKSTN-2961.raw | `c0daf36eac4d1d2ff1a328bec21bcf73a2b3c958735d127319a1552d75dc9266` |

## Key indicators (defanged)

- Sender: `westaylor23[@]outlook[.]com`
- URL: `hxxps://files[.]boogeymanisback[.]lol/aa2a9c53cbb80416d3b47d85538d9971/update[.]png`
- Paths: `C:\ProgramData\update.js`, `C:\Windows\Tasks\updater.exe`
- Task name (unverified): `Updater`
- Registry (unverified): `HKCU\Software\Microsoft\Windows\CurrentVersion`, value `debug`

## Tools

Volatility 3 (2.28.2: pslist, pstree, cmdline, netstat, malfind, dlllist, filescan), olevba 0.60.2, Linux `strings`.

## Handling note

The evidence contains live malware. Analyze copies only and never execute payloads on the analysis host.
