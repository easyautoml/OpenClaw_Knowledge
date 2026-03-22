# CLI User Guide

## Contents

1. [How the CLI Works](#1-how-the-cli-works)
2. [Level 1 — First-Time Setup](#2-level-1--first-time-setup)
3. [Level 2 — Daily Use](#3-level-2--daily-use)
4. [Level 3 — Managing Channels](#4-level-3--managing-channels)
5. [Level 4 — Managing Models](#5-level-4--managing-models)
6. [Level 5 — Sessions and Memory](#6-level-5--sessions-and-memory)
7. [Level 6 — Gateway and Daemon](#7-level-6--gateway-and-daemon)
8. [Level 7 — Scheduling with Cron](#8-level-7--scheduling-with-cron)
9. [Level 8 — Plugins and Skills](#9-level-8--plugins-and-skills)
10. [Level 9 — Security and Approvals](#10-level-9--security-and-approvals)
11. [Level 10 — Advanced and Infrastructure](#11-level-10--advanced-and-infrastructure)
12. [Quick Reference — All Command Groups](#12-quick-reference--all-command-groups)

---

## 1. How the CLI Works

The entry point is the `openclaw` command. Every action is a subcommand:

```
openclaw <command> [subcommand] [options]
```

**Getting help at any level:**

```bash
openclaw --help                   # top-level help
openclaw channels --help          # help for the channels group
openclaw channels add --help      # help for a specific subcommand
```

**Output formats:** Most commands support `--json` for machine-readable output and work well when piped to `jq`.

**Global options** available on all commands:
```bash
--verbose      # more detailed output
--debug        # debug-level logs
--config <path>  # override the config file location
```

---

## 2. Level 1 — First-Time Setup

### onboard (interactive wizard)

The quickest way to go from zero to a running agent. Walks you through everything step by step.

```bash
openclaw onboard
```

It will ask you to:
1. Choose an LLM provider and enter credentials
2. Connect at least one messaging channel
3. Configure your first agent

Use this if you are setting up OpenClaw for the first time.

---

### configure

Set up or update any part of the configuration interactively.

```bash
openclaw configure            # full setup wizard
openclaw configure gateway    # just the gateway settings
openclaw configure channels   # just channels
```

`configure` is for changing existing settings. `onboard` is for first-time setup.

---

### doctor

Scans everything and tells you what is broken and how to fix it.

```bash
openclaw doctor
```

Checks: config validity, channel auth, model connectivity, gateway status, security, plugins. Always run this first when something is not working.

---

### health

Lightweight check that the running gateway is alive and responding.

```bash
openclaw health
openclaw health --json          # JSON output
openclaw health --verbose       # more detail
```

---

## 3. Level 2 — Daily Use

### status

The most useful command for day-to-day monitoring. Shows everything at once.

```bash
openclaw status                # full status overview
openclaw status --deep         # includes local config checks
openclaw status --all          # show all agents and channels
openclaw status --usage        # include model token usage
openclaw status --json         # JSON output
```

---

### agents

View and manage your agents.

```bash
openclaw agents list              # list all configured agents
openclaw agents list --bindings   # show channel bindings
openclaw agents list --json       # JSON

openclaw agents add               # wizard to create a new agent
openclaw agents delete --agent myagent

openclaw agents identity          # show agent identity (AGENTS.md, SOUL.md)
openclaw agents bind              # bind a channel to an agent
```

---

### sessions

View past conversation sessions.

```bash
openclaw sessions                          # sessions for the default agent
openclaw sessions --agent myagent         # sessions for a specific agent
openclaw sessions --all-agents            # sessions across all agents
openclaw sessions --json

openclaw sessions cleanup                 # remove old sessions
```

---

### logs

Tail the live gateway log in your terminal.

```bash
openclaw logs                    # follow logs (like tail -f)
openclaw logs --lines 100        # last 100 lines
openclaw logs --json             # structured JSON log lines
```

---

### tui

Open a terminal UI connected to the gateway — like a lightweight chat client in your terminal.

```bash
openclaw tui
```

Type messages and see agent responses, all in the terminal. Useful for quick testing without opening a messaging app.

---

## 4. Level 3 — Managing Channels

Channels are how users talk to your agent (Telegram, Discord, Slack, etc.). See [../04_CHANNELS.md](../04_CHANNELS.md) for concepts.

### channels list

```bash
openclaw channels list           # all configured channels + auth status
openclaw channels list --json
```

### channels status

```bash
openclaw channels status             # check gateway channel health
openclaw channels status --probe     # actively probe credentials (slower but thorough)
openclaw channels status --json
```

### channels add

**Interactive (recommended for first time):**
```bash
openclaw channels add
```
Launches a wizard. Asks which platform, walks through auth.

**Non-interactive (for scripting):**
```bash
# Telegram
openclaw channels add --channel telegram --token YOUR_BOT_TOKEN

# Slack
openclaw channels add --channel slack --bot-token xoxb-... --app-token xapp-...

# Discord
openclaw channels add --channel discord --token YOUR_BOT_TOKEN

# WhatsApp (requires pairing, so use interactive)
openclaw channels add --channel whatsapp
```

**With allowlist (only you can message the bot):**
```bash
openclaw channels add --channel telegram --token TOKEN --dm-allowlist "@yourusername"
```

**Multiple accounts:**
```bash
openclaw channels add --channel telegram --account work --token TOKEN_2
```

### channels remove

```bash
openclaw channels remove --channel telegram              # disable (wizard)
openclaw channels remove --channel telegram --delete     # delete config entirely
```

### channels login / logout

For platforms that require an active session (WhatsApp, Signal):
```bash
openclaw channels login --channel whatsapp     # link session
openclaw channels logout --channel whatsapp    # end session
```

### channels capabilities

See what a channel can do (permissions, features):
```bash
openclaw channels capabilities --channel discord
openclaw channels capabilities --channel telegram --json
```

### channels resolve

Look up a user/group ID from a username:
```bash
openclaw channels resolve "@username" --channel telegram
openclaw channels resolve "Server Name" --channel discord
```

### channels logs

See recent channel-specific log entries:
```bash
openclaw channels logs --channel telegram
openclaw channels logs --channel all --lines 500
```

---

## 5. Level 4 — Managing Models

### models list

```bash
openclaw models list                    # configured models
openclaw models list --all              # full catalog
openclaw models list --local            # only local models (Ollama, vLLM)
openclaw models list --provider openai
openclaw models list --json
```

### models status

Health-check your configured providers:
```bash
openclaw models status                        # status overview
openclaw models status --check                # verify auth credentials
openclaw models status --probe                # send a real test request
openclaw models status --probe-provider openai
openclaw models status --agent myagent        # check models used by a specific agent
```

### models set

Set the active model for an agent:
```bash
openclaw models set                           # interactive picker
openclaw models set gpt-4o --agent myagent
openclaw models set-image dall-e-3            # set the image generation model
```

### models auth

Manage API credentials per provider:
```bash
openclaw models auth add                  # wizard to add credentials
openclaw models auth login                # re-authenticate
openclaw models auth setup-token          # set an API key directly
openclaw models auth paste-token          # paste a token interactively
openclaw models auth order get            # see provider priority order
openclaw models auth order set            # set provider priority
openclaw models auth order clear          # reset priority order
```

### models aliases

Create short names for models:
```bash
openclaw models aliases list
openclaw models aliases add fast gpt-4o-mini
openclaw models aliases remove fast
```

### models fallbacks

Configure fallback chains (try model A, fall back to B if it fails):
```bash
openclaw models fallbacks list
openclaw models fallbacks add gpt-4o gpt-4o-mini   # if gpt-4o fails, try gpt-4o-mini
openclaw models fallbacks remove gpt-4o
openclaw models fallbacks clear
```

### models scan

Auto-discover available models from a provider:
```bash
openclaw models scan                      # scan all providers
openclaw models scan --provider ollama    # scan Ollama for local models
```

---

## 6. Level 5 — Sessions and Memory

### sessions

Already covered above. Key additional subcommand:

```bash
openclaw sessions cleanup                 # remove sessions older than threshold
openclaw sessions cleanup --dry-run       # preview what would be removed
openclaw sessions cleanup --days 30       # remove sessions older than 30 days
```

### memory

Manage the agent's persistent vector memory store.

```bash
openclaw memory status                         # overview of memory state
openclaw memory status --agent myagent
openclaw memory status --deep                  # detailed breakdown
openclaw memory status --index                 # show index statistics
openclaw memory status --json

openclaw memory search "query text"            # semantic search in memory
openclaw memory search "query" --agent myagent

openclaw memory rebuild                        # rebuild the vector index from transcripts
openclaw memory rebuild --agent myagent
openclaw memory rebuild --force                # rebuild even if up-to-date

openclaw memory clear                          # delete all memory for an agent
openclaw memory clear --agent myagent
```

> **Tip:** Memory is built from session transcripts. Run `memory rebuild` after importing old sessions or after a crash.

---

## 7. Level 6 — Gateway and Daemon

The **gateway** is the server process that everything connects to. You must start it before agents can receive messages.

### gateway start

```bash
openclaw gateway start                  # start in foreground (Ctrl+C to stop)
openclaw gateway start --port 18789     # custom port
openclaw gateway start --host 0.0.0.0  # bind to all interfaces
openclaw gateway start --daemon         # start as background daemon
```

### gateway stop / restart

```bash
openclaw gateway stop
openclaw gateway restart
```

### gateway status

```bash
openclaw gateway status
openclaw gateway status --json
```

### gateway reload

Reload configuration without restarting (applies most config changes live):
```bash
openclaw gateway reload
```

### daemon (legacy alias)

`daemon` is an older name for `gateway`. Commands are identical:
```bash
openclaw daemon start
openclaw daemon stop
openclaw daemon status
```

---

## 8. Level 7 — Scheduling with Cron

Schedule agents to run automatically at set times.

```bash
openclaw cron list                              # list all scheduled jobs
openclaw cron list --json

openclaw cron add                               # interactive wizard
openclaw cron add --agent myagent --schedule "0 9 * * 1-5" --message "Morning standup"

openclaw cron remove --id <job-id>
openclaw cron enable --id <job-id>
openclaw cron disable --id <job-id>

openclaw cron run --id <job-id>                # trigger a cron job immediately
```

**Schedule format:** Standard 5-field cron (`* * * * *` = minute, hour, day, month, weekday).

```
"0 9 * * 1-5"    → Every weekday at 9:00am
"0 */4 * * *"    → Every 4 hours
"30 8 * * 1"     → Every Monday at 8:30am
```

---

## 9. Level 8 — Plugins and Skills

### plugins list

```bash
openclaw plugins list                  # all loaded plugins
openclaw plugins list --json
openclaw plugins list --verbose        # include source paths and versions
```

### plugins install

Install a plugin from a path or npm package:
```bash
openclaw plugins install ./my-plugin
openclaw plugins install @myorg/my-plugin
```

### plugins enable / disable

```bash
openclaw plugins enable my-plugin
openclaw plugins disable my-plugin
```

### skills

List and inspect available skills:
```bash
openclaw skills list                   # skills for the default agent
openclaw skills list --agent myagent
openclaw skills list --json
openclaw skills show my-skill          # show a skill's SKILL.md content
```

### hooks

Manage plugin lifecycle hooks:
```bash
openclaw hooks list                    # all registered hooks
openclaw hooks list --agent myagent
openclaw hooks show before_agent_start
```

---

## 10. Level 9 — Security and Approvals

### approvals

Control which shell commands the agent is allowed to run.

```bash
openclaw approvals list                       # see all approved commands
openclaw approvals list --json

openclaw approvals add "npm test"             # approve a command pattern
openclaw approvals remove "npm test"          # revoke approval

openclaw approvals import --file approvals.json   # bulk import
openclaw approvals export > approvals.json         # backup current approvals
```

> Approvals are stored in `~/.openclaw/exec-approvals.json`. Pre-populate this file to skip per-command prompts in automated environments.

### security

Run a local security audit on your config:
```bash
openclaw security audit                # full audit
openclaw security audit --json
```

Checks: file permissions, world-writable plugin paths, suspicious token storage, config schema issues.

### secrets

Reload secret references without restarting the gateway:
```bash
openclaw secrets reload
openclaw secrets list
```

---

## 11. Level 10 — Advanced and Infrastructure

### config get / set / unset

Read and write individual config values by path:
```bash
openclaw config get agents.default.model
openclaw config get gateway.port --json

openclaw config unset agents.default.model   # remove a value (reset to default)
```

For writing, use `configure` (wizard) or edit `openclaw.json` directly.

### pairing

Manage DM pairing — approving unknown users who message the agent:
```bash
openclaw pairing list                         # pending pairing requests
openclaw pairing approve --code 123456        # approve a user
openclaw pairing reject --code 123456         # reject a user
```

### devices

Token management for mobile/device clients:
```bash
openclaw devices list
openclaw devices add
openclaw devices remove --id <device-id>
```

### nodes

Manage gateway-owned node connections (remote or mobile nodes):
```bash
openclaw nodes list
openclaw nodes status
openclaw nodes invoke <node-id> <command>
openclaw nodes canvas <node-id>              # open canvas on a node
```

### node

Run the headless node host service (for running OpenClaw on a phone or remote machine):
```bash
openclaw node start
openclaw node stop
openclaw node status
```

### sandbox

Manage isolated sandbox containers for untrusted agent code:
```bash
openclaw sandbox list
openclaw sandbox create
openclaw sandbox delete --id <sandbox-id>
```

### acp

Agent Control Protocol tools — for connecting agents to external MCP-compatible tools:
```bash
openclaw acp list
openclaw acp status
openclaw acp connect
```

### dns

DNS helpers for wide-area discovery via Tailscale and CoreDNS:
```bash
openclaw dns status
openclaw dns resolve <hostname>
```

### directory

Look up contact and group IDs across channels:
```bash
openclaw directory lookup --channel telegram
openclaw directory lookup @username --channel slack
```

### webhooks

Configure and test webhook integrations:
```bash
openclaw webhooks list
openclaw webhooks test --id <webhook-id>
```

### qr

Generate an iOS pairing QR code for mobile clients:
```bash
openclaw qr                     # print QR to terminal
openclaw qr --output qr.png     # save to file
```

### docs

Search the OpenClaw documentation from the terminal:
```bash
openclaw docs "how do I add a channel"
openclaw docs "exec approvals"
```

### update

Manage OpenClaw version updates:
```bash
openclaw update                  # check for updates
openclaw update install          # install the latest version
openclaw update channel          # see which release channel you are on
```

### completion

Generate shell tab-completion script:
```bash
openclaw completion bash >> ~/.bashrc
openclaw completion zsh >> ~/.zshrc
openclaw completion fish > ~/.config/fish/completions/openclaw.fish
```

---

## 12. Quick Reference — All Command Groups

| Group | What it does |
|---|---|
| `onboard` | First-time interactive setup wizard |
| `configure` | Update any part of the configuration |
| `doctor` | Diagnose and fix problems |
| `health` | Check the gateway is alive |
| `status` | Full system overview |
| `agents` | Create, list, delete, bind agents |
| `channels` | Connect messaging platforms (Telegram, Slack, Discord, etc.) |
| `models` | Manage LLM providers, credentials, fallbacks |
| `sessions` | View and clean up conversation history |
| `memory` | Search, rebuild, clear the vector memory store |
| `gateway` | Start/stop/manage the gateway server |
| `daemon` | Legacy alias for `gateway` |
| `logs` | Tail live gateway logs |
| `tui` | Terminal chat UI |
| `cron` | Schedule recurring agent runs |
| `plugins` | Install, enable, disable extensions |
| `skills` | List and inspect skills |
| `hooks` | View registered lifecycle hooks |
| `approvals` | Manage which shell commands the agent can run |
| `security` | Local config security audit |
| `secrets` | Runtime secret reload |
| `config` | Read/unset individual config values |
| `pairing` | Approve inbound DM pairing requests |
| `devices` | Mobile/device token management |
| `nodes` | Remote/mobile node management |
| `node` | Run the headless node host service |
| `sandbox` | Isolated sandbox containers |
| `acp` | Agent Control Protocol (MCP bridge) |
| `directory` | Look up contact/group IDs |
| `dns` | DNS helpers for Tailscale/CoreDNS |
| `webhooks` | Webhook integration tools |
| `qr` | iOS pairing QR code |
| `docs` | Search OpenClaw documentation |
| `update` | Check and install updates |
| `completion` | Generate shell tab-completion |
