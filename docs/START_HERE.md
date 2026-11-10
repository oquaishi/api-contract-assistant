# Continue from VS Code

## Clone and open

In PowerShell, navigate to the parent folder where you keep coding projects, then run:

```powershell
git clone https://github.com/oquaishi/api-contract-assistant.git
cd api-contract-assistant
code .
```

If `code` is not recognized, open VS Code manually and choose File > Open Folder > api-contract-assistant. Then use Terminal > New Terminal.

This initial repository contains planning documents, not the implemented application. Do not run npm install or a server command yet: Phase 1 creates those files. The implementation agent should first check Python, Node.js, npm, and Git versions and give precise installation steps only for missing prerequisites. Target Python 3.12 and a supported Node.js LTS release; record the versions actually tested.

## Paste this into your coding agent

```text
Read AGENTS.md, CLAUDE.md, docs/MVP_PLAN.md, and docs/STATUS.md before changing anything.

Implement Phase 1 of the API Contract Assistant MVP in this repository. I am using VS Code on Windows with PowerShell. Build the planned web interface; working in a terminal does not mean the product should be terminal-only.

Prioritize working implementation and verification. I will study the explanations later, so do not block progress on lessons or quizzes. Follow the exact scope and acceptance criteria. Do not claim unfinished features work.

When the phase is complete, output a full front-to-back explanation of the actual implementation using docs/PHASE_REPORT_TEMPLATE.md, save it in docs/phase-reports/phase-01.md, and update docs/STATUS.md. Include files/functions, the complete input-to-output flow, tests actually run, commands to launch it, and known limitations.

Do not start the next phase in this session unless I ask. Do not publish secrets, make live paid API calls without a configured budget, or change unrelated files.
```

For a subsequent phase: “Read the project instructions and latest status. Implement the next unfinished phase, verify its acceptance checks, and save and output its full implementation explanation.”

If you prefer one uninterrupted build, explicitly ask the agent to complete all remaining phases. It must still write and output each phase report before proceeding, without requiring you to study it first.

## Working with two assistants

Use one assistant for implementation. Give the other the completed diff and ask it to identify correctness bugs, unsupported claims, and missed acceptance checks. Bring actionable findings back to the implementation session. Do not have both assistants editing the same checkout simultaneously.

## Local credentials

The live Gemini key belongs in a local ignored environment file or process environment. Never paste it into a public issue, commit, README, or phase report. Offline comparison, tests, and replay mode should work without a key. Phase 4 documents exactly how to configure the provider and budget.
