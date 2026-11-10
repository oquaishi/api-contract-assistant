# API Contract Assistant

**Status: planning complete; implementation has not started.**

A planned developer tool that compares two API specifications, detects supported breaking changes, and uses a bounded AI agent to inspect a TypeScript client and explain potential impact with file-and-line evidence.

The product will have a small React/TypeScript web interface and a Python/FastAPI backend. Development happens in VS Code; the product is not limited to a terminal interface.

## Start here

1. Read [AGENTS.md](AGENTS.md).
2. Read the complete [MVP plan](docs/MVP_PLAN.md).
3. Read [STATUS.md](docs/STATUS.md) to find the next unfinished phase.
4. Follow [VS Code setup and starting prompt](docs/START_HERE.md).

After every implementation phase, the agent must run the relevant checks, update status, and write a full explanation in `docs/phase-reports/` using the [report template](docs/PHASE_REPORT_TEMPLATE.md). Explanations describe actual implementation, not planned features. Learning can happen after delivery; explanations must still be saved as work progresses.

## Planned MVP

- Upload old/new OpenAPI JSON specifications and a small TypeScript client file.
- Detect three explicitly supported kinds of contract changes.
- Review a deterministic report without an API key.
- Run agent-assisted analysis with inspectable tool calls and source citations.
- Export a Markdown migration report.
- Measure results against labeled examples and a deterministic baseline.
- Run automated checks in GitHub Actions and deploy a labeled replay demonstration.

No working application, passing tests, evaluation scores, or deployment are claimed yet. See the plan for exact boundaries and release criteria.
