# Agentic Swarms

> **Architecting Multi-Agent Swarms: Orchestration, Cost Optimization, and Quality Assurance**

A field-tested **Orchestrator-Worker-Adversary (OWA)** architecture for multi-agent software engineering. This repository contains the research paper, a ready-to-use skill package, scripts, and prompt templates that let engineering teams deploy cost-effective, high-quality agentic coding workflows in production.

## What's Inside

```
agentic-swarms/
├── README.md                          ← you are here
├── paper/
│   ├── Architecting-Multi-Agent-Swarms.pdf    ← the paper (PDF)
│   └── Architecting-Multi-Agent-Swarms.docx   ← the paper (editable)
├── skill/                             ← drop-in OWA skill package
│   ├── SKILL.md                       ← the main skill (7 phases + pitfalls)
│   ├── references/                    ← detailed setup guides
│   │   ├── owa-loop-reference.md
│   │   ├── model-allocation-template.md
│   │   ├── cost-optimization-checklist.md
│   │   ├── post-build-audit.md
│   │   ├── self-hosted-ci-setup.md
│   │   └── path-filtered-ci.md
│   ├── templates/                     ← agent prompt templates
│   │   ├── orchestrator-prompt.md
│   │   ├── worker-prompt.md
│   │   ├── adversary-prompt.md
│   │   ├── verifier-prompt.md
│   │   └── owa-record-template.md
│   └── scripts/                       ← automation scripts
│       ├── pre-build-prune-template.sh
│       ├── post-build-restore-template.sh
│       ├── parallel-gate-template.sh
│       ├── local-gate-template.sh
│       └── block-checkpoint-template.sh
└── LICENSE                            ← MIT
```

## The Problem

Single-agent coding assistants are great for demos. But when you try to
ship real software with them, you hit three walls:

1. **Quality** — one agent writing and reviewing its own code is a
   conflict of interest. It will rubber-stamp its own mistakes.
2. **Cost** — agentic workflows burn tokens on context bloat, cloud CI
   round-trips, and irrelevant tool schemas shipped on every API call.
3. **Failure modes** — silent transport timeouts, stale cache false-greens,
   directory isolation breaches, asynchronous racing. These are
   misdiagnosed as "model failures" but are actually pipeline bugs.

## The Solution: OWA

The **Orchestrator-Worker-Adversary** pattern assigns agents opposing
incentives:

| Role | Job | Key Trait |
|---|---|---|
| **Orchestrator** | Plans, dispatches, merges | Judgment. Does NOT edit code. |
| **Worker** | Writes code | Polite, accurate generation. |
| **Adversary** | Tries to break the Worker's code | Zero incentive to be nice. |
| **Verifier** | Integration assurance at block-end | Runs once per block, not per task. |

Code quality goes **up** when you add the Adversary, because catches that
a human reviewer would rubber-stamp at 2am get caught by an agent whose
entire identity is "find the break."

## Measured Impact

Validated across a multi-block build run of a security-sensitive,
containerized backend:

| Metric | Before | After | Saving |
|---|---|---|---|
| Wall clock per subtask | 40-80 min | 20-45 min | ~50% |
| Token cost per block | $15-40 | $5-15 | ~65% |
| Cloud CI minutes | rapid quota exhaustion | 0 | 100% |
| System prompt overhead per call | ~20KB | 0 | ~20KB/call |
| Per-subtask context footprint | 20-30K tokens | ~5KB | ~99% |
| Verifier invocations per block | per-subtask | once per block | ~89% |

Full details in the [paper](paper/Architecting-Multi-Agent-Swarms.pdf).

## Quick Start

### 1. Read the paper

Start with
[`paper/Architecting-Multi-Agent-Swarms.pdf`](paper/Architecting-Multi-Agent-Swarms.pdf).
It's ~6 pages and covers the why, the architecture, the measured impact,
and the operational pitfalls.

### 2. Adopt the skill

The [`skill/`](skill/) directory is a project-agnostic skill package.
Every `<PLACEHOLDER>` is a customization point — find-and-replace with
your models, commands, and paths.

If you use [Hermes Agent](https://github.com/NousResearch/hermes-agent)
as your orchestration layer, install the skill:

```bash
# Copy the skill directory into your Hermes profile
cp -r skill/ ~/.hermes/profiles/<your-profile>/skills/software-development/owa-multi-agent-coding/
```

If you use a different orchestration layer (Claude Code with subagents,
a custom script runner, etc.), use the SKILL.md as a playbook and the
scripts/templates as starting points.

### 3. Assign your models

Edit [`skill/references/model-allocation-template.md`](skill/references/model-allocation-template.md)
and fill in the placeholders:

```yaml
orchestrator:
  model: <your-highest-reasoning-model>    # judgment, dispatch, merge eg: GLM5.2
worker:
  model: <your-best-code-model>            # code generation eg: GPT5.5 through codex
adversary:
  model: <your-best-testing-model> # destructive testing ($0) eg: Grok 4.1
verifier:
  model: <your-highest-reasoning-model>    # integration assurance (1x/block) eg: Composer 2.5
```

### 4. Run the OWA loop

Follow the 7 phases in [`skill/SKILL.md`](skill/SKILL.md):

0. **Strategic scoping** — cut the plan, don't generate it
1. **Model allocation** — match model nature to role, see example above
2. **OWA loop** — worker → gate → adversary → merge → record → prune
3. **Local-first execution** — no PRs per subtask, push once at block-end
4. **Context pruning** — disable irrelevant toolsets/skills/MCP
5. **Parallel gates** — run build/test/lint concurrently, save time
6. **Block-end verification** — one Verifier pass, fresh context
7. **Post-build audit** — verify against remote, dont rely on closure comments

## Key Principles

1. **The expensive model's best use is deciding what NOT to build.** Spend
   more time cutting the plan than generating it.

2. **Adding agents with opposing incentives makes quality go UP.** The
   Adversary has no reason to be polite. It writes failing tests on purpose.

3. **The orchestrator does not edit code.** Strip the file-patch tool from
   its profile. Every fix routes through a Worker and passes through the
   Adversary and the gate.

4. **The cloud round-trip is a tax, not a best practice.** If your CI
   hardware is your dev machine, running gates locally and pushing at
   block-end saves 50% wall clock and 65% token cost.

5. **Context is a financial budget.** Every loaded tool, skill, and MCP
   schema is tokens you pay for on every API call. Prune ruthlessly.

6. **Process steps that catch nothing are theater.** A per-subtask
   Verifier caught 0 novel issues across 9 subtasks in our audit. Move it
   to block-end; keep the Adversary per-subtask.

7. **An agent's closure comment is a claim, not evidence.** Always verify
   against the remote repository state.

8. **OWA is overkill for trivial changes.** Maintain a fast path for
   one-line config bumps and doc typos.

## Stack Compatibility

The skill is stack-agnostic. The examples lean toward compiled, testable
backends (Go, Rust, TypeScript) because that's where OWA's value is
highest, but the pattern works for any project with:

- A testable codebase (unit tests, integration tests)
- A build/lint/test gate you can run locally
- An orchestration layer that can dispatch and merge agent outputs

For non-testable work (prototypes, design exploration), use a single
agent — OWA needs something to break.

## Orchestration Layer

The OWA pattern requires an orchestration layer to manage handoffs, state,
and tool execution. This repository is orchestration-layer-agnostic, but
the skill was originally developed and tested with
[Hermes Agent](https://github.com/NousResearch/hermes-agent) by Nous
Research. The scripts use `<your-disable-command>` / `<your-enable-command>`
placeholders that map to your orchestrator's CLI.

## Citation

If you use this work, please cite:

```bibtex
@misc{syed2026agentic,
  author       = {Parvez Syed Mohamed},
  title        = {Architecting Multi-Agent Swarms: Orchestration, Cost Optimization, and Quality Assurance},
  year         = {2026},
  howpublished = {\url{https://github.com/parvezsyed/agentic-swarms}},
}
```

## License

MIT — see [LICENSE](LICENSE).

## Contributing

This is an open-source project. Contributions welcome:

- Adaptations for specific stacks (Go, Rust, Python, TypeScript, etc.)
- Integrations with other orchestration layers
- Additional pitfall documentation from real builds
- Metric data from your own runs

Open an issue or submit a PR.

## References

- He, J., Treude, C., & Lo, D. (2025). *LLM-Based Multi-Agent Systems for Software Engineering: Literature Review, Vision and the Road Ahead.* ACM Transactions on Software Engineering and Methodology.
- Kumar, R. (2026). *AgentForge: Execution-Grounded Multi-Agent LLM Framework for Autonomous Software Engineering.* arXiv preprint.
- Salim, M. (2025). *Tokenomics: Quantifying Where Tokens Are Used in Agentic Software Engineering.* arXiv preprint.
- Tran, K.-T., et al. (2025). *Multi-Agent Collaboration Mechanisms: A Survey of LLMs.* arXiv preprint.
- Wang, M., Xie, X., & Huo, Y. (2026). *TrajAudit: Automated Failure Diagnosis for Agentic Coding Systems.* arXiv preprint.
