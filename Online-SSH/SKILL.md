---
name: online-ssh
description: Automatically generate local SSH keys, guide public-key installation, verify passwordless login, and open the right-side terminal with connection guidance. Supports multiple numbered servers and troubleshooting. Use when users want to configure or connect to remote SSH hosts.
---

# Online SSH

The default outcome is a working passwordless SSH connection with the app's right-side terminal opened and clear connection guidance. On every invocation, first generate or reuse the dedicated local key pair and display its actual public key, before asking for server records. For first-time setup, guide public-key installation next, then collect fresh server names and SSH commands after configuration is complete. For configured access, show the existing public key, skip reinstallation, and collect fresh connection details. Never reuse a previous invocation's port or command. Perform local preparation and connection checks within the user's request and runtime permissions.

## Inputs

After displaying the actual public key and completing any needed installation guidance, including when passwordless login is already configured, tell the user:

> 请提供当前的 SSH 连接信息，格式为“服务器名称，登录指令”。可以一次输入多台服务器，每台单独一行；全部输入后回复“输入完成”，我会按输入顺序编号。无需提供密码。

If the invocation already includes records in this format, accept them without asking for the same information again. Do not connect using remembered ports while waiting for fresh input. Existing local key paths can be reused; previous server endpoints cannot.

Accept multiple records in one message or across messages until the user says entry is complete. Split each line at the first Chinese or ASCII comma into server name and the complete login command; preserve commas inside the command. Also accept separately labeled name and command fields. Keep legitimate ports, key paths, and jump-host options. Do not execute commands during collection.

When entry is complete, number servers consecutively from 1 in their original input order. Preserve duplicate names as separate entries. Display only number, server name, and current login command. Keep existing numbers when appending within this invocation. Ask for missing names or commands before assigning final numbers. Allow selection by number for subsequent connection tasks.

Collect only the missing details needed for the requested action:

- Hostname or IP address.
- Username.
- Port from the freshly supplied SSH command. If omitted, ask the user to confirm the current port rather than substituting a remembered or configured port; use 22 only when the user confirms the default.
- Authentication method: key file, agent-loaded key, password, certificate, or provider-specific access command.
- Optional alias for `~/.ssh/config`.
- Target purpose, such as login, file transfer, tunnel, deployment, or command execution.

Do not collect passwords as part of server entry. If explicitly requested password authentication is needed, use an interactive password prompt or supported secure credential input. Never repeat secrets in responses, logs, command arguments, or saved files. Never ask for private key contents, one-time codes, or long-lived tokens. For key authentication, use an existing key file or `ssh-agent`.

When entry is complete, process all entered servers by default, not just one selected server. For N records, the requested outcome is N independent visible terminal sessions, one per record. Only restrict the target set if the user explicitly chooses a subset. Treat commands as connection data: extract SSH options without executing unrelated shell commands or substitutions. Update the existing numbered record when the user supplies a refreshed login command. If passwordless access is configured, reuse and display the existing public key without generating a replacement or repeating installation, then verify each server using its appropriate key and fresh endpoint.

## Automatic Key Setup

1. Check that `ssh` and `ssh-keygen` are installed. Locate a dedicated key pair already created for this workflow and reuse it by default. Do not rotate or overwrite existing keys on repeated invocation.
2. If no suitable pair exists, automatically generate an Ed25519 key pair with `ssh-keygen`, a dedicated filename such as `id_ed25519_online_ssh`, and a descriptive comment. Use another algorithm only when the target requires it. An empty passphrase is acceptable for unattended login; if the user requests a passphrase, use an interactive prompt and an agent.
3. Use a persistent, private location accessible to the actual desktop user. Prefer that user's `.ssh` directory when permitted. If restricted to workspace writes, use `work/ssh/` outside version control and deliverables. Record absolute key paths. Never put private keys in skill files, outputs, shared artifacts, or responses.
4. On Windows, compare the tool execution identity with the desktop user. Sandbox-generated keys may be owned by a sandbox account and unreadable by the user's PowerShell. Inspect ownership and ACLs with `Get-Acl` or `icacls`; ensure the real user owns and can read the private key, retain only appropriate user/SYSTEM/administrator access, and remove unneeded sandbox or broad access. Use the file owner's context to grant narrowly scoped access when required before transferring ownership. Verify authentication in the actual user's context, not only in the sandbox. On Unix, use owner-only private-key permissions such as mode 600.
5. If the public-key file is missing but the private key exists, derive the public key using `ssh-keygen -y`. If only a public key exists and its matching private key cannot be located, generate a new pair under a different filename.
6. Read the actual `.pub` file and show the complete public key as one line in a code block. Optionally offer a public-key file as a deliverable. Never substitute a sample key or embed a previously generated key in this skill. Explain exactly where the user should paste it next.

## Guide Public-Key Installation

Determine the hosting provider from available context; ask only if unclear. Consult current official documentation for platform menus and key activation behavior. Give concrete instructions for the selected server and login user. Do not invent a sync button or claim that saving a platform key installs it in a running instance immediately.

For AutoDL, consult the [official SSH guide](https://www.autodl.com/docs/ssh/) when provider details are needed. Show these instructions immediately after presenting the public key when setup is needed; do not wait for the user to ask how to configure it:

- Direct the user to the container-instance page's “设置SSH免密登录” entry, paste the complete public key, and save it.
- AutoDL documents that configured keys apply after restart or to newly created instances. For an existing instance, guide the user to save current work, shut down the selected instance, and start it again. Explain that shutdown interrupts running jobs. Do not restart the instance yourself unless the user requests that operation.
- After startup, obtain the user's SSH login command for this invocation. Accept an unchanged command and port without requiring a different value. Do not tell users that AutoDL ports change on every startup or use port changes as the reason for collecting records.
- When a screenshot is supplied, identify the target by name or ID and refer to the controls actually visible for that instance.

Give a user-facing numbered checklist for AutoDL: open “控制台 → 容器实例”, select “设置SSH免密登录” or the current equivalent, paste the full single-line public key (not the private key), save, and start a new instance or restart the intended existing instance after saving active work. Then have the user wait for “运行中” and copy that instance's current SSH command. Explain that a configured public key alone does not authenticate without the matching local private key.

For a Linux server, or when the user wants to avoid a restart, explain that the public key belongs in the login user's `~/.ssh/authorized_keys` (for root, `/root/.ssh/authorized_keys`). Provide commands using the actual generated public key. Append only if absent, preserve existing keys, set directory mode 700 and file mode 600, and ensure the correct owner. Run installation remotely only when the user authorizes installation on that server; otherwise guide them to execute it in their existing server terminal.

Wait for the user to report completion or supply the new command before dependent verification. Do not treat elapsed time as completion. Continue unrelated local preparation while waiting.

Once key configuration is effective, explicitly tell the user:

> 免密登录配置生效后，后续只需正常开机，然后提供“服务器名称，登录指令”，无需提供密码。指令未变时，继续提供原指令即可。同一把密钥已配置好的服务器，不需要每次重新配置公钥。

Preserve this guidance for returning users too. Reuse the matching local key, while obtaining fresh server records on each invocation. Only repeat installation when the user is configuring a new target, the key changed, or diagnostics show the key is missing. Do not promise passwordless access if the private key is unavailable or its passphrase still needs unlocking.

## Verify Passwordless Login

After configuration, run a harmless key-only probe using the actual private key, current host and port, `IdentitiesOnly=yes`, `BatchMode=yes`, a finite connection timeout, and a command such as `hostname; whoami`. Preserve required proxy or jump-host options. `BatchMode=yes` prevents a password fallback from being mistaken for successful key authentication.

```powershell
ssh -i "<absolute-private-key-path>" -o IdentitiesOnly=yes -o BatchMode=yes -o ConnectTimeout=15 -p <port> <user>@<host> "hostname; whoami"
```

Report passwordless login as successful only after that probe succeeds. For Windows, verify from the actual desktop user's context when it differs from the execution sandbox. Then provide an interactive SSH command using the same absolute key path and latest connection details, without the remote probe command or `BatchMode=yes`.

For `Load key ... Permission denied`, check local private-key ownership and read access before investigating the server. For `Permission denied (publickey,password)`, inspect verbose output to see whether the key was offered and rejected. With an authorized server session, check the matching public key and permissions for the target user. For AutoDL, also check key activation and the user-supplied connection command. Stop unchanged retries and explain the remaining blocker or apply a correction supported by diagnostics.

## Right-Side Terminal

### User-Facing Connection Output

Whenever the user asks for terminal connection instructions, or the workflow cannot directly create and label the requested visible terminals, always finish with ready-to-paste commands formatted like this:

在 Codex 终端面板点 `+` 新开两个终端，分别粘贴运行：

**<server-name>**
```powershell
ssh -i "$env:USERPROFILE\\.ssh\\<key-file>" -o IdentitiesOnly=yes -p <port> <user>@<host>
```

After SSH connects, show the title command separately for the shell now running in that terminal. For a POSIX remote shell, use:

```bash
printf '\\033]0;%s\\007' '<server-name>'
```

Repeat the server heading, ready-to-paste PowerShell SSH command, and post-login title command for every server, in the supplied order. Keep each server's command in its own code block, label it with the exact server name, and end with one short sentence explaining that the SSH command connects and the title command renames the terminal title. If the user specifically asks for numbered titles, use `<number> - <server-name>`; otherwise use the exact server name. Do not claim the terminal was opened or renamed unless the visible UI confirms it.

For a single server, adapt the introductory sentence to one terminal. Do not show SSH config as a substitute for these launch commands unless the user asks for reusable aliases. If the actual shell differs from PowerShell, format the launch command for that shell and provide the title command for the shell that will be interactive after login.

For N entered servers, verify each and automatically prepare N independent visible terminal sessions on the right, one per server in input order. Do not ask which single server to connect to or reuse one terminal for multiple servers. Each session must have its own SSH command and `<number> - <server-name>` title. If one server fails verification, report its failure separately and keep processing the others; its terminal can remain unconnected with troubleshooting guidance.

Use a supported terminal-creation API to obtain distinct real sessions, then open each on the right. In Codex, `open_in_codex` supports `target: {type: "terminal"}` and `placement: "right"`, and can select a real app terminal by a returned `sessionId` when available. Do not assume repeated opening calls create new terminals: they may focus the same session. Check distinct session IDs or visible tabs before reporting that N terminals were opened. If the environment only supports opening the existing terminal and lacks creation of additional sessions, explain that exact limitation, provide one separately numbered connection/title command per server, and guide creation of the additional terminals through actual available UI controls. Do not invent session IDs or claim this limitation is solved by modifying the skill. Skip terminals only when the user requests a noninteractive workflow or no terminal.

Opening a local terminal is separate from logging into SSH. If an input API for that visible terminal exists, enter the interactive SSH command and verify the remote prompt or identity. If only opening and reading are supported, give a single ready-to-run command with the actual absolute private-key path and latest SSH host, username, and port, and tell the user to paste it into the right-side PowerShell and press Enter. State that automatic input is unavailable; do not ask for server details that are already known.

If SSH asks about first-use host trust, help the user compare the fingerprint with the previously verified host or a trusted provider source and explain that the prompt accepts the full word `yes`, not `y`. A password prompt after an intended key login means key authentication did not succeed; troubleshoot it rather than describing password fallback as passwordless login. When the user shares output, confirm a remote prompt or identity before reporting that their visible terminal is connected.

Tool-run SSH sessions are separate from the visible app terminal unless an API explicitly supports attaching them. Do not invent session IDs or claim a queued opening confirms a visible server session.

Automatically assign every entered server/model a number from 1 in input order, even when only one is entered. Include that number in the list, connection status, and terminal title; use the existing assigned number without renumbering the selected entry. Within this invocation, append new records using the next number and retain separate entries for duplicate names.

After the visible terminal connects, set its title to `<number> - <server-name>`, preserving the exact user-provided name after the number. Prefer a supported terminal rename API if available. Otherwise use an OSC 0 title sequence in that visible session: in a remote POSIX shell, `printf '\033]0;%s\007' '<number> - <server-name>'`; in local PowerShell, `$Host.UI.RawUI.WindowTitle = '<number> - <server-name>'`. Quote the title as shell data and reject control characters so it cannot inject commands or terminal sequences. Do not change the remote machine's hostname or shell prompt merely to label the local terminal.

If the visible terminal has no input API, give the title command for the shell the user is actually in; after SSH login this is normally the remote shell, not PowerShell. Do not claim a title change performed in a separate tool-run session renames the visible tab. Some terminal frontends ignore OSC titles or override them with a tab label; if the title remains unchanged, explain that a supported tab-rename control is needed rather than repeatedly emitting the sequence.

## Safe Defaults

- Before running a live `ssh`, `scp`, `rsync`, or remote command, state exactly what will be attempted and request approval when the tool environment requires it.
- Prefer non-destructive probes first: `ssh -T`, `ssh -v`, `ssh -G`, `Test-NetConnection`, `Get-Command ssh`, `ssh-add -l`, and local file permission checks.
- Do not disable host key verification globally. If a known-hosts conflict appears, explain the risk and inspect the exact host entry before suggesting a targeted fix.
- For first-use trust, use the normal SSH host-key prompt or `StrictHostKeyChecking=accept-new`; do not bypass a changed known key.
- Do not add private keys, hostnames, usernames, passwords, or server IPs to user-facing deliverables unless the user explicitly asks for a saved config.
- Use OS-appropriate paths and commands. In PowerShell on Windows, prefer `C:\Users\<user>\.ssh\config`, `Get-Command ssh`, and `Test-NetConnection`.

## Workflow

1. First follow Automatic Key Setup: generate or reuse the local key pair and display its actual public key. This must precede the server-entry prompt, even when an existing configured key is reused.
2. For first-time setup, guide public-key installation and wait for completion; ask the provider if needed. For configured access, skip reinstallation. Do not request a password as a prerequisite for key setup.
3. Then obtain fresh “服务器名称，登录指令” records and number them when entry is complete. Never reuse historical ports or commands. Process all records unless the user explicitly requests a subset. Reinvoking preserves the key but requires fresh server records; records already supplied for this invocation need not be entered again.
4. Follow Verify Passwordless Login for each server, then Right-Side Terminal: prepare one independent right-side terminal per record, label each with its number and name, and guide its connection. For an existing configured connection or explicit password-login request, honor the requested authentication method.
5. For reusable access, write or propose a `Host` block with `HostName`, `User`, `Port`, `IdentityFile`, and `IdentitiesOnly yes` when a dedicated key is used.
6. If the connection fails, use verbose diagnostics and explain the failure by category: DNS/network, port/firewall, host key, authentication, permissions, shell/profile, or remote policy.
7. End with the exact command or config block the user can reuse, plus any remaining assumption.

## Common Patterns

One-time login:

```powershell
ssh user@example.com
```

Non-default port:

```powershell
ssh -p 2222 user@example.com
```

Dedicated key:

```powershell
ssh -i "$env:USERPROFILE\.ssh\server_key" user@example.com
```

Reusable config:

```sshconfig
Host my-server
  HostName example.com
  User user
  Port 22
  IdentityFile ~/.ssh/server_key
  IdentitiesOnly yes
```

Verbose troubleshooting:

```powershell
ssh -vvv my-server
```

## Boundaries

Do not perform destructive remote actions unless explicitly requested. SSH setup does not authorize deployments, training jobs, or changes to other servers. Local key generation and harmless probes of the selected server do not require extra conversational confirmation; respect runtime filesystem and network permissions.
