# AutoDL Codex

Codex skills for connecting to AutoDL GPU instances, setting up OpenAI Codex CLI, and recovering remote terminal sessions. Maintained by **sg-audex**.

[中文说明](../README.md) · [Download](https://github.com/Yanyadua/autodl-codex/releases/latest) · [Issues](https://github.com/Yanyadua/autodl-codex/issues)

## Problems covered

- AutoDL SSH banner timeouts and Fake-IP DNS routing.
- Codex CLI network failures on a remote GPU instance.
- Optional loopback HTTP proxy forwarding over SSH.
- screen session persistence, Codex resume, and stale writer diagnosis.

## Install

Paste this into Codex:

```text
Use $skill-installer to install skills/autodl-ssh and skills/autodl-codex from https://github.com/Yanyadua/autodl-codex.
```

Alternatively copy both complete skill directories into `~/.agents/skills/`. Restart Codex if they do not appear.

For supported plugin clients, add the marketplace and install AutoDL Codex from the plugin interface:

```bash
codex plugin marketplace add Yanyadua/autodl-codex
```

## Try it

“Help me connect to my AutoDL GPU instance and set up Codex CLI.”

“Diagnose my SSH banner timeout on AutoDL.”

“Recover my Codex session after an SSH disconnect.”

## Requirements and limits

You need your own AutoDL instance, authorized SSH access and Codex authentication. Local terminal execution is required. Experience is based on macOS, FlClash and AutoDL Linux; other environments need adaptation. screen keeps the process alive across SSH disconnects, but a local proxy going offline or an instance shutdown still interrupts operation.

22 local checks were performed for v1.0.0, including mocked command behavior; no live instance or long-running uptime test was performed for that release. See the [validation report](validation.md). This is an independent community project, without bundled credentials, telemetry or an MCP service. It is not listed in the official OpenAI plugin directory. No additional open-source license has been granted.
