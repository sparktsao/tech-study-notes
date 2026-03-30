# Finding NEMO
## An Agent Ticket System to Find the Right NemoClaw
### _Make your Agents work together faster, better and safer_

<div align="center">
  <img src="claw.png" alt="Finding NEMO — Are you the right NEMO to take my tickets?" width="620"/>
  <br/>
  <sub><strong>NEMO: Network of Execution and Management Orchestration</strong> · Finding NEMO Proposal v1.0 · March 2026</sub>
</div>

---

> **NEMO**: **N**etwork of **E**xecution and **M**anagement **O**rchestration
> _"An agent ticket system that routes every task to the right NemoClaw"_

---

## 1. Purpose

### The Problem

Modern AI workflows are fragmented. A human gives an instruction; a single agent attempts everything; quality, safety, and accountability are afterthoughts. As task complexity grows — spanning code generation, research, document creation, infrastructure automation — no single agent is the right tool for every sub-problem. The result is slower output, correlated failure modes, and no audit trail.

### What Finding NEMO Is

Finding NEMO is a **multi-agent orchestration layer** modeled on how a well-run company hires and manages employees — except the employees are AI agents. The ticket system acts as the **company**: it posts Job Descriptions (JDs), qualifies candidates, coaches workers, enforces security controls, and evaluates performance when the work is done.

The goal is simple: **make your agents work together faster, better, and safer.**

- **Faster** — tasks are parallelized across pre-qualified specialized agents; no single agent context-switches between unrelated domains
- **Better** — the right agent for each job, verified before assignment; agent capability is declared and tested, not assumed
- **Safer** — the company (ticket system) maintains oversight at every stage: it does not merely dispatch and forget

**The three-stage lifecycle:**

| Stage | Company Analogy | What Happens |
|-------|----------------|--------------|
| **Post** | Write a Job Description | Ticket is created with task requirements, required capabilities, acceptance criteria, and budget |
| **Qualify & Assign** | Screen and hire | Agent must prove qualification (unit tests, reputation score, or config match) before checking out the ticket |
| **Coach, Control & Evaluate** | Manage, audit, appraise | System coaches the agent during execution, enforces security controls on inputs/outputs, then runs three independent evaluations on the result: **performance**, **security**, and **cost** |

### Where It Comes From

Finding NEMO synthesizes ideas from multiple layers of research and tooling:

| Layer | Source | Contribution |
|-------|--------|-------------|
| **Orchestration control plane** | [Paperclip](https://github.com/paperclipai/paperclip) | Persistent task queue, atomic checkout, budget enforcement, heartbeat loop, goal ancestry — the "company OS" for agents |
| **Agent privilege separation** | Tsao & Cheng — "Agent Privilege Separation: A Structural Defense Against Prompt Injection" (arXiv:2603.13424v1) | Structural isolation between unprivileged readers and privileged actors prevents prompt injection across any multi-agent system — not limited to OpenClaw; generalizes to any agent runtime |
| **Long-running agent harness** | Anthropic Engineering | Checkpointing, observability, and recovery patterns for agents that run beyond a single context window |
| **Security & independence** | Girard, TrendAI — SoD-First AI Code Security (March 2026) | The generator must never be the reviewer; multi-LLM independence, blind review, provenance tagging |

### Execution Runtime — OpenShell and NemoClaw

The execution layer is built on **OpenShell** and **NemoClaw**, enabling native integration with **NVIDIA's agent ecosystem** (NIM microservices, NeMo Agent Toolkit, CUDA-accelerated inference) — while remaining runtime-agnostic. A NemoClaw is any execution agent registered in the platform; it can run on NVIDIA infrastructure, Claude Code, Codex, Gemini, or any HTTP-compatible agent runtime.

> The name is intentional: in a vast ocean of tasks, NEMO's job is not to do all the swimming — it is to find the *right claw* and hand off with confidence, then verify the catch independently.

### NEMO in the Stack — Positioning vs Paperclip

NEMO and Paperclip occupy **different but complementary layers** of the agentic stack. Building NEMO on top of a Paperclip-like control plane is the intended architecture, not a competition.

| Layer | System | Responsibility |
|-------|--------|---------------|
| **Control plane** | Paperclip (or equivalent) | Org structure, task ownership, budget governance, audit trail, atomic checkout, approval gates |
| **Execution plane** | Finding NEMO | DAG-based task decomposition, capability-aware agent routing, privilege-separated execution, multi-tier evaluation loops, memory management |

**Paperclip is the company. NEMO is the execution engine the company runs.**

Where Paperclip enforces *who owns what and whether it happened*, NEMO enforces *how it is executed safely and correctly*. A Paperclip task that checks out to a NemoClaw worker enters NEMO's execution pipeline: it is decomposed into a DAG, routed to qualified agents, passed through privilege separation, and evaluated independently before the Paperclip ticket closes.

**NEMO's differentiating capabilities** relative to a raw Paperclip deployment:

| Capability | NEMO adds |
|-----------|-----------|
| DAG execution | Tasks have declared dependencies; NEMO schedules the graph, not just the queue |
| Adaptive planning | Planner decides agent count and topology dynamically per task complexity |
| Multi-agent reasoning loops | Evaluate → Refine → Re-execute until acceptance criteria pass |
| Security isolation | Structural privilege separation (Reader/Actor split) enforced at the platform level |
| Capability-aware routing | Agent Registry with qualification gates — not anonymous workers |
| 3-tier memory | Task-local, shared workspace, and global knowledge — shared context without shared privilege |

Taken together: **Paperclip as control plane + NEMO as execution plane ≈ Kubernetes + Ray for AI agents.**

---

## 2. Design Principles

These principles are non-negotiable and come directly from the three reference pillars:

| Principle | Source | Implication |
|-----------|--------|-------------|
| **Agent privilege separation** | Tsao & Cheng — arXiv:2603.13424v1 | Structurally separate unprivileged Reader agents (untrusted data, read-only) from privileged Actor agents (validated JSON only, tool access). This pattern is runtime-agnostic — it applies inside OpenClaw, NemoClaw, or any multi-agent execution layer. |
| **Separation of Duties (SoD)** | Girard, TrendAI (March 2026) | The agent that generates output must never be the agent that certifies it. Generator ≠ Reviewer. Execution ≠ Evaluation. |
| **Harness design for long-running tasks** | Anthropic Engineering | Long tasks must be checkpointable, observable, and recoverable. Agents operate via heartbeat loops, not fire-and-forget. |
| **Task graph over flat lists** | Architecture review | Tasks are nodes in a DAG, not items in a queue. Dependencies are declared explicitly; the scheduler respects ordering before parallelizing. Flat "spawn 100 agents" thinking breaks on interdependent tasks. |
| **Adaptive planning** | Architecture review | The Planner decides agent count and topology dynamically from task complexity — never a fixed N. Simple tasks get one agent; complex tasks get a graph. Spawning excess agents wastes budget and produces inconsistent outputs. |
| **Iterative refinement** | Architecture review | The operating loop is Plan → Execute → Evaluate → Refine → Repeat, not Plan → Execute → Done. Evaluation failures feed back into the Planner as re-decomposition signals, not dead ends. |
| **Shared context, separated privilege** | Architecture review | Agents in the same task graph share structured workspace memory — intermediate results, partial context — but never raw untrusted data. Memory tiers enforce privilege boundaries the same way Reader/Actor roles do. |

---

## 3. System Architecture

### 3.1 System Diagram

```mermaid
flowchart TD
    H(["👤 Human\ninstructs"])

    subgraph PLANNER["🧠 Assistant Agent  (Planner)"]
        direction TB
        BP["Adaptive decomposition:\nbuild task DAG · decide agent count dynamically\nApply best practices:\n· Agent Privilege Separation — Tsao & Cheng\n· Long-running harness — Anthropic\n· SoD-First — Girard, TrendAI"]
        CHK["Author Job Description per task:\nrequired capabilities · input/output schema\nacceptance criteria · budget · risk level\ndependencies (DAG edges)"]
        BP --> CHK
    end

    subgraph COMPANY["📋 NEMO Ticket System  (The Company)"]
        direction TB
        JD[/"🗒️ Job Description Tickets  (DAG)\nTask-A → Task-B → Task-C\n— capabilities required\n— acceptance criteria\n— budget ceiling\n— dependency edges"/]
        REG[("🗄️ Agent Registry\nqualified agents · reputation scores\nunit-test results · capability tags\ncost profiles")]
        GATE{"✅ Qualification Gate\nDoes agent meet JD requirements?\nunit tests · reputation · config"}
        COACH["🎓 Company Coach\nContext, guardrails, policies\nduring execution"]
        JD --> GATE
        REG --> GATE
        GATE -->|"qualified"| COACH
        GATE -->|"not qualified"| JD
    end

    subgraph MEMORY["🧠 Memory Layer"]
        direction TB
        ML["🔵 Task-local\n(per agent · cleared on task close)"]
        MW["🟡 Shared Workspace\n(per task graph · structured JSON only)"]
        MG[("🟢 Global Knowledge\n(RAG / DB · read-only for agents)")]
        ML ~~~ MW ~~~ MG
    end

    subgraph EXEC["⚙️ Execution Agent  ·  OpenShell / NemoClaw"]
        direction LR
        READER["🔍 Reader\n(unprivileged · untrusted data only)"]
        ACTOR["⚡ Actor\n(privileged · validated JSON only)"]
        READER -->|"structured JSON"| ACTOR
    end

    subgraph WORKERS["Specialized Workers"]
        direction LR
        AA["Agent-A\n(e.g. summarize)"] ~~~ AB["Agent-B\n(e.g. build/code)"] ~~~ AC["Agent-C\n(e.g. create asset)"]
    end

    subgraph EVAL3["🏛️ Independent Evaluation  (Post-execution)"]
        direction LR
        EP["📊 Performance Eval\noutput quality · task completion\nspeed · accuracy"]
        ES["🔒 Security Eval\nprivilege adherence · SoD check\nprompt injection · provenance"]
        EC["💰 Cost Eval\ntoken spend · budget variance\ncost-per-task · ROI"]
    end

    H -->|instructs| PLANNER
    PLANNER -->|"writes DAG tickets"| JD
    COACH -->|"coached handoff"| EXEC
    EXEC --> WORKERS
    EXEC <-->|"read/write structured JSON"| MW
    EXEC -->|"result + provenance tags"| EVAL3
    MG -->|"read-only context"| EXEC
    EP & ES & EC -->|"verdict · score · feedback"| COMPANY
    EP & ES & EC -->|"fail → re-plan signal"| PLANNER
    COMPANY -.->|"heartbeat · checkpoint"| EXEC
```

> **Reading the diagram:** The Planner **adaptively decomposes** instructions into a **DAG of Job Description tickets** — deciding agent count and topology from task complexity, not a fixed N. The Ticket System qualifies agents via the Agent Registry before any checkout. The **Memory Layer** provides three-tier context: task-local state per agent, structured shared workspace across the graph, and read-only global knowledge. Inside the Execution Agent, **privilege separation** keeps untrusted data in the Reader; only structured JSON reaches the Actor. On completion, three **independent evaluations** run. Critically, **evaluation failures feed back to the Planner as re-plan signals** — triggering re-decomposition rather than hard failure. This closes the Plan → Execute → Evaluate → Refine loop.

---

### 3.2 Component Detail

```
Human: instruct
  │
  ▼
┌─────────────────────────────────────────────────┐
│           ASSISTANT AGENT (Planner)             │
│  • Breaks task into sub-tasks                   │
│  • Applies best-practice decomposition rules    │
│  • Reviews task structure for speed/quality/    │
│    safety before dispatch                       │
└──────────────────────┬──────────────────────────┘
                       │ structured sub-tasks
                       ▼
┌─────────────────────────────────────────────────┐
│     PAPERCLIP-LIKE TASK PLATFORM (Jira layer)   │
│  • Task-1, Task-2, Task-3, …                    │
│  • Atomic checkout (one agent per task)         │
│  • Budget enforcement, audit log                │
│  • Goal ancestry (every task traces to mission) │
└───────────┬─────────────────────────────────────┘
            │ checkout
            ▼
┌─────────────────────────────────────────────────┐
│        EXECUTION AGENT (OpenShell / OpenClaw)   │
│  ┌──────────────┐   structured JSON             │
│  │ Reader Agent │ ──────────────────────────►   │
│  │ (read-only,  │                               │
│  │ untrusted    │   ◄── raw external content    │
│  │ data only)   │                               │
│  └──────────────┘                               │
│  ┌──────────────┐                               │
│  │ Actor Agent  │ ◄── validated JSON only       │
│  │ (privileged, │                               │
│  │ no raw input)│                               │
│  └──────────────┘                               │
│                                                 │
│  Dispatches to:  Agent-A │ Agent-B │ Agent-C    │
└───────────┬─────────────────────────────────────┘
            │ result
            ▼
┌─────────────────────────────────────────────────┐
│           EVALUATION AGENT (Independent)        │
│  • Different model lineage from Execution agent │
│  • Read-only access to outputs                  │
│  • Structured pass/fail/escalate verdict        │
│  • Posts result back to Paperclip task          │
└─────────────────────────────────────────────────┘
```

---

### 3.3 Task Graph — DAG Execution Model

Tasks in NEMO are **nodes in a directed acyclic graph**, not items in a flat queue. The Planner declares dependencies explicitly; the scheduler respects ordering before releasing tasks for parallel execution.

**Why this matters:** Many real tasks are interdependent. "Research competitors" must complete before "draft report"; "lint code" and "run tests" can parallelize; "deploy to production" must never run before "security review passes". A flat task list cannot express this — it produces either sequential bottlenecks or dangerous race conditions.

**DAG example:**

```
Task-A: research (no deps)  ─┐
Task-B: research (no deps)  ─┼─→ Task-C: synthesize (deps: A, B)
                              │         │
                              │         ▼
                              │   Task-D: draft report (dep: C)
                              │         │
                              └─→ Task-E: security review (dep: D)
                                        │
                                        ▼
                                  Task-F: deliver (dep: E, pass only)
```

**Scheduler rules:**

| Rule | Description |
|------|-------------|
| **Dependency gate** | A task may not be checked out until all declared `dependencies` have `verdict = pass` |
| **Parallel release** | Tasks with satisfied dependencies and no shared output targets are released simultaneously |
| **Failure propagation** | A task with `verdict = fail` and `max_retries` exhausted marks all downstream dependents `blocked` |
| **Re-plan trigger** | Planner is notified; it may re-decompose the subgraph rather than let the whole graph fail |
| **Cycle detection** | The platform rejects any DAG with a cycle at write time (`400 Bad Request`) |

**Adaptive planning rule:** The Planner chooses the graph shape from task complexity. A simple one-step instruction produces one task node. A multi-domain instruction produces a branching graph with parallel leaves and merge points. The Planner must justify agent count in the task's `planning_rationale` field.

---

### 3.4 Memory Layer

NEMO maintains a **three-tier memory model** that gives agents shared context without giving them shared privilege. Each tier has a defined scope, lifetime, and access control.

```
┌────────────────────────────────────────────────────────────────────┐
│  Tier 1: Task-local memory                                         │
│  Scope: one agent, one task checkout                               │
│  Lifetime: cleared when task closes (pass, fail, or reassign)      │
│  Access: exclusive to the checked-out agent                        │
│  Content: scratchpad, intermediate reasoning, tool outputs         │
└─────────────────────────────┬──────────────────────────────────────┘
                              │ structured JSON only
┌─────────────────────────────▼──────────────────────────────────────┐
│  Tier 2: Shared workspace memory                                   │
│  Scope: all agents in the same task graph (same `goal_id`)         │
│  Lifetime: persists until the graph's root task closes             │
│  Access: read/write for agents in the graph · read-only otherwise  │
│  Content: validated intermediate outputs from upstream tasks       │
│  Invariant: only structured JSON written here — never raw strings  │
└─────────────────────────────┬──────────────────────────────────────┘
                              │ read-only
┌─────────────────────────────▼──────────────────────────────────────┐
│  Tier 3: Global knowledge                                          │
│  Scope: platform-wide                                              │
│  Lifetime: persistent (managed externally — RAG, DB, vector store) │
│  Access: read-only for all agents · mutations require admin gate   │
│  Content: domain knowledge, policy docs, reference data            │
└────────────────────────────────────────────────────────────────────┘
```

**Security invariant:** The privilege boundary between tiers mirrors the Reader/Actor split. An Actor may write to Tier-1 (task-local) and Tier-2 (shared workspace) but only via the structured JSON format validated at checkout. A Reader may never write to any tier. No agent reads raw external content from Tier-2 — it must have already been sanitized by a Reader step before being stored there.

**Why this matters:** Without a shared workspace, agents in the same graph duplicate research, contradict each other, and amplify inconsistencies. Without a global knowledge tier, every agent starts from zero on domain context. Without per-tier access control, shared memory becomes a prompt injection vector.

---

## 4. Agent Roles and Constraints

### 4.1 Assistant Agent (Planner)

**Responsibility**: Receive human instructions (or re-plan signals from the Evaluation layer); adaptively decompose into a task DAG with explicit acceptance criteria, schema contracts, and dependency edges.

**Constraints**:
- Must produce a **task DAG**, not a flat list: every task declares its `dependencies` before being written to the platform
- Must decide agent count dynamically from task complexity — must not use a fixed N; must justify agent count in `planning_rationale`
- Must produce `input_schema` and `output_schema` for each task (JSON Schema or Zod-compatible)
- Must apply the decomposition checklist from §5 before dispatch
- Cannot directly execute tasks — it only writes to the task platform
- Must tag each task with `@plannedBy <model-id>` and `@plannedAt <timestamp>`
- On receiving a re-plan signal (evaluation `verdict = fail`): must assess whether to retry the failed node, re-decompose the subgraph, or escalate to human

**Re-plan decision logic:**
```
on evaluation_signal(task_id, verdict):
  if verdict = "fail" and task.retry_count < task.max_retries:
    → increment retry_count; reset to "pending"; same agent or re-route
  if verdict = "fail" and retry_count >= max_retries:
    → re-decompose: replace failed task with a sub-DAG of smaller tasks
  if re-decomposition produces no viable graph:
    → escalate to human approval gate
```

**Known task types**:
- `summarize` — summarize a document, email thread, or dataset
- `build` — construct a system, feature, or integration
- `create-asset` — produce a deliverable (report, PPT, code module)
- `research` — gather and structure information
- `review` — evaluate an artifact for correctness, quality, or security

### 4.2 Task Platform — The Company (Paperclip layer)

**Responsibility**: Act as the company. Post Job Descriptions, qualify agents, coach execution, enforce security controls, and close tasks only after evaluation.

**Job Description (JD) model**: Every ticket is a JD, not just a task description. It must declare:
- `required_capabilities` — what skills/tools the agent must possess (e.g., `code-execution`, `web-search`, `gpu-inference`)
- `qualification_method` — how capability is verified: `unit_test`, `reputation_score`, `config_match`, or `manual`
- `acceptance_criteria` — explicit, testable definition of done
- `security_constraints` — privilege level, allowed external sources, output sanitization rules
- `budget_ceiling` — max token spend in cents
- `coaching_context` — guidance, policies, and guardrails provided to the agent at handoff

**Coaching**: Before handing off to an execution agent, the platform injects `coaching_context` — task-specific policies, domain guidelines, and security rules. The agent is not left to interpret the task cold.

**Security control during execution**: The platform monitors heartbeats and can intervene: pause, redirect, or terminate an agent that exceeds budget, violates privilege constraints, or goes silent.

**Core API contract** (from Paperclip architecture):
- `POST /tasks/checkout` — atomic lock after qualification gate passes; only one qualified agent wins
- `PATCH /tasks/:id` — update status, append output
- `POST /tasks/:id/subtasks` — create child tasks
- `GET /agents/me` — agent identity, assigned task, coaching context
- All actions append to immutable `activity_log`

**Invariants**:
- Every task has a `goal_id` linking it to a company/mission goal
- No agent may check out a task without passing the qualification gate
- No task may be marked `completed` without a passing evaluation verdict (SoD gate)
- Budget exhaustion transitions task to `blocked`, not `failed`

### 4.3 Execution Agent (OpenShell / OpenClaw)

**Responsibility**: Execute a single checked-out task by orchestrating Reader → Actor → Sub-agents.

**SoD within execution** (prompt injection isolation):

| Sub-role | Access | Input | Output |
|----------|--------|-------|--------|
| **Reader Agent** | Read-only; no tool calls that mutate state | Raw, untrusted external content (files, APIs, web) | Structured JSON summary |
| **Actor Agent** | Privileged; can call tools, write outputs | Validated JSON from Reader only | Task result artifact |

**Sub-agent dispatch**:
- Agent-A, Agent-B, Agent-C are domain-specialized workers (e.g., code, data, comms)
- Selection is deterministic based on `task.type` and `task.tags`
- Routing table is defined in §7

**Constraints**:
- Actor never receives raw external content
- Execution agent cannot approve its own output
- Must tag output with `@executedBy <model-id> <framework-version>` and `@executedAt <timestamp>`

### 4.4 Evaluation Agent (Independent Reviewer)

**Responsibility**: Assess execution output against acceptance criteria, SoD compliance, and security properties.

**Independence requirements** (from Girard SoD paper):
- Must come from a **different model vendor/lineage** than the Execution agent
- Has **read-only** access to task output; cannot mutate any artifact
- Operates in **blind review** mode: does not receive the original planner intent or generator rationale
- Receives only: output artifact, acceptance criteria, deployment context, explicit threat assumptions

**Verdict schema**:
```json
{
  "task_id": "...",
  "verdict": "pass | fail | escalate",
  "confidence": 0.0–1.0,
  "findings": [
    {
      "severity": "critical | high | medium | low | info",
      "claim": "...",
      "exploit_path": "...",
      "preconditions": "...",
      "recommended_fix": "..."
    }
  ],
  "reviewed_by": "<model-id>",
  "reviewed_at": "<ISO-8601>",
  "generator_origin": "<from task tag>"
}
```

**Escalation triggers**:
- `verdict = escalate` → human approval gate opens in Paperclip
- `confidence < 0.6` → always escalate regardless of verdict
- Generator and reviewer model IDs match → SoD violation, hard block

---

### 4.5 Agent Registry and Qualification

Every agent that works in the NEMO system must be registered and qualified before it can take tickets. This is the hiring process.

**Registration record**:

| Field | Description |
|-------|-------------|
| `agent_id` | Unique identifier |
| `capabilities` | Declared skill tags (e.g., `code-execution`, `web-search`, `gpu-inference`, `summarization`) |
| `runtime` | Execution environment (OpenShell, NemoClaw, Claude Code, Gemini, HTTP, etc.) |
| `qualification_status` | `pending`, `qualified`, `probation`, `disqualified` |
| `reputation_score` | Rolling score 0.0–1.0 updated after each evaluation |
| `unit_test_results` | Pass/fail records per capability claim |
| `last_evaluated_at` | ISO-8601 timestamp |
| `cost_profile` | Average token spend per task type |

**Qualification methods** (any one is sufficient per capability):

1. **Unit test** — the platform runs a known task with a known correct answer and scores the result
2. **Reputation score** — rolling average of past evaluation verdicts; agents start on probation until score ≥ 0.7
3. **Config match** — declarative check: agent's registered tools/permissions match the JD's `required_capabilities`
4. **Manual** — human explicitly approves the agent for a task class (used for novel or high-risk work)

**Qualification gate logic**:
```
for each required_capability in JD.required_capabilities:
    if agent does not satisfy capability via any qualification method:
        → reject checkout; agent stays in queue
if all capabilities satisfied:
    → grant checkout; inject coaching_context
```

**Reputation update after evaluation**:
- Performance verdict `pass` → +0.05 to reputation
- Performance verdict `fail` → -0.10 to reputation
- Security violation → -0.20 to reputation; triggers review
- Reputation < 0.4 → agent moved to `probation`; requires manual re-qualification

---

## 5. Task Decomposition Rules (Planner Checklist)

Before dispatching any sub-task, the Planner must confirm:

**Correctness**
- [ ] Task has a single, unambiguous acceptance criterion
- [ ] Task is assigned to exactly one agent type (`summarize`, `build`, `create-asset`, `research`, `review`)
- [ ] Task has a budget ceiling in token-cents
- [ ] Task has a `goal_id` traceable to the company mission

**DAG integrity**
- [ ] All `dependencies` are listed explicitly (not inferred at runtime)
- [ ] The dependency graph has no cycles (Planner verifies before write)
- [ ] Agent count is justified in `planning_rationale` — no fixed N
- [ ] `max_retries` is set (default: 2; high-risk tasks: 1 with mandatory re-plan)

**Schema contracts**
- [ ] `input_schema` declares all fields this task will consume from shared workspace or coaching context
- [ ] `output_schema` declares the required shape of the output artifact
- [ ] `workspace_keys` and `workspace_writes` are declared (no undeclared shared state)

**Security**
- [ ] If task touches external data: a Reader sub-step is planned before any Actor step
- [ ] If task produces a code artifact: an independent Evaluation step is in the plan
- [ ] No single agent is planned as both executor and evaluator of the same artifact
- [ ] High-risk tasks (risk_level = high) have a human approval gate in the DAG

---

## 6. Routing Logic — "Finding the Right Claw"

The routing decision maps `(task.type, task.domain, task.risk_level)` → `execution_agent`:

| Task Type | Domain | Risk | Assigned Claw |
|-----------|--------|------|---------------|
| `summarize` | any | low | Agent-A (Reader-only, no Actor needed) |
| `build` | code | medium | Agent-B (OpenClaw with full Reader→Actor pipeline) |
| `build` | infra | high | Agent-B + human approval gate |
| `create-asset` | document/ppt | low | Agent-C |
| `create-asset` | code | medium | Agent-B |
| `research` | any | low | Agent-A |
| `review` | security | high | Evaluation Agent (independent lineage required) |
| `review` | code quality | medium | Evaluation Agent |

**Risk escalation**: any task with external data sources automatically upgrades risk by one level.

---

## 7. Security Model

### 7.1 Agent Privilege Separation

Based on Tsao & Cheng's "Agent Privilege Separation: A Structural Defense Against Prompt Injection" (arXiv:2603.13424v1), this pattern is a **general architectural principle** — not tied to any single runtime. It applies equally inside OpenShell, NemoClaw, Claude Code, or any multi-agent execution layer.

**Core principle**: Every agent that touches external content must be structurally split into two roles with different privilege levels:

```
[Untrusted External Source]
         │
         ▼
  ┌─────────────────┐
  │  Unprivileged   │  Read-only. No tool calls that mutate state.
  │  Reader Agent   │  Sees raw content. Produces structured summary.
  └────────┬────────┘
           │  structured JSON only
           ▼
  ┌─────────────────┐
  │  Privileged     │  Can call tools, write outputs, execute actions.
  │  Actor Agent    │  Never sees raw external content.
  └─────────────────┘
```

**Why this generalizes beyond OpenClaw**:
- Any agent framework where an agent reads from untrusted sources (emails, web, files, APIs, user uploads) and then takes actions is vulnerable to prompt injection
- The structural fix is always the same: separate read from act at the privilege boundary
- In NEMO, this pattern is enforced at the platform level — the coaching context the platform injects explicitly forbids Actors from accepting raw strings; all external content must arrive as validated JSON from a Reader step
- NemoClaw implements this natively; for other runtimes (Claude Code, Gemini, HTTP agents), the platform enforces the boundary via input schema validation before the Actor receives its context

### 7.2 SoD Enforcement

Three roles are architecturally separated:

| Role | Model Constraint | Access |
|------|-----------------|--------|
| Generator / Execution Agent | Any approved model | Write to task output, read task definition |
| Security Reviewer / Evaluation Agent | **Different vendor lineage from generator** | Read-only on outputs |
| Approver / Gate | Rules engine + optional human | Block, escalate, log; cannot execute |

Platform-level enforcement:
- Task cannot transition to `completed` unless `evaluation.verdict = pass` from a non-generator agent
- Generator model ID and reviewer model ID are logged and compared at gate; match → hard block
- Audit log is immutable; append-only

### 7.3 Generator-Origin Tagging

All agent outputs carry machine-readable provenance tags:
```
# @generatedBy <model-id>/<version>
# @generationTime <ISO-8601>
# @taskId <task-uuid>
# @reviewedBy <model-id>/<version> <verdict>
```

These tags enable:
- Auditors to reconstruct which model produced each artifact
- Automated detection of same-model generation+review loops
- Drift monitoring when model versions change

---

## 8. Evaluation Framework

Every completed task is independently evaluated on three pillars before the ticket closes. Evaluation agents must be from a different model lineage than the execution agent (Girard SoD). Verdicts feed back into agent reputation scores in the registry.

---

### 8.1 Performance Evaluation

**Question**: Did the agent do the job correctly and efficiently?

| Metric | Description | Target |
|--------|-------------|--------|
| `task_completion_rate` | Output satisfies all acceptance criteria | 100% |
| `first_pass_rate` | Tasks passing evaluation without rework | > 85% |
| `output_quality_score` | Rubric-based score against acceptance criteria | ≥ 0.8 |
| `latency_vs_baseline` | Time taken vs. established agent baseline | ≤ 1.2× |
| `subtask_success_rate` | % of sub-tasks completed without escalation | > 90% |

---

### 8.2 Security Evaluation

**Question**: Did the agent stay within its privilege boundary and produce safe outputs?

| Check | Description | Failure Action |
|-------|-------------|----------------|
| **Privilege adherence** | Actor never received raw external content | Hard fail; security event logged |
| **SoD compliance** | Execution agent ≠ evaluation agent | Hard block if same lineage |
| **Prompt injection detection** | Scan output for signs of injected instruction leakage | Escalate to human |
| **Provenance completeness** | All outputs tagged `@generatedBy`, `@reviewedBy` | Fail if missing |
| **Coaching compliance** | Agent respected injected constraints | Log violation; reputation penalty |
| **Output sanitization** | No credential, PII, or secret leakage in artifact | Hard fail |

For high-risk tasks, a multi-tier review applies:
```
Tier 0: Deterministic scan (secrets, SAST, dangerous API usage — not LLM-based)
Tier 1: Independent LLM reviewer (different vendor from executor)
Tier 2: Adversarial LLM reviewer (different from both executor and Tier-1)
Tier 3: Adjudication gate — deduplicates findings, routes disagreements to human
```

---

### 8.3 Cost Evaluation

**Question**: Was the agent economically efficient? Is this agent worth rehiring?

| Metric | Description | Target |
|--------|-------------|--------|
| `spend_vs_budget` | Actual token spend vs. declared budget ceiling | ≤ 100% |
| `cost_per_task_type` | Average spend by task category | Tracked; used for future budgeting |
| `budget_variance` | Deviation from estimate; high variance → inaccurate self-reporting | < ±20% |
| `cost_efficiency_ratio` | Output quality score / token spend | Tracked per agent for comparison |
| `roi_estimate` | Value of output vs. cost (human-evaluated for strategic tasks) | Qualitative |

Cost evaluation results update the agent's `cost_profile` in the registry and inform future JD budget ceilings.

---

### 8.4 Aggregate Verdict and Reputation Update

All three pillar scores are combined into a single task verdict:

```
verdict = "pass"      if performance ≥ 0.8 AND security = clean AND spend ≤ budget
verdict = "fail"      if performance < 0.6 OR any security hard-fail
verdict = "escalate"  otherwise (human reviews before close)
```

The verdict feeds directly into the Agent Registry (§4.5) to update reputation score, cost profile, and qualification status.

---

## 9. Heartbeat / Long-Running Task Protocol

NEMO operates on a **Plan → Execute → Evaluate → Refine** loop, not a one-shot batch. Execution agents use a heartbeat cycle; evaluation failures feed back to the Planner for re-planning.

### 9.1 Execution Agent Heartbeat

1. **Checkout** — atomically acquire one pending task (dependency gate must be satisfied)
2. **Acknowledge** — post `in_progress` status with agent identity; load `coaching_context`
3. **Load context** — pull shared workspace memory (Tier-2) for upstream task outputs relevant to this task's `input_schema`
4. **Execute** — run Reader → Actor → Sub-agent chain; write intermediate state to Tier-1 (task-local)
5. **Checkpoint** — write validated intermediate results to Tier-2 (shared workspace); allow recovery on failure
6. **Complete** — post output artifact with provenance tags; validate against task's `output_schema`
7. **Yield** — release lock; Evaluation Agent picks up

Budget exhaustion at any step → transition to `blocked`, not `failed`. Human can increase budget and resume.

### 9.2 Iterative Refinement Loop

After evaluation, the loop does not end — it feeds back:

```
Plan (Planner builds DAG)
  │
  ▼
Execute (Execution Agent — heartbeat above)
  │
  ▼
Evaluate (Performance + Security + Cost — §8)
  │
  ├── verdict = pass ──────────────────────► Close task; update Agent Registry
  │
  ├── verdict = escalate ──────────────────► Human approval gate; Planner waits
  │
  └── verdict = fail
        │
        ├── retry_count < max_retries ──────► Re-queue same task; increment retry
        │
        └── retry_count >= max_retries ─────► Re-plan signal to Planner
                                                │
                                                ▼
                                        Planner re-decomposes subgraph
                                        (smaller tasks, different agent, narrower scope)
                                                │
                                                ▼
                                        Loop continues
```

**Termination conditions:**
- All tasks in the DAG reach `verdict = pass` → graph closes; root task completes
- Re-decomposition produces no viable graph → escalate to human
- Graph-level budget ceiling hit → pause entire graph; human reviews

---

## 10. Data Model

### Task object
```typescript
interface Task {
  id: string;
  goal_id: string;
  title: string;
  description: string;
  type: "summarize" | "build" | "create-asset" | "research" | "review";
  domain: string;
  risk_level: "low" | "medium" | "high";
  status: "pending" | "in_progress" | "blocked" | "awaiting_review" | "completed" | "failed";
  assigned_agent_id: string | null;
  budget_cents: number;
  spent_cents: number;
  acceptance_criteria: string;

  // DAG fields
  dependencies: string[];            // task IDs that must reach verdict=pass before this task is released
  parent_task_id: string | null;     // for Paperclip-compatible subtask hierarchy
  planning_rationale: string;        // Planner's justification for this task's scope and agent count

  // Schema contracts (JSON Schema or Zod-compatible)
  input_schema: object;              // expected shape of inputs consumed (from shared workspace or coaching context)
  output_schema: object;             // required shape of the output artifact; validated at Complete step

  // Retry and re-plan
  retry_count: number;               // how many times this task has been re-queued after fail
  max_retries: number;               // if exceeded, triggers Planner re-decomposition instead of retry

  // Memory
  workspace_keys: string[];          // keys this task reads from Tier-2 shared workspace
  workspace_writes: string[];        // keys this task writes to Tier-2 on completion

  output_artifact_url: string | null;
  provenance_tags: ProvenanceTags;
  evaluation: EvaluationVerdict | null;
  created_at: string;
  updated_at: string;
}

interface ProvenanceTags {
  generated_by: string;      // model-id/version
  generated_at: string;      // ISO-8601
  reviewed_by: string | null;
  reviewed_at: string | null;
  verdict: "pass" | "fail" | "escalate" | null;
}
```

---

## 11. Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Task checkout atomicity | No two agents may hold the same task simultaneously |
| Audit log immutability | Append-only; no deletes; tamper-evident |
| SoD hard enforcement | Zero tolerance: generator = reviewer → automatic block |
| Escalation latency | Human approval gate notified within 60 seconds of escalation |
| Budget accuracy | Spend tracking to within ±1% of actual token cost |
| Provenance completeness | 100% of completed tasks have `@generatedBy` and `@reviewedBy` tags |

---

## 12. References

1. Tsao, W-K. & Cheng, D. — "Agent Privilege Separation in OpenClaw: A Structural Defense Against Prompt Injection" — arXiv:2603.13424v1
2. Anthropic Engineering — "Harness Design for Long-Running Apps" — anthropic.com/engineering/harness-design-long-running-apps
3. Girard, D. — "Separation-of-Duties-First AI Code Security" — TrendAI Security for AI, March 2026
4. Paperclip — Multi-agent orchestration control plane — github.com/paperclipai/paperclip
