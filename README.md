# sodp-skills

Claude Code slash commands for integrating the [SODP](https://github.com/orkestri/SODP) (State-Oriented Data Protocol) client into any project.

SODP replaces polling and request/response with a `WATCH → STATE_INIT → DELTA` stream over WebSocket. Clients subscribe once and receive only the changed fields on every mutation.

## Install

Copy the commands into your project's Claude commands directory:

```bash
# From your project root
mkdir -p .claude/commands
BASE=https://raw.githubusercontent.com/orkestri/sodp-skills/main/.claude/commands
for cmd in sodp-setup sodp-watch sodp-set sodp-presence \
           sodp-server-setup sodp-server-go sodp-server-acl sodp-server-schema; do
  curl -fsSL "$BASE/$cmd.md" -o ".claude/commands/$cmd.md"
done
```

Or clone and copy:

```bash
git clone --depth=1 https://github.com/orkestri/sodp-skills /tmp/sodp-skills
cp /tmp/sodp-skills/.claude/commands/*.md .claude/commands/
```

## Commands

### Client

| Command | What it does |
|---------|-------------|
| `/sodp-setup` | Detects your language, installs the dependency, creates a configured client |
| `/sodp-watch [key]` | Adds a watch callback for a state key |
| `/sodp-set [key] [op]` | Adds set, patch, or set_in write code |
| `/sodp-presence [key]` | Adds live presence registration + watcher (auto-removed on disconnect) |

### Server

| Command | What it does |
|---------|-------------|
| `/sodp-server-setup` | Gets the server running — Docker, binary, or from source (Rust or Go) |
| `/sodp-server-go` | Embeds the Go server into an existing Go HTTP service |
| `/sodp-server-acl` | Generates ACL rules to protect state keys per user/role |
| `/sodp-server-schema` | Generates schema validation config for state keys |

## Supported languages

### Client SDKs
- TypeScript / JavaScript (`@sodp/client`)
- React (`@sodp/react`)
- Python (`sodp-client`)
- Java (`site.orkestri:sodp-client`)

### Server
- Rust (standalone — highest throughput)
- Go (standalone or embedded library)

## Example

```
/sodp-setup
```
Claude detects your project language, installs the package, and creates a working client file.

```
/sodp-watch game.score
```
Claude adds a typed watch callback for `game.score` in the right place in your code.

```
/sodp-presence collab.cursors
```
Claude adds live cursor presence that auto-cleans when users disconnect.

## SODP server

Run one locally for development:

```bash
# Docker
docker run -p 7777:7777 ghcr.io/orkestri/sodp-server:latest

# Or build from source
git clone https://github.com/orkestri/SODP && cd SODP
cargo run --bin sodp-server -- 0.0.0.0:7777
```

## Links

- [Protocol spec & server](https://github.com/orkestri/SODP)
- [TypeScript client](https://www.npmjs.com/package/@sodp/client)
- [React hooks](https://www.npmjs.com/package/@sodp/react)
- [Python client](https://pypi.org/project/sodp-client/)
- [Java client](https://central.sonatype.com/artifact/site.orkestri/sodp-client)
