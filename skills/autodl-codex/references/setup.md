# AutoDL Codex CLI setup

Use this workflow when the user wants to run Codex CLI inside an AutoDL container while the Mac/desktop has FlClash available. Treat the host, tunnel, and remote container as separate layers and verify them in that order.

## Non-negotiable safety and configuration rules

- Never print, paste, copy, or commit `~/.codex/auth.json`, access tokens, refresh tokens, API keys, or proxy subscription data. Treat `auth.json` as a password.
- The working proxy arrangement is a static reverse HTTP/mixed-port forward: local FlClash `127.0.0.1:7890` -> SSH reverse tunnel -> AutoDL `127.0.0.1:7891`. Do not replace it with remote dynamic SOCKS forwarding; AutoDL may reject that channel.
- AutoDL hostnames may resolve through a `198.18.x.x` Fake-IP/TUN address. If SSH is unstable, resolve the current real endpoint from AutoDL and use the real IP with `HostKeyAlias='[HOST]:PORT'`; retain strict host-key checking and `IdentitiesOnly=yes`.
- SSH passwords are entered at the interactive `password:` prompt. Never append a password after `root@host`—anything after the host is parsed as a remote command, and a pasted password must be rotated immediately.
- Do not delete a Codex writer-lock file as the first fix. Find the actual process or `screen` session holding it, stop only that stale Codex process, then retry.
- `codex resume --last` is correct. `codex resume last` asks for a session whose literal ID is `last` and is incorrect.

## Parameters

Collect the current host, SSH port, user, authorized key path, remote work directory and actual local HTTP/mixed proxy port. Use the user's current configuration; there are no embedded server defaults. Set AUTODL_REAL_IP only after resolving the current endpoint. Suggested proxy ports are local 7890 and remote 7891; check availability first. Before commands, assign and validate AUTODL_HOST, AUTODL_REAL_IP, AUTODL_PORT, REMOTE_USER, REMOTE_WORKDIR, LOCAL_FLCLASH_PORT and REMOTE_PROXY_PORT in the appropriate shell. Remote shells do not automatically inherit local variables; define REMOTE_PROXY_PORT and REMOTE_WORKDIR there too. For unattended autossh, supply the authorized key using -i or a verified SSH alias.

## New AutoDL instance procedure

Treat every new container as a new remote machine. Do not reuse the old instance's host, port, real IP, process IDs, writer locks, or cached login state without checking them.

1. In the AutoDL console, start the instance and copy its current SSH host, port, username, and authentication method. Update the parameters above. If the hostname resolves to `198.18.x.x`, obtain the real endpoint from the AutoDL connection information or a successful SSH diagnostic and set `AUTODL_REAL_IP` accordingly.
2. From the Mac/desktop, run the local FlClash test, then start a new `autossh` reverse tunnel using the new host/IP/port. The remote forwarded port may remain `7891` because it is inside the new container, but first confirm it is free.
3. On the new instance, run the remote `7891` proxy test. Only after it passes should you install Codex and set the four proxy environment variables.
4. Run `codex login status`. A new instance normally needs a fresh `codex login --device-auth`; never copy the old machine's `~/.codex/auth.json`. If using a persistent disk, it is acceptable to retain the Codex directory only if the user explicitly wants the existing local state and its permissions are protected; still do not expose its contents.
5. Create a fresh `screen -S codex` session and start Codex from the project directory. An old session ID is not expected to exist on a fresh non-persistent container.

Projects and experiment outputs may persist under AutoDL's mounted persistent path (for example `~/autodl-tmp`) while Codex history normally lives under `~/.codex/sessions`. Confirm both separately. If the old history is unavailable, continue from the project files, result JSON, logs, and Git history rather than inventing a missing transcript.

## 1. Verify the local FlClash layer

On the Mac/desktop, check that FlClash is running and that its mixed/HTTP port is actually `7890` (the port can differ by profile). Test without exposing headers or credentials:

```bash
curl --noproxy '' -x "http://127.0.0.1:$LOCAL_FLCLASH_PORT" -I --connect-timeout 5 --max-time 20 https://chatgpt.com
```

`HTTP/1.1 200 Connection established` followed by an HTTP response such as `403` proves the proxy path and TLS handshake reached the site; a 403 alone is not a proxy failure. A connection timeout means fix FlClash/profile/routing before touching AutoDL.

## 2. Establish reliable SSH and the reverse tunnel

Use the current real IP if the hostname resolves to Fake-IP. For an interactive SSH connection:

```bash
ssh \
  -o Hostname="$AUTODL_REAL_IP" \
  -o HostKeyAlias="[$AUTODL_HOST]:$AUTODL_PORT" \
  -o StrictHostKeyChecking=yes \
  -o IdentitiesOnly=yes \
  -o ServerAliveInterval=20 \
  -o ServerAliveCountMax=3 \
  -p "$AUTODL_PORT" "$REMOTE_USER@$AUTODL_HOST"
```

From the Mac/desktop, keep the proxy tunnel alive independently of the interactive SSH shell. Prefer `autossh`:

```bash
env AUTOSSH_GATETIME=0 autossh -M 0 -f -N -T \
  -o Hostname="$AUTODL_REAL_IP" \
  -o HostKeyAlias="[$AUTODL_HOST]:$AUTODL_PORT" \
  -o StrictHostKeyChecking=yes \
  -o IdentitiesOnly=yes \
  -o BatchMode=yes \
  -o ServerAliveInterval=20 \
  -o ServerAliveCountMax=3 \
  -o TCPKeepAlive=yes \
  -o ExitOnForwardFailure=yes \
  -R 127.0.0.1:$REMOTE_PROXY_PORT:127.0.0.1:$LOCAL_FLCLASH_PORT \
  -p "$AUTODL_PORT" "$REMOTE_USER@$AUTODL_HOST"
```

If `autossh` is unavailable on macOS, install it with the user's package manager (for example `brew install autossh`) and do not substitute a SOCKS reverse forward. A single `ssh -N -T ... -R ...` is acceptable for diagnosis but will not automatically recover.

On AutoDL, verify the tunnel before starting Codex:

```bash
curl --noproxy '' -x "http://127.0.0.1:$REMOTE_PROXY_PORT" -I --connect-timeout 8 --max-time 20 https://chatgpt.com
```

If this says `Connection refused`, the local FlClash port may still be healthy while the reverse tunnel is down. Restart `autossh` on the Mac/desktop and retest. If it connects but times out during TLS, test the local proxy again and inspect FlClash's selected node/mode.

## 3. Install and configure Codex on AutoDL

On AutoDL, export proxy variables in the same shell that starts Codex:

```bash
export PATH="$HOME/.local/bin:$PATH"
export HTTP_PROXY="http://127.0.0.1:$REMOTE_PROXY_PORT"
export HTTPS_PROXY="$HTTP_PROXY"
export http_proxy="$HTTP_PROXY"
export https_proxy="$HTTPS_PROXY"
```

Install or update Codex only after the proxy test succeeds. If npm is already available, the official package is `npm install -g @openai/codex@latest`. Check current official installation guidance and Node requirements before installing dependencies; do not assume npm or Node exists. Verify with:

```bash
codex --version
```

If the installer endpoint is unavailable, use the official installer with the proxy; for release-download failures, use the installer's documented GitHub fallback rather than downloading an untrusted binary or changing the API base URL blindly.

## 4. Authenticate without sharing credentials

Check first:

```bash
codex login status
```

When the cached ChatGPT refresh token is invalid or already used, run a fresh login on AutoDL:

```bash
codex logout
codex login --device-auth
```

Complete only the newly generated device URL/code in a browser using the same working network path. Do not reuse an old authorization page and do not copy the Mac's `~/.codex/auth.json` to AutoDL; doing so can cause refresh-token reuse and invalidate the session.

If the browser shows `Route Error (400 Invalid content type: text/html; charset=UTF-8)`, close the old page, generate a new device code, and retry in a clean/incognito browser window. Check FlClash global routing if the callback is being rewritten. The error is in the authorization callback, not a missing Linux package.

## 5. Keep the Codex process alive across SSH disconnects

AutoDL may not have `tmux`; `screen` is a reliable fallback. Start Codex inside a named screen session, after exporting the proxy variables:

```bash
screen -S codex -R
cd "$REMOTE_WORKDIR"
codex resume --last
```

Detach with `Ctrl-a`, then `d`. Reconnect later with:

```bash
screen -r codex
```

The remote `screen` protects the Codex process from an interactive SSH disconnect, while local `autossh` protects the proxy tunnel. Both layers must remain available for continued network requests.

For repeat use, copy the bundled `../scripts/codex-autodl` to a user-owned executable directory such as `~/.local/bin/codex-autodl`, chmod it executable, and ensure that directory is on PATH. No privileged installation is needed. It exports the proxy variables, checks the remote proxy, creates or reattaches the `codex` screen session, uses the explicit CODEX_WORKDIR (or the current directory), and starts Codex:

```bash
codex-autodl
codex-autodl login --device-auth
codex-autodl resume --last
```

The launcher accepts `CODEX_WORKDIR`, `CODEX_PROXY_PORT`, and `CODEX_SCREEN_SESSION` overrides. It never performs login automatically and never stores credentials. For an existing screen session, arguments apply only when creating a new session; to run a new login command, use the current remote shell or a distinct session name. Existing sessions retain their original environment and working directory. Keep the local `autossh` tunnel running separately.

The launcher defaults to Codex's normal alternate-screen TUI because `--no-alt-screen` can leave each live redraw visible as a snapshot in terminal scrollback. For persistent scrollback, pass `--no-alt-screen` explicitly if supported by the installed Codex version.

## 6. Resume old sessions and diagnose stale writers

Use these forms:

```bash
codex resume --last       # newest interactive session in the current directory
codex resume --all        # picker including sessions from other directories
codex resume SESSION_ID   # exact UUID or session name
```

If `codex resume --last` reports `already has an active writer (code -32600)`, inspect processes and screen sessions:

```bash
ps -eo pid,ppid,stat,etime,args | grep -E '(^|/)(codex|codex-cli)( |$)' | grep -v grep
screen -ls
```

Exit the old Codex UI normally when possible. If it is stuck, terminate only the identified stale Codex PID, wait briefly, and retry the resume command. Do not kill unrelated Jupyter, training, proxy, or supervisor processes. A leftover lock file may remain after the process exits; only remove one after confirming no Codex process owns the session and only if the current Codex version still refuses to resume.

## 7. Interpret common errors

- `codex resume last`: syntax error in the command; use `--last`.
- `Connection refused` on `127.0.0.1:7891`: reverse tunnel absent, not necessarily FlClash failure.
- `Request timed out`, `stream disconnected`, or `Connection refused` from `chatgpt.com/backend-api/codex/responses`: first test remote `7891`, then restart the tunnel and restart the stuck Codex process.
- `MCP client for codex_apps timed out`: an optional desktop/app MCP is unavailable in the remote CLI. If no MCP server is needed, ignore it or disable the configured `mcp_servers` entry; increasing `startup_timeout_sec` only delays startup and does not repair a dead proxy.
- `;1R...rgb...` pasted into Bash: terminal escape/control characters. Run `reset` (or `stty sane`) and retype the command manually.

After repair, verify in this order: local FlClash `7890` -> AutoDL `7891` -> `codex login status` -> a fresh Codex request inside `screen`. Do not infer network health from an old Codex process that is still displaying a reconnect loop.

## Scope and verification limits

Use an existing direct, working network path when available; do not force a proxy. The helper launcher is for the reverse-proxy workflow only. A reverse tunnel and screen do not survive a powered-off local proxy machine or terminated GPU instance. Screen keeps the process alive across SSH disconnects, not across instance shutdown. Before reattaching, inspect screen -ls; do not detach another active user with -D automatically. For direct access, launch Codex in screen without the helper.

A proxied HTTP response establishes reachability only, not successful Codex authentication, API access or streaming. Finish with a real small Codex request. Read only process metadata during diagnosis; redact secrets if process arguments contain them. Never disable approvals or sandboxing to make the setup work. Follow current service access requirements rather than presenting networking as a way to bypass restrictions.
