<p align="center">
  <strong>OpenCode Kit</strong><br/>
  Reusable configuration kit for <a href="https://opencode.ai">OpenCode</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/OpenCode-config_kit-FF6B35?style=flat-square" alt="OpenCode Config Kit"/>
  <img src="https://img.shields.io/badge/agents-5-1a73e8?style=flat-square" alt="Agents"/>
  <img src="https://img.shields.io/badge/skills-4-f9ab00?style=flat-square" alt="Skills"/>
  <img src="https://img.shields.io/badge/commands-5-0d904f?style=flat-square" alt="Commands"/>
  <img src="https://img.shields.io/badge/languages-31_via_tree--sitter-00897b?style=flat-square" alt="Languages"/>
</p>

---

Structured multi-agent development workflow with built-in planning, implementation, and code review phases. Supports any language and framework — Go, Python, TypeScript, Rust, Java, and 26 more via tree-sitter analysis.

---

## 📑 Table of Contents

- [⚡ Quick Start](#-quick-start)
- [🔧 Commands](#-commands)
- [🏗 Architecture](#-architecture)
- [🔌 MCP Servers](#-mcp-servers)
- [📂 Project Structure](#-project-structure)
- [📐 Conventions](#-conventions)

---

## ⚡ Quick Start

### Installation

```bash
curl -sL https://raw.githubusercontent.com/0rac1e/opencode-kit/refs/heads/main/install.sh | bash
```

### Update existing installation

```bash
curl -sL https://raw.githubusercontent.com/0rac1e/opencode-kit/refs/heads/main/install.sh | bash -s -- --update
```

### First Steps

```bash
# 1. Edit AGENTS.md — update Language Profile to match your project stack
# 2. Analyze codebase and generate PROJECT-KNOWLEDGE.md
/project-researcher

# 3. Validate configuration
/init
```

### Options

```bash
KIT_VERSION=v1.0.0 bash install.sh    # install specific version
INSTALL_DIR=/path/to/project bash install.sh --update   # install to specific directory
```

<details>
<summary>Manual Installation (advanced)</summary>

```bash
git clone https://github.com/hex0xdeadbeef/opencode-kit.git
cd opencode-kit
bash install.sh                        # install to current directory
bash install.sh --update               # update existing installation

# Or copy manually:
cp -r .opencode/ /path/to/your/project/
cp AGENTS.md /path/to/your/project/
cp opencode.json /path/to/your/project/
# Merge .gitignore manually
```

</details>

---

## 🔧 Commands

### `/workflow` — Full Development Cycle

The main command that orchestrates the entire development process. Executes all phases sequentially with user confirmation between steps.

**Pipeline:** `task-analysis` → `planner` → `plan-review` (agent) → `coder` → `code-review` (agent)

```bash
/workflow Add new REST endpoint for profiles
/workflow --auto Implement resource update         # autonomous mode, no confirmations
/workflow --from-phase 3                            # resume from specified phase
```

<details>
<summary>⚙️ Modes & Phases</summary>

**Modes:**

| Mode        | Flag             | Description                                |
| ----------- | ---------------- | ------------------------------------------ |
| Interactive | _(default)_      | Confirmation before each phase             |
| Autonomous  | `--auto`         | All phases automatically, no confirmations |
| Resume      | `--from-phase N` | Resume from specified phase                |

**Phases:**

| #   | Phase          | Description                                                                                         |
| --- | -------------- | --------------------------------------------------------------------------------------------------- |
| 1   | Task Analysis  | Complexity classification (S/M/L/XL) and route selection                                            |
| 1.5 | Design         | Requirements exploration + approach selection _(L/XL only, optional for M new_feature/integration)_ |
| 2   | Planning       | Codebase research, implementation plan creation                                                     |
| 3   | Plan Review    | Plan validation against architecture _(skipped for S-complexity)_                                   |
| 4   | Implementation | Code writing strictly per approved plan, running tests                                              |
| 5   | Code Review    | Change review: architecture, security, quality                                                      |
| 6   | Completion     | Git commit + lessons learned _(if non-trivial)_                                                     |

</details>

**Result:** implemented, tested, and reviewed code with a git commit.

---

### `/planner` — Implementation Planning

Researches the codebase and creates a detailed implementation plan with code examples and acceptance criteria. Does not modify project files.

```bash
/planner Add pagination to list endpoint
/planner --minimal Add field to model               # minimal plan without deep research
```

**Result:** plan file at `.opencode/prompts/{feature}.md`

---

### `/coder` — Code Implementation

Implements code strictly per approved plan. Runs formatting, linting, and tests after implementation.

```bash
/coder                          # auto-find plan in prompts/
/coder my-feature               # implement specific plan
```

**Result:** working code with passing tests + evaluate output with deviation documentation.

---

### `/project-researcher` — Project Analysis

Autonomous agent for deep codebase analysis: architecture, dependencies, and DB schema. Generates `PROJECT-KNOWLEDGE.md` used by other commands as context.

```bash
/project-researcher
```

---

### `/review-checklist` — Review Checklist Reference

Displays the code review checklist: architecture, security (OWASP), code quality, performance.

```bash
/review-checklist
```

---

### 🗺 Command Selection Guide

| Scenario                                        | Command               |
| ----------------------------------------------- | --------------------- |
| Full feature implementation from scratch        | `/workflow`           |
| Autonomous implementation without confirmations | `/workflow --auto`    |
| Need a plan before writing code                 | `/planner`            |
| Plan approved, need implementation              | `/coder`              |
| Setting up kit in a new project                 | `/init`               |
| Understand project structure                    | `/project-researcher` |

---

## 🏗 Architecture

The system is a **5-phase development pipeline** managed by the orchestrator (`/workflow`), which sequentially delegates work to specialized agents. Each agent has a strictly defined responsibility zone, model assignment, and skill set.

<details>
<summary>🔄 Development Pipeline</summary>

```mermaid
flowchart TB
    subgraph STARTUP ["Startup"]
        TA["Task Analysis<br/>(S/M/L/XL)"] --> S1["Memory search"]
        S1 --> S3["Session recovery check"]
    end

    S3 -->|S| ROUTE_S["Minimal route:<br/>skip Plan Review"]
    S3 -->|M| ROUTE_M["Standard route"]
    S3 -->|L| ROUTE_L["Full route +<br/>Sequential Thinking"]
    S3 -->|XL| ROUTE_XL["Full route +<br/>ST required"]

    ROUTE_S --> PLANNER
    ROUTE_M --> PLANNER
    ROUTE_L --> PLANNER
    ROUTE_XL --> PLANNER

    subgraph PHASE1 ["Phase 1: Planning — /planner (plan agent)"]
        PLANNER["Understand scope"] --> RESEARCH["Research codebase"]
        RESEARCH --> DESIGN["Design solution"]
        DESIGN --> DOCUMENT["Write plan to<br/>prompts/feature.md"]
    end

    RESEARCH -.->|"L/XL: Task tool"| CRES["code-researcher<br/>(haiku)"]

    DOCUMENT --> CHECK_S{"S-complexity?"}
    CHECK_S -->|Yes| EVALUATE
    CHECK_S -->|No| PLAN_REVIEW

    subgraph PHASE2 ["Phase 2: Plan Review — plan-reviewer (subagent)"]
        PLAN_REVIEW["Read plan +<br/>check architecture"]
        PLAN_REVIEW --> VERDICT1{"Verdict?"}
    end

    VERDICT1 -->|APPROVED| EVALUATE
    VERDICT1 -->|NEEDS_CHANGES| LOOP1{"Iteration < 3?"}
    VERDICT1 -->|REJECTED| STOP1["STOP pipeline"]

    LOOP1 -->|Yes| PLANNER
    LOOP1 -->|"No: limit reached"| STOP2["STOP: show summary,<br/>request user help"]

    subgraph PHASE3 ["Phase 3: Implementation — /coder (build agent)"]
        EVALUATE{"Evaluate plan:<br/>PROCEED / REVISE / RETURN"}
        EVALUATE -->|PROCEED| IMPLEMENT["Implement Parts<br/>in dependency order"]
        EVALUATE -->|REVISE| ADJUST["Note adjustments"] --> IMPLEMENT
        IMPLEMENT --> SIMPLIFY{"SIMPLIFY<br/>(L/XL, ≥5 parts)"}
        SIMPLIFY -->|"applied / skipped"| VERIFY{"fmt + lint + test"}
        VERIFY -->|PASS| HANDOFF3["Form handoff"]
        VERIFY -->|"FAIL (max 3x)"| STOP3["STOP: test failures,<br/>request manual fix"]
    end

    EVALUATE -->|RETURN| PLAN_REVIEW

    IMPLEMENT -.->|"L/XL: Task tool"| CRES

    HANDOFF3 --> CODE_REVIEW

    subgraph PHASE4 ["Phase 4: Code Review — code-reviewer (subagent)"]
        CODE_REVIEW["Read diff +<br/>check architecture, security,<br/>tests, style"]
        CODE_REVIEW --> VERDICT2{"Verdict?"}
    end

    VERDICT2 -->|APPROVED| COMPLETION
    VERDICT2 -->|APPROVED_WITH_COMMENTS| COMPLETION
    VERDICT2 -->|CHANGES_REQUESTED| LOOP2{"Iteration < 3?"}

    LOOP2 -->|Yes| EVALUATE
    LOOP2 -->|"No: limit reached"| STOP4["STOP: show summary,<br/>request user help"]

    subgraph PHASE5 ["Phase 5: Completion"]
        COMPLETION["Git commit"] --> LESSONS{"Non-trivial?"}
        LESSONS -->|Yes| SAVE["Save lessons<br/>to Memory"]
        LESSONS -->|No| FINAL["Done"]
        SAVE --> FINAL
    end

    style STARTUP fill:#e0e0e0,color:#333,stroke:#999
    style PHASE1 fill:#1a73e8,color:#fff,stroke:#1557b0
    style PHASE2 fill:#9334e6,color:#fff,stroke:#7627bb
    style PHASE3 fill:#9334e6,color:#fff,stroke:#7627bb
    style PHASE4 fill:#9334e6,color:#fff,stroke:#7627bb
    style PHASE5 fill:#0d904f,color:#fff,stroke:#0a7040
    style STOP1 fill:#d93025,color:#fff,stroke:#b3261e
    style STOP2 fill:#d93025,color:#fff,stroke:#b3261e
    style STOP3 fill:#d93025,color:#fff,stroke:#b3261e
    style STOP4 fill:#d93025,color:#fff,stroke:#b3261e
    style CRES fill:#00897b,color:#fff,stroke:#00695c
```

</details>

<details>
<summary>📨 Handoff Data Flow</summary>

```mermaid
flowchart LR
    PL2["/planner"] -->|"artifact path<br/>key_decisions<br/>known_risks<br/>complexity"| PR2["plan-reviewer"]

    PR2 -->|"APPROVED:<br/>verdict, approved_notes,<br/>iteration N/3"| CO2["/coder"]
    PR2 -.->|"NEEDS_CHANGES:<br/>issues list"| PL2

    CO2 -->|"branch<br/>parts_implemented<br/>evaluate_adjustments<br/>deviations_from_plan<br/>risks_mitigated"| CR2["code-reviewer"]

    CR2 -->|"APPROVED:<br/>verdict, iteration N/3"| DONE2["completion"]
    CR2 -.->|"CHANGES_REQUESTED:<br/>issues[]"| CO2

    style PL2 fill:#1a73e8,color:#fff,stroke:#1557b0
    style PR2 fill:#9334e6,color:#fff,stroke:#7627bb
    style CO2 fill:#9334e6,color:#fff,stroke:#7627bb
    style CR2 fill:#9334e6,color:#fff,stroke:#7627bb
    style DONE2 fill:#0d904f,color:#fff,stroke:#0a7040
```

</details>

<details>
<summary>📦 Skill Loading</summary>

```mermaid
flowchart LR
    subgraph SKILLS ["Skills (on-demand loading)"]
        WP["workflow-protocols"]
        PLR["planner-rules"]
        CDR["coder-rules"]
        CRR["code-review-rules"]
    end

    WF2["/workflow"] --> WP
    PL2["/planner"] --> PLR
    CO2["/coder"] --> CDR
    PREV["plan-reviewer"] --> PRR
    CREV["code-reviewer"] --> CRR

    style SKILLS fill:#f9ab00,color:#333,stroke:#e69500
    style WF2 fill:#1a73e8,color:#fff,stroke:#1557b0
    style PL2 fill:#1a73e8,color:#fff,stroke:#1557b0
    style CO2 fill:#9334e6,color:#fff,stroke:#7627bb
    style PREV fill:#9334e6,color:#fff,stroke:#7627bb
    style CREV fill:#9334e6,color:#fff,stroke:#7627bb
```

</details>

### ⚙️ Model Routing

| Model      | Components                                                          | Purpose                                               |
| ---------- | ------------------------------------------------------------------- | ----------------------------------------------------- |
| **sonnet** | `/workflow`, `/planner`, `/coder`, `plan-reviewer`, `code-reviewer` | Deep reasoning, orchestration, implementation, review |
| **haiku**  | `code-researcher`                                                   | Fast read-only search                                 |

### 📊 Complexity Routing

| Complexity | Parts | Layers | Plan Review | Sequential Thinking | code-researcher |
| ---------- | ----- | ------ | ----------- | ------------------- | --------------- |
| **S**      | 1     | 1      | skip        | not needed          | skip            |
| **M**      | 2–3   | 2      | standard    | as needed           | skip            |
| **L**      | 4–6   | 3+     | standard    | recommended         | yes             |
| **XL**     | 7+    | 4+     | standard    | required            | yes             |

### 🔑 Key Principles

- **Sequential execution** — phases don't run in parallel
- **Handoff Protocol** — 4 typed payload contracts between phases
- **Context Isolation** — review phases run as isolated subagents (clean context, no authorship bias)
- **Loop Limits** — max 3 iterations per review cycle, then STOP and ask user
- **Checkpoint Protocol** — state saved after each phase for session recovery
- **Evaluate Protocol** — coder critically evaluates plan before implementation (PROCEED/REVISE/RETURN gate)
- **Conditional Deps Loading** — S-complexity skips heavy skill loading
- **Re-Routing** — pipeline adjusts route on complexity mismatch (downgrade/upgrade)

---

## 🔌 MCP Servers

Configure in `opencode.json`:

### Optional

| Server                | Package                   | Purpose                                |
| --------------------- | ------------------------- | -------------------------------------- |
| `sequential-thinking` | —                         | Structured reasoning for complex tasks |
| `context7`            | `@upstash/context7-mcp`   | Library documentation lookup           |
| `postgres`            | `@anthropic/mcp-postgres` | Required for `/db-explorer`            |

---

## 📂 Project Structure

```
.opencode/
├── agents/                # Autonomous agents
│   ├── plan-reviewer.md   # Plan validation agent (invoked by /workflow)
│   ├── code-reviewer.md   # Code review agent (invoked by /workflow)
│   └── code-researcher.md # Codebase exploration agent (hidden)
├── commands/              # Slash commands (/workflow, /planner, /coder, etc.)
├── skills/                # Reusable domain knowledge
│   ├── workflow-protocols/# Orchestration, handoff, checkpoints, re-routing
│   ├── planner-rules/     # Planning methodology, task analysis, data flow
│   ├── coder-rules/       # Implementation rules, evaluate protocol
│   └── code-review-rules/ # Security checklist (OWASP), review checklists
├── templates/             # Templates for creating new artifacts
├── prompts/               # Generated implementation plans
├── rules/                 # Cross-cutting constraints (architecture rules)
└── PROJECT-KNOWLEDGE.md   # Auto-generated project knowledge base

AGENTS.md                  # Project instructions (equivalent to CLAUDE.md)
opencode.json              # OpenCode configuration (permissions, agents, commands)
```

---

## 📐 Conventions

- Artifacts use YAML-first format (>80% YAML, minimal prose)
- Language: English for code, YAML keys, and artifact specs
- Size limits enforced by agents
- Examples use grep/glob patterns to find current code, not hardcoded snippets
- Commands use frontmatter for metadata (name, description, agent, model)
- Agents use frontmatter for configuration (name, description, mode, model, permissions)
