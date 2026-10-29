# Installation

**Updated: 2026-10-29.** This guide covers the current Windows/Linux setup for the
top-level SecAudit application.

SecAudit is a Python desktop application. The Windows and Linux setup below
installs the application dependencies. Ollama, a local model, and ClamAV are optional
services that must be installed separately.

## Requirements

- Python 3.11 or newer for the application; Python 3.12 is required for the pinned
  Windows Semgrep installation
- A desktop session supported by PyQt6 (the interface uses native Qt widgets)
- Ollama and at least one installed local chat model
- A second installed local Ollama model for medium/high escalation (optional)
- ClamAV (`clamscan`) for malware signature checks
- Semgrep 1.182.1 for local SAST; on Windows, install and run it only with Python 3.12

The app checks which local models are installed. If a saved model is missing, it
selects an installed model for the current session and tells you which one it chose;
save Settings to keep that selection. Without Ollama or an installed model,
deterministic checks still run and AI review is marked incomplete. Without Semgrep or ClamAV signatures, those checks are marked unavailable, never
clean. Semgrep becomes available only after both its version check and a real scan of
a bundled-rule fixture succeed. ClamAV becomes engine-verified only after a local
custom-signature detection and clean-fixture probe; a recent app-managed signature
database is also required for an audit.

The Dev → Batch Review workflow is standalone local static analysis. It processes at
most 200 files, 4 MiB per file, and 64 MiB total, reading file data in 64 KiB chunks.
Over-limit files are explicitly skipped. Semgrep and ClamAV are not run in Batch
Review; the report labels both `NOT RUN`.

## Windows

Open PowerShell in the repository root:

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

The app's startup probes run in background workers after the window appears. Run
Monitor and Settings show the last persisted Semgrep and ClamAV status. Probe results
expire after 24 hours or when an engine version changes. A version-only response does
not make a scanner available.

Open **Settings → Local model management**, refresh the installed model list, then
choose the first-pass and medium/high local models. The assistant also has a model
selector and saves chat history locally. Pull, delete, and tune operations target the
configured Ollama host; deleting a model requires confirmation.

The **Skills** screen describes the audit stages and the purpose of each built-in
review profile. Use **Create skill** to save up to 30 of your own plain-language
review workflows on this device, then select one in **New Audit**. Custom profiles
are model guidance only; they do not add scan targets, run commands, or change which
scanner checks are available. The **About** screen links to the Strix, Ollama, Semgrep,
PyQt6, and optional OpenRouter projects.

**File review** provides a bounded SHA-256 digest, file-type hints, and offline
Authenticode status where supported. It never runs the selected file. Add a file to
the explicitly reviewed New Audit scope to use available static checks and optional
ClamAV. A Python venv is not an execution sandbox, so unknown `.exe`, `.ps1`, `.bat`,
`.cmd`, and `.sh` files are not launched by the app.

**Network** shows active interface addresses and measured upload/download byte rates
while the page is open. The optional diagnostics are read-only and run only after
confirmation: local-stack, gateway, public-IP ping, and DNS checks. They do not reset
Winsock/IP, renew DHCP leases, change firewall settings, or run elevated commands.
Network collection runs in background workers; Windows system commands use hidden,
bounded, fixed-argument processes and do not open a console window. The **Reports →
Network** tab displays a live local network snapshot and any confirmed diagnostic
results. Network data is not added to exported audit copies.

Audit and maintenance CLI tools also run in background workers without opening a
terminal window. Their output is captured by the app rather than written into a
terminal; POSIX commands start in a separate process session. Bounded subprocesses
stop at their configured timeout, and scans finalize the report after those checks
return. The local Ollama server is a persistent background service and remains
available beyond one audit run.

The Assistant can attach a saved audit report to a question. In Cloud mode, the app
asks before sending that report context. Each scan automatically saves a redacted
summary in **Memory** for future Local reviews; raw source files and excerpts are not
part of learned memory. The **Train** button creates an Ollama profile from aggregate
memory and selects it for medium/high local review. It does not update model weights
and works only with Local selected and a loopback Ollama host. You can remove selected
memories, clear all summaries, or clear saved reports separately.

The Assistant can also import a local JSON, SARIF, Markdown, or text report and
optionally attach it to a chat question. Imported context is capped, credential-like
values are redacted, and Cloud mode asks before sending it. This imported report is
temporary and is not added to the saved audit archive.

**App-level offline mode** is available in **Settings → Enable app-level offline mode**.
Save Settings to persist the choice. It blocks Cloud requests, public search,
non-loopback Ollama hosts, remote Semgrep rule-pack resolution, model pulls, and
ClamAV signature downloads. Offline scans can still use local pattern checks,
available local ClamAV signatures, and loopback Ollama models. Confirmed network
diagnostics then test loopback only. This application policy is not an OS firewall:
other processes, host services, and operating-system traffic may still use the
network. Use Windows Firewall or equivalent network controls when a true egress
restriction is required.

**Enterprise → Verify latest fix** re-runs static checks in a worker on the exact
scope captured with the prior findings. It does not call a model or resolve the
remote Semgrep ruleset. The result says `still detected`, `not reproduced in static
re-check`, or `inconclusive`; none of these statuses proves that software is safe or
that a vulnerability is exploitable.

**Reports → Export signed evidence pack** creates a ZIP from the latest redacted saved
report and its SARIF view, appending a locally signed hash-chain entry. On first use,
choose a passphrase for the encrypted local Ed25519 key; later exports require it to
unlock that key. Back up the key and remember the passphrase—if the key is lost while
its chain exists, SecAudit refuses to silently replace it. **Verify evidence pack**
checks the signature, content hashes, and included chain. Because the public key in
the pack is self-signed, verify its fingerprint separately before trusting signer
identity. The pack does not provide external timestamping, compliance-framework
mapping, or SOC 2/ISO certification.

Enterprise offline update bundles can be created and their manifest/signature
integrity checked. The verifier does not install or apply the bundle and the embedded
public key does not establish signer identity. Stage and authenticate updates using
your organization's trusted offline distribution process.

If PowerShell blocks virtual-environment activation, run the interpreter directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r llm_app/requirements.txt
.\.venv\Scripts\python.exe -m llm_app.main
```

To build the Windows desktop application, run
`.\scripts\build_windows.ps1` from the repository root. The script creates a local
build environment and writes the executable to `dist\SecAudit\SecAudit.exe`.
Distribute the complete `dist\SecAudit` folder. New user settings and local model
files are stored under `%LOCALAPPDATA%\SecAudit`. If a prior `%LOCALAPPDATA%\CyberGuardian`
or Linux `cyber-guardian` data directory exists and no SecAudit data directory exists,
the app continues to use the existing directory in place; it does not move or delete
user data during the rename.

## Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r llm_app/requirements.txt
ollama pull qwen2.5:3b
python -m llm_app.main
```

Start Ollama using the method provided by your installation. If the model is not
available, the app will say so when an AI explanation is requested.

## Optional cloud model

Select **Cloud only · OpenRouter** and add an API key in Settings. Cloud chat uses
OpenRouter directly. Before a scan sends bounded source excerpts and scanner evidence,
the app asks for confirmation. Local models are not probed or used in Cloud mode, even
if the cloud key/request fails. Keep Local selected for on-device model review. Do not
store a real key in a tracked file.

## Optional ClamAV

Install ClamAV using its official instructions for your operating system, then update
its signature database. On Windows, SecAudit keeps signatures in its app-managed
database and updates them through the Settings action using a fixed FreshClam
executable and argument list. App-level offline mode blocks this download. Every
scan records the engine version, database build and age, scan mode, duration, coverage
counts, and any daemon fallback reason. Malware scanning is unavailable when
signatures are missing or older than 72 hours. ClamAV receives only confirmed target
paths; the app does not invoke a shell or run files from the target.

Settings can keep a ClamAV daemon warm for the current app session. It binds only to
127.0.0.1, is not installed as a Windows service, and uses approximately 1–1.5 GB of
RAM. The default is enabled on machines with at least 8 GB of memory and disabled
below that threshold. If daemon startup fails, the scan falls back to one-shot mode
and records the reason. Closing SecAudit stops the daemon.

The Windows `scripts/setup_tools.ps1` setup installs pinned Semgrep with `py -3.12`,
unblocks and compile-checks its package, and runs two static fixture scans while
logging cold/warm durations. It prefetches the online ruleset only when app-level
offline mode is not set. `scripts/prepare_authorized_lab_host.ps1` is an elevated,
lab-only helper that applies scanner-specific Defender exclusions; it excludes no
scan targets. Review its effects before running it because any Defender exclusion
reduces protection for the excluded scanner files and local data.

## Launcher scripts

The shell scripts under `scripts/` are for Linux and compatible shells. On Windows,
use the PowerShell commands above.

## Troubleshooting

- **Qt import error:** confirm `PyQt6` is installed into the active virtual
  environment.
- **Ollama is unavailable:** start Ollama and confirm the configured local host is
  reachable. Folder scanning does not require Ollama.
- **ClamAV is unavailable:** confirm `clamscan --version` works in the same terminal
  used to launch the app. Code-pattern checks continue without it.
- **Cloud request/privacy concern:** remove the API key or enable offline mode before
  submitting private source or incident data.
## Local model context

The app sends a bounded `num_ctx` value to Ollama for local inference. The default
is 32,768 tokens; context and temperature are selected in the **Tune from selected
model** profile dialog. Both local models receive the configured context and shared
temperature on requests. Larger contexts use more memory
and remain limited by each model's actual context support and available hardware.
Context length is not an accuracy guarantee. Audit setup offers general, telemetry,
telecom-configuration, and vulnerability-inventory focus profiles; these guide bounded
model review and do not change scanner behavior or scan scope. User-created review
workflows are saved in `workflow_skills.json` under the app's settings directory.

## Privileges and elevation

SecAudit runs ordinary read-only audits without administrator privileges. The
desktop wrapper does not silently elevate, install arbitrary tools, or run supplied
scripts. The Network tab displays whether the current
process is a standard user or administrator. Custom skills cannot change that
state and cannot grant tools access.

The controlled permission profiles are read-only inspection, workspace access,
approved local tools, approved desktop interaction, specific hardware access,
and elevated operation. Only a future fixed adapter may request the last
profile, and it must disclose the exact executable, arguments, target scope,
files it may write, and why elevation is needed before requesting Windows UAC.
There is no blanket administrator manifest or hidden relaunch. Denied or failed
elevation is reported as incomplete, never as success. Approved lab executions
are written to a redacted `execution_audit.jsonl` file in local app data.

On Windows, the application and its children are contained in a Job Object configured
to terminate the child process tree when SecAudit exits. Graceful close requests
worker cancellation and terminates known children. A hard Task Manager termination
cannot be intercepted; the local scan journal records an interrupted run for
follow-up, and the user must confirm the scope again.

## University lab tool setup

On the Linux lab VM, inspect `lab_targets.txt`, then run:

```bash
bash scripts/setup_tools.sh
```

The script uses apt for Nmap, SQLMap, ClamAV, Go, and pip; official Go module
paths for ProjectDiscovery tools and ffuf; and a pinned Semgrep package. Version
variables at the top of the script can be reviewed or overridden before use.
SecAudit does not install tools from inside the desktop UI. The Run Monitor
checks the actual executable path and version and reports a missing install
command when a tool is unavailable. Python's `httpx` package is not accepted as
the ProjectDiscovery `httpx` binary.

The Network page is split into Connected network, Diagnose network, and Tool runner
tabs. The tool runner supports an explicit UAC-session request for five fixed
adapters (`nmap`, `sqlmap`, `clamscan`, `semgrep`, and `ffuf`). Windows displays the
UAC prompt; SecAudit does not bypass it, run arbitrary commands, or elevate silently.
See [authorized training references](./lab_training_sites.md) before adding a
written-authorized lab host to `lab_targets.txt`.
