# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-10

### Added

- **10 reusable skills** for software engineering workflows:
  - `implement` — 6-phase implementation model (clarify, analyze, design, implement, test, review)
  - `code-review` — Systematic review against SOLID, Clean Code, DRY, KISS, YAGNI, security, performance
  - `refine-ticket` — Transform placeholder tickets into implementation-ready specs
  - `commit` — Analyze git diffs and suggest Conventional Commit messages
  - `create-ticket` — Create planning tickets (simple or epic)
  - `create-adr` — Create Architecture Decision Records
  - `project-status` — Show milestone progress and recommend next actions
  - `ideate` — Brainstorm and validate feature ideas through critical dialogue
  - `investigate` — Diagnose issues using Grafana (Prometheus + Loki) via MCP
  - `scaffold-feature` — Generate complete feature scaffolds from existing reference code
- **Soft validation** — Skills check CLAUDE.md for required context and ask for missing info instead of failing
- **Quality checklists** for `implement`, `code-review`, and `refine-ticket`
- **Examples** for different tech stacks:
  - .NET API + Blazor WASM (`examples/claude-md-dotnet.md`)
  - Node.js + React monorepo (`examples/claude-md-node-react.md`)
  - Python FastAPI + HTMX (`examples/claude-md-python-fastapi.md`)
  - Planning directory structure with tickets and ADRs (`examples/planning-structure.md`)
  - Workflow override patterns (`examples/workflow-overrides.md`)
- **Documentation:**
  - README with installation, skill overview, CLAUDE.md requirements matrix
  - Detailed CLAUDE.md requirements reference (`docs/claude-md-requirements.md`)
  - Customization guide (`docs/customization-guide.md`)

[1.0.0]: https://github.com/christophergoltz/claude-code-skills/releases/tag/v1.0.0
