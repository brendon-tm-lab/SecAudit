# SecAudit
SecAudit is a beginner-friendly desktop workspace for reviewing a folder for
common source-code security problems, optionally checking files with ClamAV, and
getting plain-language help from local or cloud language models. It builds on the
existing Strix-inspired draft in this repository.

**Overview last updated: 2026-10-29.** This describes the current top-level
SecAudit application; the nested upstream Strix checkout remains separate.

The desktop application is implemented in Python with native PyQt6 widgets. Qt's
widget and networking libraries use native C++ underneath; the application does not
embed a browser. The separate upstream Strix checkout is Python. Dedicated SecAudit Rust or C++
modules have not been added yet; they should be introduced only
for profiled performance or reliability needs behind stable Python interfaces.

The first workflow is deliberately simple: **choose a folder → review what will be
checked → scan → read the findings → decide what to do**. A scan is read-only. The app
does not run files from the selected folder. A separate confirmation can quarantine a
ClamAV detection; code-pattern findings never trigger file edits.

## What it can do

- Check supported source and configuration files, including scripts inside bounded ZIP archives, for common security patterns such as encoded PowerShell, download-to-shell behavior, and persistence commands.
- Run Dev → Batch Review on a selected file set with argument-aware command context, correlated duplicate indicators, redacted line-numbered evidence, per-file rule coverage, and explicit Semgrep/ClamAV `NOT RUN` status. Batch Review is bounded static analysis and does not execute files.
- On Windows, check Authenticode signatures for selected PE applications. ZIP-contained applications get an embedded-signature presence check; Windows trust is not verified from inside the archive. ClamAV checks viruses, including supported archives, when its executable and current signatures are available.
- Run ClamAV when its executable and recent signature database are available. The
  Settings screen can refresh app-managed signatures; otherwise the report says the
  malware check was skipped.
- Verify Semgrep with both a version check and a bundled-rule fixture scan, and verify
  ClamAV with a local signature probe before the tool registry marks the engine
  verified. The Run Monitor and Settings show the persisted probe result; an
  unavailable or partial check is never rendered as clean.
- Optionally keep ClamAV's loopback-only daemon warm for the current app session.
  The daemon is not installed as a Windows service, and its approximately 1–1.5 GB
  memory use is disclosed in Settings. One-shot scanning remains available if daemon
  startup fails.
- Quarantine a ClamAV detection after a separate confirmation. The app stores a
  hash-verified copy and restore manifest; restoration refuses to overwrite a file.
- Export findings as SARIF for code-scanning tools.
- Publish deterministic Batch Review output into the Reports window's **Dev Toolkit** section.
- Use File Trust Evidence for bounded SHA-256, published-checksum comparison, PowerShell/Windows signature checks where supported, entropy/blob/polyglot indicators, and an offline organization-reviewed hash record.
- Classify Regex Lab input before matching so encoded data is routed to a decoder/evidence view instead of the regex/ReDoS path.
- Export and verify passphrase-signed, hash-chained evidence packs from redacted
  saved reports. These support integrity and provenance checks only; they do not
  provide regulatory certification or externally trusted signer identity.
- Re-run scanner-only static checks against the exact original scope to see whether
  a finding is still detected, not reproduced, or inconclusive. A missing pattern
  is not proof that a fix is complete.
- Enable app-level offline mode to block Cloud, public search, non-loopback model
  endpoints, remote Semgrep rule resolution, and model/signature downloads. This is
  not an OS firewall or proof of a true air gap.
- Use an Ollama first-pass model for bounded review, with optional escalation to a
  second installed local model for medium/high requests. The Settings screen can list,
  pull, remove, and tune local model profiles.
- Keep chat history on this device, reopen and rename conversations, and edit the
  latest user prompt to branch a revised conversation.
- Ask the Assistant about saved audit reports by selecting a report to attach. Cloud
  mode asks before sending that bounded report context. The history and sources panels
  collapse automatically to keep the chat workspace open; answer sources and limits
  appear in the details panel.
- Automatically retain redacted scan summaries locally as learning memory, inspect or
  clear them from the **Memory** window, and create a reusable Ollama profile with
  **Train**. This builds a prompt profile; it does not fine-tune model weights.
- Use the **Skills** screen to understand each audit stage and create reusable local
  review workflows. A selected workflow guides model analysis without adding targets
  or running commands.
- Read about Strix and other open-source components from **About**.
- Follow scan elapsed time, active stage, checked/skipped files, findings, and completed
  checks in **Run Monitor**. Findings, check coverage, and evidence have separate report
  views to make more room for each.
- Select **Cloud only** to use OpenRouter directly. Cloud mode never probes or falls
  back to local models. A scan asks before sending bounded evidence to the provider.
- Keep **Local** selected for on-device model review. Optional web search sends search
  terms to the search provider but does not switch the selected model provider.
- In **Settings**, app-level offline mode suppresses external requests initiated by
  SecAudit. Offline update bundles are currently integrity-checked but are not
  installed by the app; enforce network egress restrictions in the operating system
  for a true air gap.
- Keep Strix web-application testing as an advanced, separately scoped workflow. It
  requires an authorized target and the Strix runtime. A local folder scan is not a
  penetration test.

The AI models help explain and prioritize evidence from scanners. They do not
establish that a device is clean, and generated remediation is advice for a person to
review. SecAudit does not train models on scanned files.

## Quick start

### Windows

1. Install Python 3.12 and Ollama. Install ClamAV if you want malware
   signature scanning.
2. Open PowerShell in this folder and run:

   ```powershell
   py -3.12 -m venv .venv
   .\.venv\Scripts\Activate.ps1
   python -m pip install --upgrade pip
   pip install -r llm_app/requirements.txt
   py -3.12 -m pip install --user semgrep==1.182.1
   ollama pull qwen2.5:3b
   ollama pull qwen3:14b
   python -m llm_app.main
   ```

Semgrep is resolved only through its verified Python 3.12 installation. Its
availability requires a successful bundled-fixture scan, not only `--version`.

To build a Windows executable, install Python 3.12 with the Windows `py` launcher,
then run `.\scripts\build_windows.ps1` from PowerShell.
The distributable is created at `dist\SecAudit\SecAudit.exe`; share the
whole `dist\SecAudit` folder, not the executable by itself. Settings and local
model files are stored under the current user's app-data directory.

### Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r llm_app/requirements.txt
ollama pull qwen2.5:3b
python -m llm_app.main
```

### Authorized university lab tools

For the controlled coursework lab only, review and run `scripts/setup_tools.sh`
on the Linux VM that owns the lab network. It installs pinned syllabus tools from
apt, official Go modules, and pip. The script does not authorize a target or
start a scan. Keep `lab_targets.txt` beside the application, add only the IPs or
CIDRs of deliberately vulnerable VMs you control, and review the authorization
banner before every network-capable action. SecAudit rejects and logs targets
outside that file.

The registry distinguishes the ProjectDiscovery `httpx` executable from the
Python `httpx` library. Every tool is reported as `available`, `not_installed`,
or `broken` using its real binary path and version response; missing tools are
never represented by mock results. Public internet, arbitrary third-party
systems, and production targets are out of scope.

### Lab tool registry

| Tool | Purpose | Version check | Install hint |
|---|---|---|---|
| ffuf | Web content fuzzing | `ffuf -V` | `go install github.com/ffuf/ffuf/v2@v2.3.0` |
| httpx (ProjectDiscovery) | HTTP probing | `httpx -version` and `Current Version` output required | `go install github.com/projectdiscovery/httpx/cmd/httpx@v1.7.0` |
| nuclei | Vulnerability templates | `nuclei -version` | `go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@v3.11.1` |
| subfinder | Passive subdomain discovery | `subfinder -version` | `go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@v2.17.0` |
| katana | Web crawling | `katana -version` | `go install github.com/projectdiscovery/katana/cmd/katana@v1.8.0` |
| naabu | Port discovery | `naabu -version` | `go install github.com/projectdiscovery/naabu/v2/cmd/naabu@v2.6.1` |
| nmap | Lab port/service scan | `nmap --version` | Official Nmap installer |
| sqlmap | Conservative lab SQLi checks | `sqlmap --version` | Official SQLMap distribution |
| semgrep | SAST | Version plus bundled-rule fixture scan | `py -3.12 -m pip install --user semgrep==1.182.1` |
| clamscan | Malware signatures | `clamscan --version` | Official ClamAV installer |

SecAudit resolves known Windows paths first, refreshes the current user PATH,
verifies the real binary, and only then enables a tool. Semgrep additionally requires
a successful static fixture scan; ClamAV requires a successful custom-signature
functional probe and a current app-managed database before the malware check is
available. Each lab-tool action
uses a fixed argument array, bounded output/time, and the allow-list in
`lab_targets.txt`. The UI displays an authorization banner and requires a
confirmation immediately before the command starts.
For Python user installs, the resolver also checks
`%APPDATA%\Python\Python315\Scripts` before `C:\Python315\Scripts`; this is
the expected location for the user-installed SQLMap executable on this workstation.
Those Python 3.15 paths are not used to resolve or run Semgrep.

In Local mode, Ollama must be running locally. The app uses the selected local model
for the first pass and can escalate through the same Ollama host. The backend is
vendor-neutral Python and does not require NVIDIA NIM, NeMo, or CUDA. Scanner results
remain available without a model; the report marks model review incomplete when a
selected model is unavailable.

## First scan

1. Open **New Audit** in the horizontal navigation and describe what you want to review.
2. Enter a target path, browse for one file or folder, or add several files or folders.
3. Review the selected scope, then start the read-only scan.
4. Check **Run Monitor** for progress and **Reports** for findings. Export SARIF or save an annotated Markdown copy.

Code-pattern checks cover selected text formats and skip common generated or
dependency folders. Batch Review accepts up to 200 files, at most 4 MiB per file and
64 MiB total, and skips larger files with an explicit reason. It reads inputs in
64 KiB chunks. ClamAV has its own file-size and scan limits. A completed scan is
limited to the files and checks listed in its report; it is not proof that the system
is free of threats.

## Models and privacy

- **Local models:** Ollama runs on this computer. Local mode does not require an API key.
- **Cloud model:** OpenRouter is optional. Add a key in Settings only if you want to
  use it. Do not send confidential source code or incident evidence to a cloud model
  unless your organization permits it.
- **Web search:** search terms are sent to the configured search provider. Use offline
  mode for private material.
- **Training:** scanned files are not used to train or fine-tune a model by this app.

## Project map

```text
llm_app/                 Native PyQt6 desktop UI, scanner adapters, model clients
strix/                   Strix report, SARIF, runtime, and skill support used by draft
cyber-guardian/           Separate upstream Strix checkout and its own Git history
docs/                    Installation, architecture, and scanner behavior
scripts/                 Setup helpers
```

The nested `cyber-guardian/` checkout is kept separate from this app. Its instructions
and documentation describe upstream Strix; they are not application prompts and do
not authorize scans of any target. Follow this repository's scope confirmation before
using any active testing feature.

## Limitations and help

This project is under active development. It is not a replacement for endpoint
protection, backups, or a qualified incident responder. See [Installation](docs/installation.md),
[Architecture](docs/architecture.md), and [Strix integration](docs/strix_integration.md)
for details.

## Ownership and acknowledgments

SecAudit is proprietary software. The repository's `LICENSE` applies only to
SecAudit-owned work and does not replace the licenses of third-party software,
dependencies, or the separately maintained `cyber-guardian/` Strix checkout.
SecAudit acknowledges the open-source projects it integrates with or references,
including [Strix](https://github.com/usestrix/strix),
[Ollama](https://github.com/ollama/ollama),
[Semgrep](https://github.com/semgrep/semgrep),
[PyQt6](https://pypi.org/project/PyQt6/), and
[ClamAV](https://github.com/Cisco-Talos/clamav). Their respective notices and
license terms remain applicable to those components.
