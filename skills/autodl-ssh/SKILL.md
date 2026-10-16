---
name: autodl-ssh
description: Set up or diagnose SSH access to AutoDL GPU instances, including banner timeouts, Fake-IP DNS, keepalives, and connection reuse.
---

# AutoDL SSH

Work from the user's local computer. Collect the current host, port, user and authentication method from the user's AutoDL console or existing authorized configuration. Do not reuse the author's instance details. Use the user's language.

1. Inspect local DNS and SSH behavior. A successful TCP probe alone does not establish that the SSH banner or authentication works. The 198.18.0.0/15 range is a proxy Fake-IP, not an instance endpoint.
2. For key-based access, use `scripts/autodl-ssh-check --host HOST --port PORT --user USER --identity KEY`. It compares system and HTTPS DNS, tests fresh connections and a private reused master, and can test idle survival with `--idle-seconds 60`. DNS is routing information, not host authentication.
3. Keep host-key verification enabled. For a new instance, independently confirm the fingerprint through a trusted source before enrollment. The helper defaults to strict checking; use `--accept-new` only after that verification. A changed key requires investigation, not deletion of known_hosts or disabling checks.
4. For password-only access, use an interactive SSH terminal. Never request passwords in chat or put them in arguments, environment variables or files. The key-only helper does not support passwords. If unattended recovery is wanted, establish authorized key access first.
5. After successful diagnostics, offer a host-specific SSH alias using the current verified endpoint, `HostKeyAlias [HOST]:PORT`, explicit IdentityFile, `IdentitiesOnly yes`, `ServerAliveInterval 20`, `ServerAliveCountMax 3`, `ControlMaster auto`, `ControlPersist 10m`, and `ControlPath ~/.ssh/cm-%C`. Preserve unrelated configuration. Create ~/.ssh with mode 700 if needed.
6. Re-check endpoint and key when an instance is rebuilt. Report fresh, reused and idle test outcomes separately. Do not claim permanent stability from a successful sample.

Do not source AutoDL's `/etc/network_turbo` on the local computer: it addresses remote outbound downloads, not inbound SSH. For Codex outbound networking, use the bundled autodl-codex skill. No credentials or private keys may be bundled or copied between computers.
