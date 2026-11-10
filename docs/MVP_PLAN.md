# API Contract Assistant: start-to-finish MVP plan

**Status:** approved direction, implementation not started. **Delivery target:** one focused week, approximately 16–24 hours with AI implementation assistance, plus debugging buffer. These are estimates, not a promise. The web interface adds work compared with the earlier CLI estimate.

## Product in plain English

An API is the agreement between a backend and the software calling it. An OpenAPI document is a machine-readable description of that agreement: routes, input fields, and output shapes.

When a backend changes, frontend code can silently stop working. This app compares the old and new agreement, finds a small set of supported breaking changes, and inspects a supplied TypeScript client to explain where changes may matter. The user sees concrete evidence and can export a migration report.

Example: the old `/rewards` response returns `amount` as a number, but the new response returns it as a string. The detector identifies that type change. The agent searches the supplied client, reads a call to `amount.toFixed(2)`, and flags the line as potentially incompatible. It recommends parsing or preserving the numeric contract, without claiming to have run the user's application.

This is a web product developed from VS Code. The plan and agent instructions remain in the repository so a coding assistant can pick up work without relying on chat history. Implementation comes first; full explanations are produced after each phase so the owner can study the completed system later.

## What the finished demo looks like

One page with four areas:

1. **Inputs:** old/new spec uploads, optional TypeScript client upload, and a “Load example” button.
2. **Changes:** operation, affected field, change type, severity, and supported/unsupported notices.
3. **Impact:** candidate client references, agent explanation, source line citations, and limitations.
4. **Evidence:** expandable tool-call timeline and Download Markdown Report button.

Primary actions: “Compare contracts” (offline deterministic mode) and “Analyze client impact” (live agent mode when configured). Replay mode is distinctly labeled “Recorded demonstration”; never present saved output as a live model response.

No authentication, database, multi-page navigation, payments, arbitrary repository cloning, or automatic source changes. One coherent flow is enough.

## Fixed scope and technical choices

| Area | MVP decision |
|---|---|
| Frontend | React + TypeScript + Vite; ordinary CSS; single page |
| Backend | Python 3.12 + FastAPI; Pydantic models; in-memory request processing |
| Contract inputs | OpenAPI 3.0.x JSON only, explicitly limited subset |
| Client input | One UTF-8 `.ts` or `.tsx` file, at most 100 KB; repository-wide scanning is deferred |
| AI | One configurable Gemini model through the current official SDK; actual tested model/version recorded |
| Agent tools | `list_changes`, `search_client`, `read_client_lines` |
| Output | Validated JSON and downloadable Markdown |
| Tests | pytest for backend; Vitest for UI logic/components; one browser smoke walkthrough |
| CI | GitHub Actions: backend checks, frontend checks and production build |
| Delivery | Static replay demo on GitHub Pages; actual live analysis available locally |
| Storage | No persistent upload storage; run JSON is returned for explicit local saving |

Use current compatible package versions and commit lockfiles/pins after installation. A framework is not needed merely to call this an agent: the model must actually choose tools, receive their outputs, and decide when to finish.

### The supported OpenAPI subset

Implement only direct operation objects (`get`, `post`, `put`, `patch`, `delete`) with inline JSON object schemas. Compare direct properties of `requestBody.content.application/json.schema` and a `200` JSON response schema. The three supported change rules are:

1. **Removed operation:** an old method/path pair is absent in the new spec. Adding an operation is not breaking.
2. **Newly required request-body property:** a property enters the request schema's `required` list. Also flag an optional request body becoming required as a separate detail under this rule.
3. **Changed response-property type:** a property present in both old/new `200` JSON schemas changes its declared primitive type. Mark it a potentially breaking contract change, not proof of runtime failure.

For v0.1, no `$ref` resolution, nested schemas, arrays as modeled response properties, `allOf`/`oneOf`/`anyOf`, YAML, OpenAPI 3.1+, parameter comparison, response status migration, rename inference, or removal of individual response fields. Validate and identify unsupported structures in the inspected operations. Supported operations may still be compared, but the overall report must say **partial analysis** when any relevant content is unsupported. Never equate “zero supported changes found” with “API is fully compatible.”

Each detected change has a stable ID, method/path, rule, JSON pointer, old/new values, explanation, and severity. Stable ordering makes reports and regression tests reproducible.

## Architecture and data flow

```text
Browser: load example / upload JSON and client
    |
POST /api/compare
    |
Validate sizes + supported schema subset
    |
Deterministic rules -> changes + unsupported warnings
    |
Browser shows change cards and baseline references
    |
User explicitly selects Analyze client impact
    |
POST /api/analyze
    |
Revalidate/recompute changes -> bounded Gemini tool loop
    |
Allowed reads of supplied client -> tool trace
    |
Validate structured answer + citations
    |
Browser renders potential impact -> Markdown export
```

The backend recomputes changes for analysis rather than trusting a list of changes supplied by the browser. The agent cannot invent detector findings or silently rewrite source files. Source text and specs are untrusted data, not instructions.

Proposed API contracts:

| Route | Request | Response |
|---|---|---|
| `GET /api/health` | None | Service status and whether live analysis is configured; no secret values |
| `POST /api/compare` | Old/new spec JSON plus optional client name/content | Changes, unsupported warnings, candidate references, analysis coverage |
| `POST /api/analyze` | Same inputs plus configured mode | Changes, impact findings, citations, tool trace, timings, actual usage if available |
| `POST /api/report` | Validated report data | Markdown document attachment |

No server-side sessions are required. Keep endpoints synchronous from the user's perspective with a bounded duration; a job queue or websocket system is unnecessary. Set overall request and upload limits and show useful validation messages.

Planned source structure, to be created during implementation:

```text
backend/
  pyproject.toml
  app/{main.py,models.py,validation.py,diff.py,client_search.py}
  app/{agent.py,tools.py,provider.py,report.py}
  tests/
frontend/
  package.json
  src/{App.tsx,api.ts,types.ts,styles.css}
  src/components/{InputPanel,ChangeList,ImpactPanel,TracePanel}.tsx
examples/{type-change,removed-operation,required-field}/
evals/{cases.json,expected.json,README.md}
docs/phase-reports/
docs/{MVP_PLAN,STATUS,START_HERE,PHASE_REPORT_TEMPLATE}.md
.github/workflows/{ci,publish-demo}.yml
```

## Phase 1 — Foundation and the upload experience

**Plain English:** Create the app's shell so someone can open it, load the sample files, and submit them to the backend. This connects the browser to Python without pretending change detection exists yet.

**Implementation:**

- Create backend/frontend projects, version declarations, dependency files, and ignored local environments.
- Add FastAPI health and validation-only compare endpoints. Define shared response shapes and a typed frontend request helper.
- Build the single-page upload layout with file names, parse errors, loading/error states, and example loading.
- Create synthetic old/new specs and a small TypeScript rewards client. Clearly mark all examples synthetic.
- Validate JSON syntax, OpenAPI version, file size, and client extension/encoding. Do not trust client-side checks alone.
- Return a clear “Comparison logic not implemented yet” status until Phase 2; no fabricated results.
- Document two-terminal startup commands for PowerShell and exact local ports.

**Acceptance:** A fresh setup starts both services; example data reaches the backend; invalid JSON and oversized input produce readable errors; a failed backend connection is visible in the UI. Health and validation tests pass. The UI is inspected in a browser.

**Expected effort:** 3–4 hours.

**Post-phase explanation:** Trace file selection through React state, the typed request, FastAPI/Pydantic validation, and the rendered response. Explain frontend/backend roles, the local development proxy, dependency files, and how errors travel back to the user.

## Phase 2 — Deterministic breaking-change detection

**Plain English:** Teach ordinary Python code to compare the two agreements. It should find supported differences consistently, without depending on AI judgment.

**Implementation:**

- Normalize paths/methods into operation records and compare old/new sets.
- Implement the three rules above as pure functions with structured change objects.
- Include JSON-pointer evidence, stable change IDs, and deterministic sorting.
- Detect unsupported constructs and show a coverage summary: full within the supported subset, partial, or unsupported.
- Wire real comparison results into the change list; remove the temporary Phase 1 placeholder.
- Add tests for each positive rule, unchanged specs, nonbreaking additions, multiple simultaneous changes, missing content/schema, and unsupported input.

**Acceptance:** The three example cases produce the exact expected changes. An unchanged spec yields zero changes without misleading compatibility claims. Unsupported `$ref`/composition inputs visibly report limitations. Tests compare structured findings, not wording alone.

**Expected effort:** 3–4 hours.

**Post-phase explanation:** Walk through old/new JSON to normalized operations, each rule, a change object, and a UI card. Explain a required field, a response type, set comparison, JSON pointers, and why this part does not need an LLM.

## Phase 3 — Client inspection and useful reports

**Plain English:** Show where a contract change might affect the supplied client, and create a report a developer can actually use.

**Implementation:**

- Create deterministic searches for the endpoint and changed property names in the uploaded TypeScript file. Return numbered snippets and search terms.
- Label matches “candidate references”; matching a string is not proof of data flow or a runtime bug. Include an unrelated same-name variable in examples to illustrate the limitation.
- Build impact and source-snippet components, empty states, and the report export button.
- Produce Markdown containing input identifiers/hashes, changes, coverage limits, candidate references, and a statement that no client code was executed.
- Add tests for line numbers, no matches, duplicate matches, large lines, and report escaping. Render all uploaded text as plain text; do not interpret HTML embedded in it.

**Acceptance:** A user can compare two specs, inspect actual client lines, and download an accurate report with no API key. Uploads cannot render active HTML. A misleading lexical match is identified as a candidate rather than a proven defect.

**Expected effort:** 2–3 hours.

**Post-phase explanation:** Trace a changed field into search terms, line indexing, candidate references, browser rendering, and Markdown output. Explain why lexical search has false positives and how this baseline will be compared to the agent later.

## Phase 4 — A bounded AI agent with evidence

**Plain English:** Let the model investigate the detected changes by choosing which client lines to inspect, then explain potential impact using those lines as evidence.

**Implementation:**

- Implement a provider adapter and an offline fake provider for tests; keep credentials and model configuration on the backend.
- Define three typed tool schemas: list the recomputed changes, search the uploaded client, and read a bounded line range from that client.
- Give the model a change summary, tool descriptions, and a structured final-response schema. Do not dump the entire client into the first prompt; tool use should have a purpose.
- Run the tool loop: model requests a tool; backend validates arguments; tool executes; result is returned to the model; repeat until the final answer or limit.
- Cap the run at 6 tool calls, 4 model turns, 30 seconds total, and a configured token/output budget. Bound retries within that same deadline.
- Validate findings against real change IDs and citations against lines actually returned by the tools. Reject unsupported citations with a clear error or at most one repair attempt within the budget.
- Record tool name, sanitized arguments, start/duration, status, model ID, prompt version, and actual token usage where the provider supplies it. Do not log credentials or full uploaded source by default.
- Add UI states for unconfigured key, running analysis, provider failure, abstention, successful result, and recorded replay.
- Before the first live request, show that supplied code/spec content is sent to the configured provider. Use synthetic fixtures for the public demo and evaluation.

**Acceptance:** A real configured-provider run shows meaningful tool selection and evidence-backed output. Mocked tests cover invalid tool names, out-of-range lines, malformed responses, hallucinated citations, timeout, and instruction-like source comments. If no live key is available, report the phase as partially verified; do not claim live agent success.

**Expected effort:** 4–5 hours.

**Post-phase explanation:** Trace one complete live run, including model input, tool request, validated arguments, returned snippet, subsequent turn, final output, citation checks, and UI. Distinguish provider API calls, local tools, deterministic facts, and model interpretations. Explain how limits prevent endless loops and why source comments cannot authorize actions.

## Phase 5 — Evaluation and reliability

**Plain English:** Measure whether the tool is useful and where it fails. Tests check software behavior; evaluations check the quality of the agent's conclusions.

**Implementation:**

- Create 12 small labeled cases: 6 development cases and 6 held-out cases. Each split includes the three supported rule types, a nonbreaking example, a misleading same-name reference, and an unsupported/insufficient-evidence example.
- Keep expected answers in evaluation-only files, outside tool access. Freeze the held-out labels before prompt tuning; version changes to fixtures.
- Evaluate the same deterministic detector in both modes. Separately compare baseline lexical candidate selection with agent-selected impact findings using human-labeled relevant lines.
- Record exact-match change detection, relevant-reference precision/recall, citation validity, required abstentions, latency, and provider usage. Show numerators and denominators; six held-out cases are a small demonstration, not a general benchmark.
- Run stochastic agent cases twice if budget permits and show differences. If not, state that only one run was performed.
- Add regression tests for the failure cases discovered. Use a fixed mocked provider in routine tests; keep live evaluation opt-in with an explicit request limit.
- Save a report with code revision, dataset version, model/prompt configuration, per-case outcomes, and failure analysis. Avoid an LLM grading its own answers as the only judge.

**Acceptance:** Reports are reproducible from saved inputs/configuration; no expected labels are exposed to the agent; at least one real failure/limitation is documented. All deterministic regression tests pass. Metrics are measured, even if disappointing; targets are not substituted for results.

**Release thresholds:** Every emitted citation must resolve to inspected text; all boundary tests pass; supported detector cases match labels. If agent relevance is poor, keep the mode explicitly experimental and discuss the result rather than hiding it.

**Expected effort:** 2–3 hours.

**Post-phase explanation:** Explain the difference between unit tests and evaluation, show one correctly diagnosed case and one failure, compute a metric from its numerator/denominator, and explain what the tiny dataset cannot prove.

## Phase 6 — CI, public demo, and portfolio release

**Plain English:** Make the project easy to check, demonstrate, and understand. Pushes should automatically run checks, and passing main-branch updates should refresh a clearly labeled replay demo.

**Implementation:**

- Add GitHub Actions to install pinned dependencies, run backend tests/lint, run frontend tests/type checking, and build the frontend on pushes and pull requests.
- Add a separate GitHub Pages delivery workflow using only synthetic examples and recorded, sanitized results. Deploy only after the same revision passes checks.
- The public static demo supports viewing sample reports and traces; it does not pretend to run the Python backend or make live model calls. Document local live mode separately.
- Verify a fresh clone's setup, offline end-to-end flow, live mode when configured, report download, and public replay page. Confirm no secrets entered Git history.
- Update README to describe only implemented features. Add architecture, evaluation results, limitations, setup, screenshots, and a two-minute demo outline.
- Create docs/PORTFOLIO.md with evidence-backed resume bullets and a LinkedIn draft. Replace all result placeholders with measured values or remove them.
- Prepare a v0.1 release and tag when authorized. Record CI run and deployment URLs, not just intended workflow files.

**Acceptance:** CI has actually passed on GitHub for the final revision; the replay URL works and is visibly labeled; a fresh clone reproduces offline behavior; evaluation evidence and all phase walkthroughs are available. A workflow file alone is not evidence of working CI/CD.

**Expected effort:** 2–3 hours, with extra buffer for hosting setup.

**Post-phase explanation:** Trace a commit through GitHub Actions jobs, tests, build artifacts, and the Pages deployment. Explain CI versus CD, replay versus live mode, secret handling, and how a reviewer can reproduce the project.

## Work schedule and scope control

| Day | Focus |
|---|---|
| 1 | Phase 1: web foundation and validation |
| 2 | Phase 2: supported diff rules |
| 3 | Phase 3: useful offline workflow and report |
| 4 | Phase 4: actual tool-calling agent |
| 5 | Phase 5: evaluation and regression fixes |
| 6 | Phase 6: CI, replay delivery, portfolio material |
| 7 | Buffer, fresh-clone checks, and reviewing saved explanations |

If behind schedule: simplify styling, keep one sample client file, shorten the report design, and use fewer live runs with honest disclosure. Do not cut evidence validation, error handling, or the requirement to distinguish planned work from completed work. Deploying a live backend is stretch scope, not needed for v0.1.

## Definition of done for resume use

- The web workflow works with new supported inputs, not just scripted example outputs.
- The three deterministic rules and unsupported-input behavior are tested.
- At least one live agent run is verified and its tools/citations can be explained.
- Evaluation results and limitations are published with actual denominators.
- Automated checks pass on GitHub, and any delivery claim has an observed deployment.
- The README clearly identifies your contributions and the use of AI assistance where relevant.
- All six phase implementation explanations are saved for study.

Suggested future bullets, only after verification:

- Built a React/TypeScript and FastAPI developer tool that detects supported OpenAPI breaking changes and uses an AI agent to inspect client code and produce migration reports with source citations.
- Added automated regression tests, bounded tool execution, and evaluation on [actual count] labeled cases; implemented GitHub Actions CI and automatic delivery of a labeled replay demo.

Do not claim broad OpenAPI compliance, automatic bug fixes, production adoption, or a measured productivity improvement unless independently established. A complete, narrow tool with honest limitations is the intended deliverable.

## Primary references

- [OpenAPI 3.0.3 specification](https://spec.openapis.org/oas/v3.0.3.html): the explicitly targeted schema version family, not a claim that it is the newest standard.
- [Gemini function calling](https://ai.google.dev/gemini-api/docs/function-calling): implementing model requests for defined tools.
- [FastAPI testing](https://fastapi.tiangolo.com/tutorial/testing/): backend endpoint tests.
- [GitHub Actions Python CI](https://docs.github.com/en/actions/tutorials/build-and-test-code/python): automated checks and artifacts.

Consult current official deployment/provider instructions during implementation. This plan deliberately defines behavior and scope without pretending untested commands or future model identifiers are already verified.
