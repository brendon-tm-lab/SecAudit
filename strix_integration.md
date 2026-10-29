# Strix and scanner integration

**Updated: 2026-10-29.** This describes the current SecAudit wrapper. It is not a
description of the separate upstream Strix checkout.

## What the current folder scan does

The class in `llm_app/core/strix_engine.py` is a wrapper used by the existing draft.
Its code-pattern checks are local heuristics. They are not the Strix autonomous
penetration-testing engine, and they do not prove a folder or file is free of
vulnerabilities or malware. If ClamAV is installed, the wrapper can additionally ask
`clamscan` to inspect selected files or folders for known malware signatures. When
Semgrep is installed and functionally verified, the wrapper invokes its bounded JSON
scan with the remote `p/r2c-security-audit` pack when online, or bundled local rules
when offline. A remote fetch failure is retried using bundled rules and marked
`local-fallback`. Only confirmed paths are passed to either scanner. The report
records check status, selected scope, scan limits, scanned/skipped paths, duration,
errors, engine verification, and any fallback.

The wrapper exports its findings through the report helpers in the top-level `strix/`
package. SARIF carries scanner results to code-review systems; it is not evidence that
the underlying checks are complete.

## Upstream Strix checkout

`cyber-guardian/` is a separate checkout with its own Git history and agent guidance.
It provides the upstream Strix application. The local desktop wrapper does not
currently invoke that checkout when a user runs a folder scan. Keep changes to the
upstream checkout separate from the SecAudit wrapper, and do not copy its
agent-facing usage instructions into the app's system prompt.

When a dedicated Strix pentest adapter is added, it must:

1. Show the exact URL, host, or source tree that Strix will inspect.
2. Require the user to confirm that they own the target or have written permission.
3. Keep any network test within the confirmed scope and use the Strix sandbox.
4. Show the selected LLM provider and whether source or findings will leave the
   machine.
5. Save results under a separate run folder and clearly label incomplete or failed
   runs.

Do not use a directory malware scan as a substitute for Strix, or a Strix pentest as
a substitute for endpoint protection.

## Finding and SARIF fields

The wrapper preserves scanner rule IDs, severity, applicable CWE, file path, line
range, redacted evidence, and check status. Batch Review findings use a normalized
schema with correlated rules, plain-language behavior, parsed command arguments,
separate detection/threat certainty, MITRE techniques, nullable behavior-validated
CWE, and per-file coverage. CWE rule references alone do not set a CWE identifier.
Secret values must not be copied into logs or report snippets. SARIF remains
available when optional tools are missing, and reports state which checks actually
ran.

Batch Review's own local rule engine does not invoke Semgrep or ClamAV; each report
states `NOT RUN` for those scanners. Only the full audit flow can invoke the optional
adapters when their tools and prerequisites are present.

## Context and specialized analysis skills

The desktop review defaults to a 32,768-token Ollama context, configurable in
Settings at 8,192, 16,384, or 32,768, with a shared model temperature. Per-file model
evidence is capped and scaled to the selected context. Both model tiers use the same
local Ollama host and settings. This backend is vendor-neutral and does not require
NVIDIA NIM, NeMo, CUDA, or NVIDIA hardware. These are capacity limits, not claims about
model accuracy.

The nested upstream Strix project has optional agent-facing skills for telemetry
triage, telecom configuration review, and vulnerability analysis. Those skills remain
inside the nested checkout and are not automatically executed by the desktop wrapper.
The desktop uses its own read-only local scan, selected review focus, local Ollama
model tiers, and explicit report coverage. It does not claim to perform live network
testing, container extraction, CVE research, or automatic VEX decisions.

The desktop **Skills** screen provides its own built-in and user-authored review
profiles. These are bounded prompt guidance used only when selected for a folder
review; they are not Strix agent skills, executable plugins, or scanner rules.

Several built-in SecAudit review profiles adapt safe analysis ideas found in
Strix's skill material: static source/trust-boundary review, dependency evidence
review, counterevidence assessment, and static fix review. These are authored as
bounded SecAudit guidance; upstream skill files are not copied or loaded at
runtime. Active reconnaissance, exploit/PoC execution, tool invocation, and
agent-orchestration instructions are intentionally excluded. The profiles only
guide model interpretation of the confirmed scope and fixed scanner evidence;
they do not run commands, expand scope, or erase scanner findings. SecAudit does
not install the upstream Strix toolchain into the frozen executable and does not
claim that every upstream Strix function is available in the desktop wrapper.

Run Monitor displays background Semgrep and ClamAV probe results. Semgrep requires a
version response and a successful bundled-fixture scan; ClamAV requires a version
response plus a custom-signature detection and clean-fixture scan. ClamAV readiness
also reflects whether current app-managed signatures exist. Probes are persisted and
re-run after 24 hours, an engine change, or a scanner-level failure. The inventory
reports verified paths, versions, availability, and install guidance; it does not
invoke the upstream Strix engine.

The optional ClamAV daemon is owned by the active SecAudit session, uses a
loopback-only listener, and is never installed as a Windows service. Failed startup
falls back to bounded one-shot scanning with the reason included in scan details.
FreshClam updates are blocked in offline mode and cause active daemon reload/restart
and a new readiness probe. Batch Review has no path to invoke either optional scanner
and continues to label both `NOT RUN`.

At the user's explicit request, the reviewed custom documents under
`cyber-guardian/strix/skills/custom` are bundled as bounded SecAudit guidance.
They are not executable Strix plugins, cannot choose tools or targets, and cannot
grant permissions. The desktop wrapper still does not invoke the upstream agent
runtime for folder audits.
