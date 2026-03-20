---
name: ssh
description: Connect to remote Linux machines over SSH using password authentication, then continue working through an interactive terminal session. Use when the user provides a host/IP, username, and password for a VM, server, VPS, container host, or lab machine, especially when login requires a live password prompt, optional `su -` escalation, reconnects after network changes, or other interactive shell work that cannot be done as a single one-shot command.
---

# SSH

Use a persistent interactive SSH session when the remote system requires password entry, `su -`, TTY prompts, or configuration changes that may interrupt the connection.

## Core workflow

1. Collect the connection details:
   - host or IP
   - SSH username
   - SSH password
   - whether root login is allowed directly or whether `su -` is needed after login
2. Start an SSH session with PTY enabled.
3. Poll until the password prompt appears.
4. Send the SSH password into the live session.
5. Wait for a working shell prompt.
6. If privileged work is required, run `su -`, wait for the password prompt, then send the root password.
7. Perform the requested work in the same session when possible.
8. If the task changes networking, SSH settings, firewall rules, or host identity, expect disconnects and reconnect using the new settings.

## SSH session pattern

Start the connection as an interactive PTY session and keep it open:

```text
exec(command="ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -o StrictHostKeyChecking=no <user>@<host>", pty=true, background=true)
```

Then drive it with process control:

1. `process(action="poll", sessionId=...)`
2. When `password:` appears, send the SSH password with `process(action="write", data="<password>\n")`
3. Poll again until a shell prompt appears

## Root escalation pattern

When the user wants administrator access through `su -`:

```bash
su -
```

Then:

1. Poll for the password prompt
2. Send the root password
3. Confirm elevation with `whoami`, `id`, or the prompt change before making system-level changes

## Operating rules

- Prefer one persistent session for the whole task instead of repeatedly reconnecting.
- Use `process(action="write")` for normal commands after login.
- Use `process(action="send-keys", keys=["CTRL_C"])` only to interrupt a stuck foreground command.
- Open a fresh SSH session if the terminal state becomes garbled after interrupts or broken heredocs.
- Warn before commands that may drop the session.
- After disruptive changes, verify reachability from the local machine before reconnecting.
- Do not save passwords to files, notes, shell history, or scripts unless the user explicitly asks.
- Do not echo secrets back unless brief confirmation is genuinely necessary.

## Common post-login checks

Use only what fits the task:

```bash
hostname
whoami
id
pwd
ip -brief addr
ip route
cat /etc/os-release
systemctl --failed --no-pager
```

## Reconnect workflow

Use this when the remote session may have been broken by configuration changes:

1. Assume the old session may be dead.
2. From the local machine, test reachability with `ping` if appropriate.
3. Start a new SSH session using the new IP, hostname, port, or username.
4. Re-authenticate.
5. Re-run basic verification such as interface addresses, routes, DNS, and service state.

## When this skill is a good fit

Use it for tasks such as:

- logging into an ad hoc VM or server with username/password
- accessing lab machines without SSH keys
- using `su -` after login for root access
- changing networking and reconnecting afterward
- editing configs through a live shell
- running package installs, service restarts, logs, or system inspection on a remote Linux host

## Reference

Read `references/interactive-ssh-flow.md` for a fuller worked example of password login, `su -`, reconnect-after-IP-change, and interactive remote command handling.
