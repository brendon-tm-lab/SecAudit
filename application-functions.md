# SecAudit application functions

**Overview updated: 2026-10-29.** This page reflects the current top-level desktop
application; it does not describe the separate upstream Strix runtime.

This guide describes the current top-level SecAudit desktop application. Folder audits
and Dev Toolkit Batch Review are bounded, read-only static analysis; supplied files
are not executed. Optional scanners run only in the full audit flow and report their
actual status. Batch Review does not invoke Semgrep or ClamAV and labels both `NOT RUN`.

The app does not execute supplied code or expand the selected scope. Offline mode
blocks app-initiated Cloud, public-search, non-loopback model, remote Semgrep rule-pack,
and model/signature download requests. It is not an operating-system firewall or
proof that no network traffic left the machine.
## Purpose

SecAudit is a native desktop workspace for reviewing selected files and folders
for security-relevant code patterns, static-analysis findings, and (when available)
known malware signatures. It collects findings and check-coverage information into
reports, and can use a selected AI provider to explain bounded evidence.

Scans are read-only: the app does not execute or modify the selected files. A local
scan is not a penetration test and cannot prove that a file or project is safe.

## Main application screens

| Screen | What it does |
| --- | --- |
| **New Audit** | Lets you select up to 20 existing files or folders, remove or clear selected targets, enter a review objective, choose a review focus, review the scope, and start a read-only scan. The objective and focus guide model analysis; neither changes the selected paths or scanner tools. |
| **Dev → Batch Review** | Analyzes selected files/folders with bounded local rules, argument-aware command context, correlated same-behavior detections, redacted line-numbered evidence, and per-file rule/scanner coverage. Semgrep and ClamAV are `NOT RUN` in this workflow. |
| **File review** | Inspects one regular file without running it. Shows its size, a SHA-256 digest when it is within the 256 MiB limit, common file-type hints, and offline Windows Authenticode status for supported Windows formats. You can add the inspected file to the New Audit scope for additional checks. A valid publisher signature is not a safety verdict. |
| **Network** | Displays active local interface addresses, available gateway/Wi-Fi details, OS-reported link speed, and upload/download rates measured from interface byte counters. Optional, confirmed diagnostics test loopback, the default gateway when available, public-IP reachability, and DNS lookup. App-level offline mode limits diagnostics to loopback. It does not change network configuration or run a speed test. |
| **Run Monitor** | Shows the current audit stage, elapsed time, selected targets, file coverage, findings, and completed checks. Semgrep and ClamAV status chips show persisted background probe results. Missing tools, skipped files, limits, and failures are reported as coverage/status information rather than silently treated as clean results. |
| **Reports** | Presents findings, check coverage, and audit details. The report identifies which checks ran and which were skipped or incomplete, including bounded supplier-intake and automation indicators performed in the scan worker. It can export SARIF, create and verify a signed/hash-chained evidence pack from the redacted saved report, save a separately annotated Markdown report copy, and—after separate confirmation—quarantine a ClamAV-detected file or restore a quarantined file. A separate Network tab shows the live network snapshot and confirmed diagnostic results; that snapshot is not added to exported audit copies. |
| **Assistant** | Provides local or cloud chat for security questions, findings, or remediation planning. You can manage chat history, optionally attach a saved or imported report, and allow public web search for a request. The details panel shows provider/model routing, sources, and assumptions for the latest answer. |
| **Skills** | Offers built-in static source/trust-boundary, dependency-evidence, counterevidence, and fix-review profiles, and lets you create, edit, or delete up to 30 locally saved review workflows. Guidance is model-only: it does not add scanner rules, execute tools, or broaden scan scope. |
| **Settings** | Selects the exclusive Local/Ollama or Cloud/OpenRouter provider, configures models and host, manages local models, tunes a local model profile, runs on-demand Semgrep/ClamAV warm-up probes, updates app-managed ClamAV signatures, and enables app-level offline mode. It can keep the loopback-only ClamAV daemon warm for the current session; the default is on at 8 GB RAM or above, off below, and the optional daemon uses about 1–1.5 GB. Cloud requests do not probe or fall back to local models. |
| **About** | Shows provider and local model-routing information and links to the project's runtime/integration acknowledgments. It clarifies that SecAudit does not invoke the upstream autonomous Strix engine for folder scans. |

## What an audit checks

For the exact files/folders the user selected and confirmed, the wrapper may run:

1. **Local code-pattern checks** for common security risks in supported text and
   script files. These are heuristics; a match needs review and can be a false
   positive.
2. **Semgrep**, when installed and verified by both a version check and a bundled
   fixture scan. Online scans may resolve `p/r2c-security-audit`; offline scans use
   bundled local rules only. Results include scanned/skipped paths, duration,
   rules-source, errors, and warm-up details.
3. **ClamAV**, when installed with sufficiently recent signatures, to look for
   known malware signatures. Missing or stale signatures make the check unavailable;
   that is not a clean result. A local custom-signature probe verifies the engine;
   current app-managed signatures are required for an audit. The optional daemon binds
   only to `127.0.0.1`; one-shot fallback and its reason are reported. The report
   compares ClamAV's scanned-file count with the bounded intake count and marks
   missing summaries or count mismatches as partial/incomplete, never clean.
4. **File signature evidence** where supported, including local Authenticode checks
   for Windows executable formats.
5. **Optional model review** of bounded file excerpts and scanner findings. In Local
   mode, a first-pass Ollama model may send notable cases to a second configured
   model. In Cloud mode, bounded evidence is sent to OpenRouter only after the user
   confirms the disclosure.

The app applies size, file-count, archive, and model-context limits. Unsupported or
over-limit files and incomplete checks are reflected in report coverage. Model
observations are advisory and do not erase scanner findings or establish that a
finding is exploitable.

Batch Review accepts at most 200 files, reads at most 4 MiB per file and 64 MiB total,
and reads file contents in bounded 64 KiB chunks. It runs on a one-thread Qt worker
pool; progress is coalesced, cancellation is checked between files, and a local
recovery journal records the selected paths and completion state. Larger files are
reported as skipped rather than partially described as clean.

Closing the desktop window during active work requests cancellation, stops child
processes, and gives workers a bounded shutdown window rather than waiting on a
confirmation dialog. Windows child processes are held in a kill-on-close Job Object;
hard termination is recovered through the local scan journal on the next launch.

## Findings, reporting, and follow-up

- Findings include available rule/source information, severity, location/evidence,
  description, and remediation guidance.
- Batch Review findings use a stable schema for analyzer, rule/correlated rules,
  file/line range, category, plain-language behavior, redacted excerpt, parsed
  command arguments, detection certainty, threat certainty/reason, MITRE techniques,
  nullable behavior-validated CWE, severity, and status. Detection confidence does
  not imply malicious intent.
- Batch Review severity and behavior use parsed arguments for supported command
  families. Scheduled-task queries are informational; high-impact task creation and
  security-control tampering are distinguished from simple command presence. Windows
  elevation primitives, registry changes, and persistence operations have separate
  rule IDs and rule-owned categories; compound `&`, `&&`, and `||` statements are
  parsed independently, with batch-variable aliases resolved only from definitions
  available before the statement.
- The Batch Review text view groups consecutive findings with the same behavior into
  a line-range block and reports its sub-match count. The structured result retains
  each per-line finding, including correlated rule IDs, for traceable export.
- File-type coverage is rule-scoped: available rules cover common Windows and Unix
  scripts, Python, JavaScript/TypeScript, SQL, XML, JSON/YAML and text configuration.
  Unknown text extensions receive only applicable generic rules; binary, malformed,
  or over-limit inputs are reported as skipped or partially checked rather than
  represented as clean.
- CWE identifiers are emitted only for validated behavior classes. A hardcoded IP
  address, scheduled task, download, or privilege operation does not receive an
  unrelated CWE identifier.
- SARIF export supports sharing results with compatible code-review tools. It retains
  scanner/review status so that exported findings do not imply every check completed.
- A signed evidence pack contains the redacted saved report, corresponding SARIF,
  integrity hashes, and a signed local hash-chain ledger. The Ed25519 signing key is
  encrypted with a user-provided passphrase and stored in app data. The pack can be
  verified inside SecAudit; compare the public-key fingerprint through a separate
  trusted channel before relying on signer identity.
- Evidence packs are a provenance/integrity feature only. The embedded public key is
  self-signed, there is no external timestamp authority or regulatory control mapping,
  and the pack does not certify SOC 2, ISO 27001, or cyber-insurance answers.
- An analyst annotation is saved into a **new Markdown copy**; it does not rewrite
  the original findings or an existing report copy.
- Threat Log triage labels are session-only annotations. They do not change scanner
  findings, SARIF output, or selected files.
- Quarantine is an explicit, separate action limited to regular files marked by
  ClamAV. It requires user confirmation, verifies hashes, and does not overwrite an
  existing file during restore.
- Saved reports and redacted aggregate review memory are stored locally. The app
  bounds retention to 100 reports and 500 memory records. Learned memory excludes
  complete source files and raw excerpts.
- The **Train** action creates a local Ollama prompt profile from aggregate saved
  summaries; it does not fine-tune model weights. It is restricted to Local mode
  with a loopback Ollama host.

## Assistant and provider behavior

- **Local** mode uses the configured Ollama host and model(s). Local scan memory may
  be used as bounded, untrusted context.
- **Cloud** mode uses OpenRouter exclusively. A selected report is sent only after
  the app asks for disclosure confirmation. Local memory is not sent to Cloud, and
  Cloud failures do not trigger local-model fallback.
- Imported JSON, SARIF, Markdown, or text reports can be attached temporarily to an
  Assistant request. Imported context is bounded and redacted, is treated as
  untrusted data, and is not added to the saved audit archive.
- Public web search is optional per request. When used, source URLs and relevant
  assumptions are shown in the Assistant details.
- Model output is advisory. It cannot choose arbitrary commands or tools, modify
  target files, or authorize changes.

## Scope and safety boundaries

- Only explicitly selected and confirmed paths are scanned; there is no implicit
  default target.
- Scanner operations run locally with fixed integrations, bounded execution, and no
  shell command constructed from model output.
- The app does not run executables, installers, scripts, or archive members as part
  of File review or a scan.
- `cyber-guardian/` is a separate upstream Strix checkout. The SecAudit wrapper does
  not invoke it for folder scans. A local source review must not be described as a
  live network test or penetration test.
- The optional Network screen is host troubleshooting, not packet capture or a
  security assessment of a network.
- Missing optional tools, provider failures, partial model coverage, and skipped
  checks must remain visible. None is evidence that the target is clean.
- Run Monitor shows background scanner readiness: Semgrep needs a successful fixture
  scan, and ClamAV needs a successful custom-signature/clean-fixture test. It does not
  install or invoke the upstream autonomous Strix engine.
- The built-in Skills list includes bounded Strix reference guidance; this
  content cannot execute tools or expand the confirmed scope.
- Network-capable lab checks display an authorization banner, accept only IPs
  from `lab_targets.txt`, and log rejected out-of-scope targets. Missing tools
  show `not_installed` and their reviewed install command.

The Network page is horizontally organized into Connected network, Diagnose
network, and Tool runner tabs. Diagnostics collect only bounded local data and
authorized lab checks; private addresses, SSIDs, DNS server addresses, and peer
identities are hidden. Offline Semgrep uses the bundled local rules rather than
resolving the remote ruleset. Public training links are catalog references only;
they never become implicit targets.

## Related documentation

- [Installation and setup](./installation.md)
- [Architecture and scan boundaries](./architecture.md)
- [Strix and scanner integration](./strix_integration.md)
- [Evaluation against commercial products](./evaluation.md)
## Batch review and assistant imports

Batch Review keeps the selected script/file list explicit. Each selected item can be removed with `× Remove selected` before analysis, and the detailed report includes the actual status, start/end timestamps, duration, file metadata, findings, skipped items, limits, evidence sources, and recommended actions. The review remains bounded and read-only.

Assistant `Import` accepts reports and supported source/script formats, including PowerShell, CMD/BAT, shell, Python, JavaScript/TypeScript, Java, C/C++, Go, Rust, PHP, HTML/CSS, SQL, JSON, YAML, XML, and related text formats. Each imported file is capped at 1 MiB; imported content is untrusted evidence and is never executed. Imported context can be removed with `× Remove imported`.

Batch analysis uses the same bounded intake, metadata, shared-rule, and type-aware rule pipeline for each supported file. It reports unsupported, binary, malformed, oversized, and archive-limited states explicitly. ZIP central-directory metadata is inspected without extraction; traversal members and excessive expansion are reported as findings. Semgrep and ClamAV remain specialist checks in the full audit path and are reported as unavailable/not run by Batch Review rather than being represented as clean results.

Settings now exposes the current Standard user versus Administrator process state. Administrator mode is only started through a visible Windows UAC prompt; it is not a blanket permission grant and does not bypass UAC.
