# Autonomous Harness

> A Hermes-led framework for turning a user's intent into disciplined,
> evidence-backed software delivery.

Autonomous Harness is not a chat prompt or a collection of isolated coding
tricks. It gives an operator, Hermes, and coding agents one shared delivery
model: clear project truth, bounded work, explicit gates, and durable evidence.

```text
User -> Hermes -> orchestration -> coding agents -> evidence -> gate
                    ^                                  |
                    +---------- framework -------------+
```

The framework lives in this repository. The skills are the project-local
connection point that lets Hermes-directed coding agents follow it inside an
existing product.

## What It Gives You

- A delivery contract for product context, safety boundaries, architecture,
  testing, handoffs, learning, and release decisions.
- Hermes as the operator that turns user intent into coordinated work.
- Four installable skills that let coding agents initialize project state,
  create Spec Kit features, deliver vertical slices, and run evidence-backed
  gates.
- A durable `.autonomous/` record in the product repository so the next agent
  continues from project truth and evidence rather than chat history.

## Prerequisites

| Component | Role | Required when |
|---|---|---|
| [Hermes Agent](https://hermes-agent.nousresearch.com/docs/) | Operator and orchestrator | Running the full Harness loop |
| A coding agent such as [Codex](https://developers.openai.com/codex/) | Implements scoped work | Always |
| Skills CLI | Installs the agent procedures | Always |
| [Spec Kit](https://github.com/github/spec-kit) | Creates specs, plans, and tasks | Before `autonomous-spec` |

For a command-line Hermes install on macOS, Linux, or WSL2, the official
installer is:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Then complete Hermes provider setup before asking it to orchestrate work. See
the [Hermes quickstart](https://hermes-agent.nousresearch.com/docs/getting-started/quickstart/).

## How the Framework Works

1. The user gives Hermes an outcome, not an implementation transcript.
2. Hermes reads the framework and existing project truth, then coordinates the
   right planning, delivery, review, or gate work.
3. Coding agents use the installed skills to create durable project state,
   Spec Kit artifacts, implementation evidence, and handoffs.
4. Gates evaluate actual artifacts and evidence before work advances.
5. Hermes reports the decision and carries the learned context into the next
   slice.

The framework is deliberately strict about two things: unknown facts stay
unknown until resolved, and a passing claim needs evidence.

## Framework Reference

The complete operating model stays in this repository under
[`docs/framework/`](docs/framework/). Start with:

| Document | Purpose |
|---|---|
| [AUTONOMOUS.md](docs/framework/AUTONOMOUS.md) | Delivery flow, roles, artifacts, gates, and principles. |
| [HERMES.md](docs/framework/HERMES.md) | Operator, delegation, worktree, and recovery model. |
| [ROLE.md](docs/framework/ROLE.md) | Coding-agent startup and working rules. |
| [AGENTS.md](docs/framework/AGENTS.md) | Project roles, boundaries, and workspace model. |
| [SUBAGENT.md](docs/framework/SUBAGENT.md) | Shared vocabulary and handoff contract. |
| [PROJECT.md](docs/framework/PROJECT.md) and [GUARDRAILS.md](docs/framework/GUARDRAILS.md) | Product truth and non-negotiable safety rules. |

Architecture, data, design, testing, version-control, debugging, commit, and
learning references are indexed in [`docs/framework/README.md`](docs/framework/README.md).

## Install the Agent Bridge

The framework remains here for the operator and Hermes to read. A consuming
product installs only the skills:

```bash
cd existing-product
npx skills add Ntrakiyski/autonomous-harness -a codex
```

This adds four procedures under `.agents/skills/`:

| Skill | Agent responsibility |
|---|---|
| `autonomous-init` | Map existing project truth and create `.autonomous/` state. |
| `autonomous-spec` | Turn approved scope into one Spec Kit feature. |
| `autonomous-deliver` | Deliver one approved vertical slice with evidence and a handoff. |
| `autonomous-gate` | Make a PRD, specification, code, or ship decision from evidence. |

No framework files are copied into the product repository. The first-run skill
gives the operator the framework handoff while preserving the product's own
documentation as its source of truth.

## First Run

1. The operator reads the framework reference and configures Hermes.
2. Hermes directs a coding agent to run:

   ```text
   Use $autonomous-init to set up autonomous delivery in this project.
   ```

3. When the project is ready to specify work, initialize Spec Kit once:

   ```bash
   specify init --here --force --integration codex --integration-options="--skills"
   ```

4. Hermes drives the scoped feature through `autonomous-spec`,
   `autonomous-deliver`, and `autonomous-gate`.

The two artifact boundaries remain separate:

```text
.autonomous/                    # Harness lifecycle state, evidence, handoffs
.specify/                        # Spec Kit configuration and templates
specs/001-feature-slug/          # Spec Kit feature artifacts
```

## Framework and Skill Boundaries

| Stays in this repository | Installs into a product |
|---|---|
| Framework contracts, Hermes operations, templates, and rationale | Four skill folders under `.agents/skills/` |
| Gate methodology and mode prompts | The matching gate reference files |
| Documentation for operators and contributors | `.autonomous/` project state created by `autonomous-init` |

The gate skill owns its check prompts in
[`skills/autonomous-gate/references/`](skills/autonomous-gate/references/).
They guide dynamic, project-specific gates; they are never copied as a fixed
pass/fail checklist.

## Repository Layout

```text
README.md                 # Operator entry point
docs/framework/           # Full Hermes-led framework
skills/                   # Skills.sh installable agent bridge
```

[![skills.sh](https://skills.sh/b/Ntrakiyski/autonomous-harness)](https://skills.sh/Ntrakiyski/autonomous-harness)
