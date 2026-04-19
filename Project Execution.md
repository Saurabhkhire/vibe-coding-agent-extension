# Project Execution

This is the **only** control document for this folder. Behavior depends on the command you give:

| Command intent | Required token in your prompt | What happens |
|----------------|-------------------------------|--------------|
| **Plan** (documentation only) | **`#plan`** exactly (hash + `plan`, no spaces) | Generate or update Phase markdown **inside each Phase file’s `Changes` section**, Phase-by-phase (1→10). No application code. |
| **Execute** (implementation) | **`execute`** exactly (this word alone as the keyword; see paste prompts below) | **Merge** each Phase’s **`Changes` → `Current`**, then write or update **code** by reading **`Current`** across **all** Phases—layered batches, not one unstructured dump. |

Do **not** trigger these modes with loose wording (e.g. “planning” or “Planner”). Only **`#plan`** enables documentation mode; only a prompt that includes **`execute`** as specified enables implementation mode.

If both **`#plan`** and **`execute`** appear, treat that as **`execute`** only after docs are merged (resolve conflicts manually if needed). If neither appears, ask which mode you intend.

---

## Folder layout (`Phases`)

Everything lives at the repository root:

| Item | Purpose |
|------|---------|
| **`Project Execution.md`** | Main control document (this file). |
| **`Phases/Phase-01-*.md` … `Phases/Phase-10-*.md`** | Living Phase specs. Each uses **`Current`** vs **`Changes`** (see below). |

The **same Phase format** applies to every `Phase-NN-*.md`; what differs per run is what you put under **`Current`** vs **`Changes`**.

### Inside every Phase document

| Section | Role |
|---------|------|
| **`Current`** | Approved baseline—the spec you treat as truth **after** a merge. **Implementation reads this.** |
| **`Changes`** | Staging area filled by the latest run that included **`#plan`**. Use **blockquotes** (`>` lines) so updates read as **highlighted** in many Markdown previews. |

---

## Mode: `#plan` (documentation agent)

1. Read **this file** and the user’s goal or change description. **Only proceed if the user’s message contains `#plan`.**
2. Work **Phase order** (1 → 10) unless the user names a subset.
3. For each Phase in scope: read **`Current`** (and earlier Phases' **`Current`**) for context; write **only into `Changes`** for that Phase, preserving the Phase section headings inside the staged content as needed.
4. Do **not** overwrite **`Current`** during **`#plan`** unless the user explicitly says to promote without **`execute`**.
5. End each Phase step with a short summary and note downstream Phases that may need updates (see table below).

**Paste prompt**

```text
#plan — Read Project Execution.md.
Goal: [WHAT YOU ARE BUILDING OR CHANGING]
Update Phase docs under Phases/: for each touched Phase, fill or refresh the Changes section only (highlight with Markdown blockquotes). Go Phase 1 through Phase 10 in order [or Phases: N–M].
```

---

## Mode: `execute` (code agent)

1. Read **this file** once. **Only proceed if the user’s message includes the token `execute` as the implementation trigger** (see paste prompt; use the word `execute` literally there).
2. **Merge before coding:** For each `Phases/Phase-NN-*.md`, if **`Changes`** is non-empty, **promote** it into **`Current`**:
   - Default rule: replace the body under **`## Current`** with the finalized spec (integrate **Changes** into **Current**; resolve conflicts so **Current** is self-contained).
   - Then clear or reset **`Changes`** to a single highlighted line such as `> *(None — merged on execute.)*`.
3. Read **`Current`** from **all** Phases **in order** (1→10). Empty or “N/A” sections: note gaps before inventing behavior.
4. **Implement in batches** (schema/models → APIs → LLM/agents → tech-flow files → tests). Map files and public APIs to Phase sections.
5. Prefer updating **code** to match **`Current`**; if the spec is wrong, fix docs in a later run that includes **`#plan`**.

**Paste prompt**

```text
execute — Read Project Execution.md.
Merge Changes into Current for each Phase file, then implement from Current across all Phases.
Scope: [WHAT TO BUILD OR CHANGE IN CODE]
Implement in layered steps; cite which Phase Current section drove each artifact.
```

---

## Why order matters

Phases are **dependent**. If **`Current`** (or staged **`Changes`**) updates an earlier Phase, **revisit** later Phases so they stay consistent.

| If you change… | Typically revisit… |
|----------------|-------------------|
| 1 — Use case / tech | 2–10 |
| 2 — Architecture | 3–10 |
| 3 — Workflow | 4–10 |
| 4 — Database | 5–10 |
| 5 — Getter/setters | 6–10 |
| 6 — APIs | 7–10 |
| 7 — LLMs | 8–10 |
| 8 — Agents | 9–10 |
| 9 — Tech flow | 10 |
| 10 — Tests | — |

---

## Phase index

| # | Phase file | Purpose | Primary inputs from earlier Phases |
|---|------------|---------|-----------------------------------|
| 1 | `Phases/Phase-01-use-case-and-technologies.md` | Problem, users, scope; tech stack by type | — |
| 2 | `Phases/Phase-02-architecture.md` | Components, boundaries, deployment view | 1 |
| 3 | `Phases/Phase-03-logical-workflow.md` | Flows, steps, diagrams | 1, 2 |
| 4 | `Phases/Phase-04-database-structure.md` | Tables, columns, types, dependency diagram | 2, 3 |
| 5 | `Phases/Phase-05-getter-setters-pojos.md` | Models/DTOs | 4 (and 6 when known) |
| 6 | `Phases/Phase-06-apis-and-flow.md` | APIs, inputs/outputs, diagrams | 3, 4, 5 |
| 7 | `Phases/Phase-07-llms-prompts.md` | LLM prompts; inputs/outputs | 3, 6 |
| 8 | `Phases/Phase-08-agent-structure.md` | Agents, diagrams | 2, 3, 6, 7 |
| 9 | `Phases/Phase-09-tech-flow.md` | Files; methods; `#` one-liners | 2–8, codebase |
| 10 | `Phases/Phase-10-test-cases.md` | Positive/negative; API & UI tests | All |
