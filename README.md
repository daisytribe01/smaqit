# smaqit

Spec-driven agent orchestration kit.

## What is it?

A framework where AI agents write specifications, then implement from those specs. No code without specs first.

## Layers

1. **Business** — Use cases, actors, goals
2. **Functional** — Behaviors, contracts, data models
3. **Stack** — Languages, frameworks, tools
4. **Infrastructure** — Compute, networking, observability
5. **Coverage** — Tests against deployed app

## Phases

1. **Develop** — Write specs (business → functional → stack), then build
2. **Deploy** — Write infrastructure spec, then deploy
3. **Validate** — Write coverage spec, then test

## Getting Started

```bash
# Install
go install github.com/smaqit/smaqit@latest

# Initialize in your project
smaqit init

# Start developing
smaqit develop
```

## Commands

| Command | Description |
|---------|-------------|
| `smaqit init` | Scaffold `.smaqit/` and `.github/agents/` |
| `smaqit develop` | Run develop phase |
| `smaqit deploy` | Run deploy phase |
| `smaqit validate` | Run validate phase |
| `smaqit status` | Show current state |

## License

MIT
