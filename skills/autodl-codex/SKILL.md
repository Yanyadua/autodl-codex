---
name: autodl-codex
description: Install, configure, diagnose, and recover Codex CLI on AutoDL Linux GPU instances, including optional local HTTP proxy forwarding and screen session persistence.
---

# Codex CLI on AutoDL

Use the user's language. First identify whether commands run on the local computer or remote AutoDL Linux instance. Collect only missing non-secret connection parameters, work directory and proxy settings. Check existing setup before changing it.

Read [the setup and recovery workflow](references/setup.md) for the applicable steps. For inbound SSH failures, use the bundled autodl-ssh skill first. For outbound failures, diagnose local proxy, SSH reverse tunnel, remote proxy, Codex authentication, then the actual Codex request. Skip proxy setup when direct access already works.

Never read, copy or bundle auth.json, private keys, tokens, passwords or proxy subscriptions. Authentication belongs to each user on each machine. Login requires the user's browser action; do not automate identity verification or copy credentials. Preserve host-key verification and unrelated services.

Use the bundled scripts/codex-autodl launcher on the remote Linux machine only when the loopback proxy workflow is configured. It checks proxy reachability and starts or reattaches a named screen session. It does not install dependencies, log in, create a local SSH tunnel or guarantee uptime. Use a distinct CODEX_SCREEN_SESSION for each project.

When a session reports an active writer, inspect matching Codex processes and screen sessions before changing anything. Prefer graceful exit; terminate only an identified stale process when authorized. Do not delete locks as a first fix.

Finish with the endpoint/configuration used (no secrets), checks that passed, checks not performed, and how to reconnect. Distinguish process persistence from network availability and authentication.
