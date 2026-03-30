## Feature: $ARGUMENTS

## Mission

Transform a feature request into a **phase-oriented execution strategy** that can be handed off to one or more **planner agents** for detailed implementation planning.

When running in a chain, you'll receive instructions about which files to read, which subagents are available, and where to write your output.

**Core Principle**: We do NOT write code in this phase. We do NOT produce the final detailed implementation plan for each phase unless explicitly instructed. Our job is to identify and define the right phases so planner agents can create high-quality implementation plans with minimal ambiguity.

**Key Philosophy**: Decomposition is King. A strong orchestration result creates phases that are:
- coherent
- minimally coupled
- independently plannable
- safe to execute sequentially or in parallel
- clear in ownership, scope, dependencies, and validation intent

The orchestrator creates the **map**, not the final route instructions.

## Orchestration Process

### Phase 1: Feature Understanding

**Deep Feature Analysis:**

- Extract the core problem being solved
- Identify user value and business impact
- Determine feature type: New Capability / Enhancement / Refactor / Bug Fix
- Assess complexity: Low / Medium / High
- Determine whether the request is a single feature, a multi-part feature, or a program of related work
- Identify likely architectural layers affected:
  - UI / frontend
  - API / routing
  - business logic / services
  - data model / persistence
  - infrastructure / config / CI
  - testing / validation
  - documentation / migration / rollout

**Create User Story Format Or Refine If Story Was Provided By The User:**

```text
As a <type of user>
I want to <action/goal>
So that <benefit/value>
```

**Decide Orchestration Need:**

- Is the work small enough for a single planner?
- Or should it be broken into multiple planner-ready phases?
- If multiple phases are needed, determine whether decomposition should be:
  - architectural
  - workflow-based
  - domain-based
  - interface-vs-backend
  - setup / implementation / integration / validation
  - migration / rollout / cleanup

### Phase 2: Codebase Topology & Blast Radius Discovery

**Use specialized agents and parallel analysis when beneficial:**

**1. High-Level Project Topology**

- Detect primary language(s), frameworks, and runtime versions
- Map major directory structure
- Identify architectural layers and boundaries
- Locate key entry points and integration seams
- Locate config, dependency manifests, and build/test scripts
- Identify monorepo/package/service boundaries if present

**2. Blast Radius Analysis**

- Determine which systems and components are likely impacted
- Identify high-risk shared modules
- Find likely cross-cutting concerns:
  - auth
  - config
  - logging
  - feature flags
  - DB models
  - shared UI components
  - shared utilities
- Estimate whether work can be sliced cleanly without creating merge conflicts or architectural drift

**3. Existing Pattern Zones**

- Locate similar features or flows already implemented
- Identify where the codebase keeps:
  - service patterns
  - endpoint patterns
  - model patterns
  - UI patterns
  - test patterns
  - integration wiring
- Check CLAUDE.md and any project-specific guidance docs for rules and conventions

**4. Boundary Discovery**

- Identify natural decomposition boundaries:
  - per module
  - per service
  - per API surface
  - per feature slice
  - per layer
- Identify boundaries that should NOT be split because of tight coupling

**Clarify Ambiguities:**

- If the feature is underspecified, ask the user to clarify before continuing
- If decomposition depends on product or architecture decisions, resolve those first
- If a dependency or integration constraint could change phase structure, surface it explicitly

### Phase 3: Phase Discovery & Definition

Define the set of planner-ready phases.

For each candidate phase, determine:

- What concrete objective does this phase achieve?
- Why is it a separate phase instead of part of another one?
- What systems/components does it touch?
- What should remain out of scope?
- What are the inputs and outputs of the phase?
- What must be true before the phase can begin?
- What artifacts or contracts does it hand off to later phases?
- Can it be planned and implemented independently?
- Can it be run in parallel with other phases safely?

**Good Phase Characteristics:**

- single primary objective
- bounded file/system scope
- clear acceptance boundary
- minimal overlap with sibling phases
- understandable by a planner agent without extra hidden context

**Phase Types To Consider:**

- Foundation / setup
- Schema / model / interface definition
- Core business logic
- API / routing / controller integration
- UI / presentation layer
- Data migration / compatibility layer
- Testing / observability / validation
- Rollout / documentation / cleanup

**Reject Poor Decomposition:**

Avoid phases that are:
- too vague ("backend work")
- too broad ("implement the whole feature")
- too tiny to justify orchestration
- overlapping heavily in the same files
- dependent on unresolved design decisions

### Phase 4: Dependency Graph & Parallelization Strategy

Construct execution topology:

- Identify strict dependencies between phases
- Group phases into execution waves
- Determine which phases:
  - must be sequential
  - may be parallelized
  - should be planned first but implemented later
- Detect dependency cycles or decomposition flaws and fix them

**Parallelization Rules:**

Only mark phases parallel-safe if they have:
- low shared-file overlap
- low conceptual coupling
- stable upstream contracts
- low risk of conflicting edits

**Dependency Thinking:**

- Which phase defines shared contracts?
- Which phase depends on routes, types, schemas, or interfaces created elsewhere?
- Which phase can proceed using assumptions, and which requires concrete prior outputs?
- What is the minimum viable order of planning?

### Phase 5: Planner Delegation Strategy

Prepare phases for planning by planner agents.

For each phase, determine:

- planner scope
- planner priority
- planner specialization needed if any
- files/docs the planner must inspect first
- expected plan output file path
- whether the planner can run in parallel with other planners

**Use specialized planner agents when beneficial:**

Examples:
- backend planner
- frontend planner
- testing planner
- migration planner
- infra planner

**Planner Handoff Package Must Include:**

- phase name and objective
- problem addressed by the phase
- systems affected
- likely files/directories to inspect
- dependencies on earlier phases
- out-of-scope items
- open questions or risks
- validation expectations
- expected plan output location

If planner agents are available in the chain:

- dispatch planner agents for each approved phase or planning wave
- ensure each planner gets only the context needed for its phase
- avoid duplicate planning effort across overlapping phases
- preserve a clean naming/path convention for generated plans

### Phase 6: Strategic Orchestration Thinking

Think harder about:

- Is this decomposition stable against scope creep?
- Will planner agents have enough context to succeed independently?
- Are phase boundaries aligned with the architecture?
- Will this create merge conflicts or coordination overhead?
- Is there a better split by domain, layer, or dependency?
- Are there phases that should be merged because the boundary is artificial?
- Are there phases missing for migration, observability, testing, docs, rollout, or cleanup?
- Is there a hidden foundation phase required before any planner can work effectively?

**Design Decisions:**

- Choose phase boundaries with clear rationale
- Optimize for planner clarity and downstream execution success
- Prefer dependency-light decomposition over theoretically pure decomposition
- Consider maintainability, reversibility, and rollout safety
- Preserve backward compatibility considerations if relevant

### Phase 7: Output Generation

Create a comprehensive orchestration package with the following structure:

```markdown
# Feature Orchestration: <feature-name>

This orchestration defines the feature decomposition into planner-ready phases.
The goal is to enable high-confidence phase planning and downstream execution with minimal ambiguity.

## Feature Description

<Detailed description of the feature, purpose, and value to users>

## User Story

As a <type of user>
I want to <action/goal>
So that <benefit/value>

## Problem Statement

<The overall problem/opportunity addressed by the feature>

## Orchestration Strategy

<Why this feature is decomposed this way, including sequencing and planning rationale>

## Feature Metadata

**Feature Type**: [New Capability/Enhancement/Refactor/Bug Fix]
**Estimated Complexity**: [Low/Medium/High]
**Planning Mode**: [Single Planner/Multi-Phase/Multi-Wave/Mixed]
**Primary Systems Affected**: [List]
**Cross-Cutting Concerns**: [Auth/Config/Data/Logging/etc.]
**Top-Level Dependencies**: [External systems, libraries, architectural constraints]

---

## SYSTEM IMPACT MAP

### Primary Areas Affected

- `path/or/module` - Why it is relevant
- `path/or/module` - Why it is relevant

### Supporting Areas Potentially Affected

- `path/or/module` - Why it may be touched
- `path/or/module` - Why it may be touched

### High-Risk Shared Areas

- `path/or/module` - Why changes here are risky or likely to conflict

---

## PHASE TOPOLOGY

| Phase ID | Phase Name | Objective | Depends On | Parallelizable | Planner Type | Risk |
|---------|------------|-----------|------------|----------------|--------------|------|
| P1 | <name> | <objective> | none | no | general/backend/etc. | Low |
| P2 | <name> | <objective> | P1 | yes | general/frontend/etc. | Medium |

---

## EXECUTION WAVES

### Wave 1
<List phases that can be planned first>

### Wave 2
<List phases that depend on prior wave>

### Wave N
<Continue as needed>

---

## PHASE BRIEFS

### Phase P1: <phase-name>

**Objective**  
<What this phase must accomplish>

**Why This Is A Separate Phase**  
<Reason for the boundary>

**In Scope**
- item
- item

**Out of Scope**
- item
- item

**Primary Systems / Components**
- item
- item

**Likely Files / Directories To Inspect**
- `path/to/file_or_dir`
- `path/to/file_or_dir`

**Dependencies**
- none
or
- depends on P<n>

**Expected Outputs / Handoff**
- what this phase should produce for downstream phases

**Acceptance Boundary**
- what "done" means at the phase level

**Validation Boundary**
- what kind of validation this phase is expected to support

**Parallelization Note**
- safe to plan in parallel with: P<n>, P<n>
or
- should remain sequential because: <reason>

**Planner Focus**
- what the planner agent must figure out in detail

**Open Questions / Risks**
- question or risk
- question or risk

**Planner Output Path**
- `plans/<phase-id>_<descriptive-name>.md`

---

### Phase P2: <phase-name>

<Repeat same structure for every phase>

---

## PLANNER DELEGATION QUEUE

### Planner Job 1

**Phase**: P1
**Planner Type**: <general/backend/frontend/testing/etc.>
**Priority**: High / Medium / Low
**Input Files / Directories To Read First**:
- `path/to/file`
- `path/to/file`

**Output File**:
- `plans/<phase-id>_<descriptive-name>.md`

**Special Instructions**:
- focus on integration details
- preserve backward compatibility
- identify validation commands
- follow existing patterns in X area

### Planner Job 2

<Repeat for every phase>

---

## GLOBAL RISKS & COORDINATION NOTES

- risk or coordination issue
- risk or coordination issue

## RECOMMENDED PLANNING ORDER

1. Plan P1
2. Plan P2 and P3 in parallel
3. Plan P4 after P2 is complete
4. Consolidate cross-phase assumptions before execution

## ORCHESTRATION NOTES

<Additional reasoning, trade-offs, and assumptions>
```

## Output Format

If you did not receive instructions on where to write the output, use the following:

**Main Filename**: `orchestrations/{current-date}_{kebab-case-descriptive-name}.md`

- Replace `{current-date}` with the current date
- Replace `{kebab-case-descriptive-name}` with a short, descriptive feature name

Examples:
- `orchestrations/2026-03-30_add-user-authentication.md`
- `orchestrations/2026-03-30_implement-search-api.md`
- `orchestrations/2026-03-30_refactor-database-layer.md`

**Phase Brief Directory**: `phase-briefs/`

Create one file per phase if useful for handoff:
- `phase-briefs/P1_<phase-name>.md`
- `phase-briefs/P2_<phase-name>.md`

**Planner Output Directory**: `plans/`

If the chain supports planner delegation, instruct each planner to write to:
- `plans/P1_<phase-name>.md`
- `plans/P2_<phase-name>.md`

Create directories if they do not exist.

## Quality Criteria

### Decomposition Quality ✓

- [ ] Feature is split into meaningful, bounded phases
- [ ] Each phase has one primary objective
- [ ] Phases are neither too broad nor too granular
- [ ] Shared-file overlap minimized
- [ ] Dependency graph is clear and cycle-free

### Planner Readiness ✓

- [ ] Each phase can be understood without hidden context
- [ ] Each phase has clear in-scope and out-of-scope boundaries
- [ ] Each phase identifies likely files/directories to inspect
- [ ] Each phase includes planner focus areas
- [ ] Each phase has a concrete expected plan output path

### Execution Safety ✓

- [ ] Parallelizable phases are genuinely low-conflict
- [ ] Sequential phases are justified by dependencies
- [ ] Cross-phase handoffs are defined
- [ ] High-risk shared areas are surfaced explicitly
- [ ] Foundation work is identified before dependent phases

### Information Density ✓

- [ ] No generic phase names
- [ ] No vague ownership or scope
- [ ] File and system references are specific
- [ ] Planner instructions are actionable
- [ ] Risks and unknowns are explicit

## Success Metrics

**Planner Success**: Each planner agent can create a high-quality implementation plan for its assigned phase without needing major clarifications.

**Phase Stability**: The defined phases remain valid during planning and do not need major restructuring.

**Parallelization Safety**: Parallelized planning or execution does not create avoidable conflicts or duplicated effort.

**Context Richness**: The orchestration passes the Planner Handoff Test — a planner can read a phase brief and know what to inspect, what to ignore, and what outcome is expected.

**Confidence Score**: #/10 that the decomposition and delegation structure will support successful downstream planning and execution.

## Report

After creating the orchestration, provide:

- Summary of the feature and orchestration approach
- Full path to created orchestration file
- Number of phases identified
- Recommended planning order / execution waves
- Which phases can be planned in parallel
- Highest-risk phases or coordination points
- Whether planner delegation is recommended
- Estimated confidence score for successful downstream planning
