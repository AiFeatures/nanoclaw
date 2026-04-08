# NanoClaw

AI assistant that runs Claude agents securely in isolated Linux containers.

> Fork of [qwibitai/nanoclaw](https://github.com/qwibitai/nanoclaw), maintained by AiFeatures.
> Upstream sync is managed via [Ai-road-4-You/fork-sync](https://github.com/Ai-road-4-You/fork-sync).

## Why NanoClaw

NanoClaw provides the same core functionality as [OpenClaw](https://github.com/openclaw/ a personal AI assistant connected to messaging  but in a codebase small enough to audit and understand. Agents run in their own Linux containers with filesystem isolation, not merely behind application-level permission checks. One process, a handful of source files, and no microservices.channels openclaw) 

## Status

| Field | Value |
|-------|-------|
| Version | 1.2.25 |
| Runtime | Node.js 20+ |
| Stability | Active development |
| Upstream | [qwibitai/nanoclaw](https://github.com/qwibitai/nanoclaw) |

## Quick Start

```bash
git clone https://github.com/AiFeatures/nanoclaw.git
cd nanoclaw
npm install
npm run build
```

To configure channels and services interactively, open [Claude Code](https://claude.ai/download) and run the `/setup` skill:

```bash
claude
# inside the Claude Code prompt:
/setup
```

> Commands prefixed with `/` (like `/setup`, `/add-whatsapp`) are [Claude Code skills](https://code.claude.com/docs/en/skills). Type them inside the `claude` CLI prompt, not in your regular terminal.

## Features

- **Multi-channel  WhatsApp, Telegram, Discord, Slack, Gmail. Add channels via skills (`/add-whatsapp`, `/add-telegram`, etc.).messaging** 
- **Container  Agents run in Docker, [Docker Sandboxes](docs/docker-sandboxes.md) (micro-VM), or [Apple Container](https://github.com/apple/container) (macOS).isolation** 
- **Per-group  Each group has its own `CLAUDE.md`, isolated filesystem, and container sandbox.memory** 
- **Scheduled  Recurring jobs that run Claude and message you back.tasks** 
- **Agent  Teams of specialized agents that collaborate on complex tasks.Swarms** 

## Architecture

```
        Container (        ResponseClaude Agent SDK)         Polling loop         SQLite Channels 
```

Single Node.js process. Channels self-register at  the orchestrator connects whichever ones have credentials present. Agents execute in isolated Linux containers with only mounted directories accessible. Per-group message queue with concurrency control. IPC via filesystem.startup 

Key files:

| Path | Purpose |
|------|---------|
| `src/index.ts` | Orchestrator: state, message loop, agent invocation |
| `src/channels/registry.ts` | Channel registry (self-registration) |
| `src/container-runner.ts` | Spawns streaming agent containers |
| `src/router.ts` | Message formatting and outbound routing |
| `src/db.ts` | SQLite operations |
| `src/task-scheduler.ts` | Scheduled task runner |
| `groups/*/CLAUDE.md` | Per-group memory (isolated) |

**Container build target:** The agent container image is defined at `container/Dockerfile` and built with `./container/build.sh`. See [docs/SPEC.md](docs/SPEC.md) for the full architecture.

## Development

### Prerequisites

- macOS or Linux
- Node.js 20+
- [Docker](https://docker.com/products/docker-desktop) or [Apple Container](https://github.com/apple/container) (macOS)

### Commands

```bash
npm run build          # Compile TypeScript
npm run dev            # Run with hot reload (tsx)
npm run typecheck      # Type-check without emitting
npm run lint           # ESLint
npm run lint:fix       # ESLint with auto-fix
npm run format:check   # Prettier check
npm run format         # Prettier write
npm test               # Run tests (vitest)
npm run test:watch     # Run tests in watch mode
```

### Container

```bash
./container/build.sh   # Build the agent container image
```

## Deployment

CI is provided by the repository workflows in `.github/workflows/`. For enterprise deployments, reference the shared reusable workflows:

```yaml
uses: Ai-road-4-You/enterprise-ci-cd/.github/workflows/ci-node.yml@v1
```

```yaml
uses: Ai-road-4-You/enterprise-ci-cd/.github/workflows/release.yml@v1
with:
  tags-override: latest
```

### Service Management

```bash
# macOS (launchd)
launchctl load ~/Library/LaunchAgents/com.nanoclaw.plist
launchctl kickstart -k gui/$(id -u)/com.nanoclaw    # restart

# Linux (systemd)
systemctl --user start nanoclaw
systemctl --user restart nanoclaw
```

## Contributing

See [Ai-road-4-You/governance](https://github.com/Ai-road-4-You/governance) for contribution guidelines and standards.

## License

[MIT](LICENSE)
