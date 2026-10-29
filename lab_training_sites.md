# Authorized security-training references

The uploaded training-site list is treated as a reference catalog, not as an
automatic target list. SecAudit does not crawl, probe, or add these hosts to
`lab_targets.txt` automatically. Before using a network-capable adapter, the
operator must deploy or obtain written authorization for a lab instance and add
the exact approved IP range or hostname to the local allow-list.

The catalog includes intentionally vulnerable and training environments such as
OWASP Juice Shop, DVWA, WebGoat, bWAPP, Mutillidae II, Damn Vulnerable Web
Services, Hackazon, Metasploitable, VulnHub, TryHackMe, Hack The Box, PicoCTF,
OverTheWire, PentesterLab, PortSwigger Web Security Academy, Google Gruyere,
SEED Labs, Root Me, and the other references supplied in the uploaded source.

Links in the source list can change ownership, availability, or authorization
terms. SecAudit therefore displays them as training references only. Public
network access remains disabled by offline mode and is never inferred from a
skill, a URL pasted into the tool runner, or a model response.

## Evidence shown after a run

The Network > Tool runner tab records the fixed executable, argument list,
target, start/end status, exit code, output limits, and parsed events. A zero
exit code is not treated as proof that a scan found a host or service. The
Network > Diagnose network tab separately reports local-stack, route/gateway,
TCP/IP-count, DNS-configuration, and adapter-signal observations without
displaying private addresses, SSIDs, DNS server addresses, or connection peers.
## Local hosting and allow-listed targets

The lab runner accepts every IP, CIDR range, or domain explicitly present in `lab_targets.txt`, not only localhost. This permits authorized remote lab VMs and private projects while keeping the target boundary explicit. The runner does not infer authorization from the training catalog and does not add public sites automatically. Add only systems you own or have written permission to test, then confirm the exact target in the UI before running a fixed adapter.

Training applications should preferably be downloaded, reviewed, and self-hosted inside an isolated lab network. SecAudit can scan the resulting allow-listed host, but it does not download, install, execute, or detonate supplied training projects.

