---
title: Agent harness methodology assessment
created_at: 2026-06-11T00:00:00Z
resolved: true
priority: medium
kind: analysis
related:
  - AGENTS.md
  - docs/00-overview.md
  - docs/01-architecture.md
  - docs/03-cockburn-method.md
  - docs/04-tdd-protocol.md
  - docs/build-harness.md
  - docs/goal-design.md
  - docs/ideation.md
  - scripts/completion-check.sh
  - scripts/next-task.sh
---

# Agent harness methodology assessment

## Question

Is this repository being built in the "standard" way for software development,
or is it a distinctive method created by this developer while using coding
agents without additional product plugins?

## Verdict

The method is a hybrid.

The ingredients are standard: TDD, Cockburn use cases, hexagonal architecture,
CI gates, executable specifications, pre-commit checks, and dogfooding. The
composition is distinctive: those practices are rearranged into a repository
local operating system that a looping coding agent can run repeatedly.

In other words, this is not just "vibe coding with tests." It is closer to:

```text
methodology docs -> use-case specs -> goal stack -> next-task router
  -> red/green/refactor -> gates -> state update -> commit/push
```

The unusual part is not that tests exist. The unusual part is that the
repository owns the work queue, the completion criteria, the agent-readable
state, and the anti-cheat checks.

## Evidence from the origin notes

The repository contains a direct record of the originating product discussion in
`docs/ideation.md`. The conversation starts by asking for Alistair Cockburn's
use-case definition technique, then asks what domain entities are needed for a
SaaS specialized in managing Cockburn-style use cases through CLI and API
surfaces.

That sequence matters. The codebase did not start from route handlers and then
grow documentation afterward. It appears to have started from a methodology
conversation, then turned that conversation into product concepts, domain
entities, use cases, architecture, and finally an agent-executable build loop.

The current domain model still carries that origin:

- `UseCase`, `Actor`, `Goal`, `Stakeholder`, `StakeholderInterest`, `Scenario`,
  and `Step` map directly to Cockburn concepts.
- `Revision`, `WorkSession`, `SpecBranch`, `MergeRequest`, and `Lock` extend the
  model for parallel coding-agent work.
- The Prisma schema explicitly calls itself a seed and notes that the domain
  language is still moving.

This is a normal way to explore a domain with an assistant, but the next step is
not normal: the result was converted into a durable harness that constrains
future agents.

## Standard parts

Several decisions are orthodox software engineering:

- `AGENTS.md` identifies the working style as Kent Beck-style test-first
  development.
- `docs/04-tdd-protocol.md` defines RED, GREEN, and REFACTOR as the basic loop.
- `docs/01-architecture.md` uses a hexagonal ports-and-adapters architecture:
  domain, application, ports, infrastructure, and adapters.
- `docs/03-cockburn-method.md` makes Cockburn's use-case method the
  authoritative domain reference.
- API, CLI, app, and contract packages are separated in a pnpm workspace.
- Unit, integration, e2e, CLI, web, and contract tests exist across the
  workspace.
- GitHub Actions separate fast blocking checks from slower verify and
  world-health sweeps.

These are not personal inventions. They are recognizable TDD, clean/hexagonal
architecture, CI, and executable-specification practices.

## Distinctive parts

The distinctive part is the repository-local agent harness.

The harness is built from ordinary files, but the combination is specialized:

- `goals/<n>-<name>.md` describes a mission in natural language.
- `goals/<n>-<name>.gates.sh` mechanically verifies that mission.
- `goals/<n>-<name>.next-task.sh` emits the next action hint.
- `scripts/completion-check.sh` runs the goal gate stack and writes the first
  failing goal to `.state/active-goal`.
- `scripts/next-task.sh` dispatches to the active goal's next-task script.
- `docs/state/*` stores agent-readable progress, next task, blockers, learnings,
  and test-plan state.
- `scripts/check-gate-rigor.sh` checks whether universal claims in goal docs are
  backed by enumerating gates.
- `scripts/commit-check.sh` runs staged-impact checks at commit time, while
  full regression is reserved for push, CI, or manual verify.

This is not a normal backlog. It is a small execution system. A human can read
it, but it is optimized for a looping agent that repeatedly asks: what is the
active goal, what is the next task, what gate is failing, and what evidence
would prove completion?

## Product and process mirror each other

The product itself is also an agent harness.

`docs/00-overview.md` defines vspec as a tool for environments where humans work
with multiple parallel AI coding agents. `docs/07-cli-spec.md` makes the CLI the
primary surface for both humans and AI agents. The API and CLI expose
agent-consumable payloads with context, warnings, affected files, and
`suggested_next_actions`.

That means the codebase is dogfooding the same belief twice:

1. In the product: use cases, sessions, locks, branches, revisions, and agent
   envelopes help external coding agents work safely.
2. In the repository: goals, gates, next-task routing, state files, and commit
   hooks help the local coding agent build the product safely.

This mirror is the clearest sign that the workflow is intentional rather than
accidental.

## Git-history signal

The Git history supports the same reading.

Early commits follow very explicit TDD naming, such as `red: UC-...`,
`green: UC-...`, and `refactor: UC-...`. Later commits include many harness and
contract operations: gate performance, findings, agent output contracts, shared
contract migrations, and verification improvements.

That pattern suggests a two-phase evolution:

1. First, the developer used a strict use-case-by-use-case TDD loop to build the
   MVP surface.
2. Then, as the project grew, the developer evolved the harness itself so agents
   could keep making changes without relying on human memory.

The workflow therefore became more than TDD. It became TDD wrapped in a
repository-owned control plane.

## What is not happening

This repository is not evidence that the developer used an external plugin as
the main abstraction. The important machinery is local:

- Markdown specs.
- Shell gates.
- State files.
- Commit hooks.
- Vitest suites.
- CI workflows.
- CLI/API contracts.

It is also not pure free-form prompting. The assistant is constrained by files,
tests, scripts, and commits. The repository narrows the agent's choices.

## Practical assessment

For a normal team, the transferable parts are:

- Write the domain language down before implementation.
- Use one use case per iteration.
- Make tests executable acceptance evidence.
- Keep API/CLI contracts explicit.
- Run staged-impact checks before commits and full sweeps before sharing.
- Keep agent instructions short, durable, and local to the repo.

The distinctive, higher-cost parts are:

- Maintaining a numbered goal stack.
- Keeping `.md`, `.gates.sh`, and `.next-task.sh` in sync.
- Writing meta-gates that check the rigor of other gates.
- Managing active-goal state.
- Treating docs, gates, and findings as an agent control plane.

Those are valuable when the goal is long-running autonomous or semi-autonomous
agent work. They are probably excessive for a small conventional app unless the
team actually intends to delegate substantial work to coding agents.

## Final judgment

This is a standard-methodology foundation with a custom agent-execution layer on
top.

Calling it "standard software development" hides the most important part: the
repository has been shaped so a coding agent can keep moving without repeatedly
asking a human what to do next.

Calling it "just this developer's quirky method" also misses the point: the
method is built from serious existing practices, especially TDD, Cockburn use
cases, hexagonal architecture, and CI.

The accurate label is: **test-first, use-case-driven development packaged as a
repository-local autonomous build harness for coding agents**.
