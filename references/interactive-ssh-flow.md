# Interactive SSH flow

## Password-only SSH login

Use this when the remote machine does not use SSH keys and expects a live password prompt.

### Start

```text
exec(command="ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -o StrictHostKeyChecking=no <user>@<host>", pty=true, background=true)
```

### Continue

1. Poll until the terminal shows `password:`
2. Send `<ssh-password>\n`
3. Poll until a normal shell prompt appears

## Escalate with `su -`

```bash
su -
```

1. Poll for the password prompt
2. Send `<root-password>\n`
3. Confirm you are root with `whoami` or `id`

## Remote command pattern

Write commands into the same running session:

```bash
hostname && whoami && id && ip -brief addr && ip route
```

For long or risky work, send one logical command block at a time and poll for completion before continuing.

## When the connection may drop

These actions can break the SSH session:

- changing interface IPs
- restarting networking
- restarting sshd
- changing firewall rules
- changing default routes
- killing DHCP clients or network managers
- changing hostname or DNS in ways that affect reconnection

Before doing them:

1. Warn that reconnect may be required
2. Apply the change
3. Test reachability from the local machine
4. Reconnect with the new address, port, or host settings

## Reconnect verification checklist

After reconnecting, verify:

```bash
hostname
ip -brief addr
ip route
cat /etc/resolv.conf
whoami
```

Then run any task-specific checks such as service status, open ports, logs, or package state.

## Practical cautions

- PTY matters for password prompts, `su -`, and many interactive commands
- `StrictHostKeyChecking=no` is useful for ad hoc first-time connections where a host key prompt would stall the session
- If a previous session becomes corrupted by interrupted input, start a new SSH session rather than fighting the broken terminal state
- Avoid embedding passwords into shell scripts or command histories
- Keep one session alive for continuity, but reconnect cleanly after disruptive network changes
