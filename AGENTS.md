===============================================================================
AGENTS.md — AI Agent Guidance for Atomikos Repos (Durable Edition)
===============================================================================

This file provides global guidance for AI agents (Codex, Claude Code, etc.) in
Atomikos codebases. It complements — and never replaces — official documentation
or Atomikos support channels.

The goal is safe velocity:
- speed up understanding of the codebase
- preserve architectural intent
- reduce merge risk and regressions
- prevent "invisible" weakening of correctness and fallback paths

-------------------------------------------------------------------------------
0) Definitions
-------------------------------------------------------------------------------

Architecture is:
- WHAT exists
- WHERE it lives
- WHY it exists

Implementation is:
- HOW it is built

"Available architecture" is:
- Architecture by design (docs) + inferred architecture (from code)

Authority rules:
- ADRs are authoritative and cannot be overridden.
- Architectural docs capture intent but may lag behind code.
- Repo structure / Maven artifact names / source code may be used as
  "inferred architecture" per the exceptions below.
- If design-docs and code conflict:
  - prefer unambiguous code for current behavior
  - record the conflict as "inferred architecture" and note docs appear outdated
  - do not proceed with changes that could weaken invariants without ADR confirmation

-------------------------------------------------------------------------------
0b) Governance hierarchy
-------------------------------------------------------------------------------

This file (AGENTS.md) defines the authoritative AI behavior and architectural
constraints for Atomikos repositories.

Other documents (including `USING_AGENTS.md`)
provide developer-facing usage guidance and operational best practices.

If any conflict exists between this file and other AI-usage guidance,
this file prevails.

Architectural invariants, kernel definitions, canonical source rules,
and STOP-the-line conditions defined here are normative and must not be
overridden by usage documents.

-------------------------------------------------------------------------------
1) Stop-the-line rules (non-negotiable)
-------------------------------------------------------------------------------

The agent must STOP and ask for clarification (or request a maintainer decision)
before proceeding when any of the following apply:

A) Architectural ambiguity:
- `specs/architecture/` is missing in a repo that appears to be development/stable
- ADRs and code appear inconsistent on a core invariant
- the change affects concurrency/locking/FSM transitions without explicit guidance

B) Invariant risk:
- the change could weaken correctness, recovery, or fallback paths
- the change modifies "core surface areas" (see section 2)

C) Merge-risk risk:
- a large edit-in-place is proposed and a safe additive alternative exists
- the change is likely to create downstream merge conflicts in stable lines

D) Red-flag simplification patterns:
- proposals containing terms like "simplify", "cleanup", "remove redundancy",
  "optimize locking", "make async", "eliminate blocking", "refactor concurrency"
- structural reduction of guards, branches, or checks in core surface areas
- lock reordering or synchronization changes

These must be treated as potential CORE-TOUCH changes and evaluated
against documented invariants.

-------------------------------------------------------------------------------
2) Core surface definition (the kernel)
-------------------------------------------------------------------------------

Some areas are "architectural core" and must be treated as kernel code:
changes here require explicit review discipline and failure-mode reasoning.

Core surface includes (non-exhaustive):
- transaction coordination paths (commit, rollback, state transitions)
- recovery mechanisms and durable state for recovery
- timeout semantics that affect correctness and 2PC behavior
- concurrency primitives: locks, synchronization, condition-waits
- FSM transitions, listeners, ordering guarantees
- deadlock prevention logic and pool internals that interact with locks/FSM

Core-touch requirements:
- explicitly label PR/commit as CORE-TOUCH
- preserve all documented invariants and fallback paths
- provide short failure-mode reasoning (see section 6 checklist)
- consult `specs/architecture/playbooks/preventing_deadlocks.md` when relevant

-------------------------------------------------------------------------------
2b) Intentional complexity in the kernel
-------------------------------------------------------------------------------

Certain complexity in the kernel is intentional.

Absence of simplification does not imply accidental design.
Kernel code must not be simplified unless invariant preservation
is explicitly demonstrated.

In particular, complexity related to:
- transaction coordination
- recovery logic
- lock ordering
- FSM transitions
- timeout propagation

must be treated as semantically meaningful, not incidental.

-------------------------------------------------------------------------------
3) Architecture guidance for agents
-------------------------------------------------------------------------------

The agent may answer architectural questions using "available architecture":
- Prefer architecture by design (`specs/architecture/`)
- Add "inferred architecture" from code, clearly labeled as such

For architectural questions (what goes where and why — including which repo
or release line), consult ALL documents under `specs/architecture/` (ADRs,
playbooks, modules, meta, and any other files), not just module docs.

If docs lag code:
- annotate findings as "inferred architecture"
- never reinterpret ADRs
- never weaken or remove fallback paths
- record doc/code conflicts and treat ADRs as authoritative

-------------------------------------------------------------------------------
4) Implementation guidance (what the agent may do)
-------------------------------------------------------------------------------

The agent is responsible for:
- comprehension and explanation of existing code
- implementing an already-decided design
- mechanical edits (renames, reference updates)
- documentation and comments (inline only)
- test assistance

The agent must NOT:
- reinterpret ADR intent
- weaken or remove fallback paths
- "simplify" recovery/correctness mechanisms without explicit design authority
- use non-canonical product names; see ADR-020 (internal) for rationale
 - assume referenced files/folders exist without checking

Code inference guidance:
- If class A is instantiated only inside class B and not exposed outside B,
  then access control and threading guards may be implemented in B rather than A.
- If B owns configuration properties, it can notify A of changes via a listener.
- This is especially relevant if A already depends on B for property values
  (e.g., a pool reading `ConnectionPoolProperties` from its datasource).

-------------------------------------------------------------------------------
5) Documentation location and secrecy
-------------------------------------------------------------------------------

Architecture documentation (`specs/architecture/`) is intentionally kept outside
the Maven build and MUST NOT be published to any Maven repository.

If a change adds architecture documentation to the Maven build:
- WARN and explain this is not intended
- architecture docs are for team members with VCS access
- architectural know-how is a competitive edge at Atomikos

Inline documentation assistance:
- improve doc comments (javadoc) and inline comments for purpose, invariants, role
- prefer clarifying intent over restating implementation details

-------------------------------------------------------------------------------
6) Core-touch checklist (minimum viable rigor)
-------------------------------------------------------------------------------

For changes that touch the core surface (section 2), include:

1) Invariants:
- Which invariant(s) does this preserve? (reference ADR/section if possible)

2) Failure modes:
- What happens on timeouts?
- What happens on partial failure (resource down)?
- What does recovery do?

3) Concurrency:
- What lock ordering / threading assumption does this rely on?
- Are there new waits/locks/listeners?

4) Fallback paths:
- What fallback path exists today?
- Why is it preserved? (or how is it improved, explicitly)

If any item is unclear: STOP and escalate.

-------------------------------------------------------------------------------
7) Repo types and safe change strategy
-------------------------------------------------------------------------------

Repo classification:
- stable: enclosing folder starts with `stable-`, OR `.hgtags` contains tags ending in `.80+`
- development: enclosing folder name is `development`
- open source: not stable, and `specs/architecture/` is missing
- if unclear: STOP and ask before making changes
- if repo appears development/stable but `specs/architecture/` is missing:
  warn on every architecture question, note docs may have moved or been deleted,
  and ask how to proceed (including whether to check VCS history)

Additive-change guidance:
- open source: apply additive-change guidance
- development: apply additive-change guidance
- stable: apply additive-change guidance only when ADRs recommend it

If an edit-in-place is likely to cause merge conflicts and an additive option exists:
- WARN and propose additive option first

When referring to a module, show:
- Maven artifactId
- folder name
(so it can be found either way)

-------------------------------------------------------------------------------
8) Support questions
-------------------------------------------------------------------------------

In development or stable repos:
- answer normally

In open source repos:
- note open source is not supported
- invite users to the free trial and point to:
  https://www.atomikos.com/Main/ExtremeTransactionsFreeTrial

Support response library:
- consult `specs/architecture/playbooks/support_response_library.md` first
- new answers must be recorded as `Status: DRAFT` until a senior marks
  them `Status: CONFIRMED`
- remind the user if relevant entries are still `Status: DRAFT` and ask them
  to get senior confirmation
- when a new reply is added, remind the user to commit the change

-------------------------------------------------------------------------------
9) Commit messages
-------------------------------------------------------------------------------

Atomikos uses conventional commit-style messages:
- `type(scope): summary`
- breaking changes may use `!`
- with a case number: `feat(12345): summary`, `fix(12345): summary`

Examples:
- `feat(12345): add dynamic pool sizing`
- `fix(12345): handle timeout edge case`
- `docs(architecture): add playbook guidance`
- `refactor(transactions-jta): extract wait helper`
- `perf(transactions-jta): reduce lock contention`
- `test(transactions-jta): cover pool resize`

-------------------------------------------------------------------------------
10) Session start guidance for agents
-------------------------------------------------------------------------------

At the start of an interaction:
- state which repo type you are in (open source / development / stable)
- if stable, state the release line if determinable
- if architecture docs are expected but missing: warn and ask how to proceed
- remind other supported stable repos may exist; refer to public terms page
  (Important Dates): https://www.atomikos.com/Main/TermsAndConditions#Appendix_B:_Important_Dates

-------------------------------------------------------------------------------
11) Test mode (optional)
-------------------------------------------------------------------------------

If the user explicitly asks for test mode, provide two answers:
1) inferred architecture only
2) architecture by design (if available)

Use the two answers to compare behavior with vs without `specs/architecture/`.

-------------------------------------------------------------------------------
12) Importance of the public/ folder
-------------------------------------------------------------------------------


The `public/` folder represents the architectural kernel of Atomikos.

These modules define:
- transaction semantics
- recovery behavior
- concurrency guarantees
- correctness invariants

They are open-sourced intentionally:
- to maximize scrutiny and robustness
- to build trust in the transaction engine
- to reach markets otherwise inaccessible

As such, changes in `public/` must be treated as kernel-impacting
unless clearly proven otherwise.

Proactive architectural scanning must be applied to all changes
in `public/`, especially those affecting concurrency, recovery,
state transitions, or event semantics.
At minimum: check ADRs + `preventing_deadlocks.md` before changing
locks, recovery paths, or FSM-related code.
  
-------------------------------------------------------------------------------
13) Canonical architectural source
-------------------------------------------------------------------------------

The development repository is the canonical source of architectural intent.

Stable repositories may optimize or fix behavior, but must not redefine
architectural invariants or transaction semantics.

Normal flow is development first, then merge downstream into stable.
Stable → development is exceptional.

If a stable change implies a shift in invariants, that shift must be:
- explicitly documented in ADRs
- backported to development as architectural update
- reviewed as CORE-TOUCH

Additional events in stable must be additive and observational.
They must not redefine transaction semantics, ordering guarantees,
or state transitions defined in development.

If an event changes correctness behavior, it is a CORE-TOUCH
architectural change and must be backported to development.

-------------------------------------------------------------------------------
14) Conditional Protocol Application
-------------------------------------------------------------------------------

Sections 15 and 16 of this document apply ONLY if the `specs/architecture/` 
directory is present in the repository.

* Internal/Stable Repos: These rules are binding and normative.
* Open Source Repos: If `specs/architecture/` is missing, the agent must fall 
  back to "Inferred Architecture" mode, clearly stating that architectural 
  intent cannot be formally verified.

-------------------------------------------------------------------------------
15) Answer Certainty & Evidence Protocol
-------------------------------------------------------------------------------

When `specs/architecture/` is present, the agent MUST label the basis of its 
information using mandatory phrases in the language of the chat (English,
French, Dutch, etc.):

* Expliciet gedocumenteerd / Explicitly documented / Explicitement documenté:
  "This is literally documented in the architectural knowledge provided by the organization."
* Volgens de code / According to the code / Selon le code:
  "Although this is not explicitly documented, source code analysis suggests that..."
* External Fallback / Secours externe / Externe fallback:
  "This is not documented architecturally, and source code analysis does not provide a clear answer. Based on external information, a possible explanation is..."
* Insufficient Information / Informations insuffisantes / Onvoldoende informatie:
  "Neither the documentation nor source code analysis provides enough information to answer this reliably, so I won’t provide an answer."

-------------------------------------------------------------------------------
16) Strict Module & Mapping Discipline
-------------------------------------------------------------------------------

When `specs/architecture/` is present, the following mapping and scanning 
rules apply:

* Module Definitions: Architectural modules only exist if there is a 
  corresponding `.md` file in `specs/architecture/modules/`, except where
  explicit exceptions below apply.
* Proactive Bidirectional Scanning: Upon session start, the agent MUST 
  validate the integrity between the `modules/` folder and the source code:
  - Missing Documentation: Alert the user if a source folder exists without 
    a corresponding `.md` file in `specs/architecture/modules/`. State:
    "Module documentation is missing for [M]."
    If a similar module doc exists that differs only by version number, the
    agent should present the same explanation as the analogous documented
    module, adapted to the specific version, and may note the analogy.
    See `specs/architecture/meta/module_contract.md` for the versioned-analogs
    exception.
    If multiple analogous docs exist, prefer the most specific (highest
    version) documented module as the source for the analogy.
  - Stale Documentation: Alert the user if an `.md` file exists in 
    `specs/architecture/modules/` but no corresponding source folder is found. 
    State: "The documentation for module [M] appears to be stale. A description 
    exists, but no corresponding source folder was found."
* Authority Lockdown: If stale documentation is detected, the agent MUST NOT 
  use that specific `.md` file as a basis for architectural guarantees until 
  the mapping discrepancy is resolved.
* Mapping Rules:
  - An architectural module M is implemented in the source code folder named M.
  - Exception: The architectural module `atomikos-util` is implemented in 
    the source code folder named `util`.
* Authority Boundaries:
  - Architectural Questions: Questions about module boundaries, ownership,
    or guarantees should be based on documentation. Use labels in the language
    of the chat (e.g., English/French/Dutch): "volgens de code" may complement
    "expliciet gedocumenteerd"; if there is a conflict, the documented facts
    yield to code when the implementation is unambiguous and directly evident
    from source code (no reasonable doubt). In that case, the agent should
    label the answer as "volgens de code" (localized to the chat language) and
    state that the documentation appears outdated.
    If no documentation exists and the answer is unambiguous and directly
    evident from source code, the agent may answer using the "volgens de code"
    label (localized to the chat language).
    Unambiguous means: a single, direct code path or definition with no
    plausible alternative interpretation.
  - Behavioral Questions: Questions about runtime behavior or errors may
    consult source code, but answers must not make architectural claims.
  - If clarity is not reached after ~5 minutes of focused inspection,
    conclude that no unambiguous answer is possible.
* Documentation Style (Code vs Docs):
  - Document only what is not unambiguously derivable from the code.
  - Avoid duplicating code-level behavior in documentation.
  - Prefer code pointers (where to look) over re-stating logic.
  - Document only intent, invariants, and non-obvious guarantees.
* General Principles (Docs vs Inference):
  - Documented architecture captures intent but is inferior to the reality in
    the code.
  - ADRs are the absolute truth and cannot be overridden.
  - Use labels in the language of the chat for documented vs inferred facts.
* Docs-Advisory Mode:
  - Treat documentation as advisory when it conflicts with unambiguous code.
  - The agent may ignore documentation if the answer is directly evident from
    code or if documentation appears outdated.
  - ADRs remain authoritative and are not overridden by code.
  - If any documentation conflicts with ADRs, treat it as an error and
    escalate.
  - ADRs must be consulted for architectural questions and for codebase
    verification when assessing architectural compliance.
  - Any doc/code conflict requires an explicit review decision: update the
    code to match the doc, or update the doc to match the code.
* Review Checklist (Docs):
  - For any change, explicitly answer: "Does this change require updating an
    ADR or a module doc?" If yes, update the relevant doc or record why not.
  - For any code change or review, the agent must check ADR compliance and
    explicitly state whether the change conforms to ADRs.
  - The agent should only mention these checks in its response when a doc
    update is required or when ADR compliance is violated.
* Versioned Module Defaults:
  - For each higher version module, set RELATIONSHIPS to:
    "Replaces <module><previous-version>" (replaces previous version).
  - STABILITY is "Stable" for all but the highest version of that module
    series; the highest version is "Evolving".
  - For derived modules, DEPENDENCIES should be inferred from the module's
    `pom.xml`, limited to compile or provided scope.
* Naming Inference Default:
  - For modules in `public/` or `private/` that are not explicitly documented,
    it may be inferred from naming that "transactions-XXX" means integration
    with technology XXX, but only if XXX is a known third-party technology or
    standard.
    The canonical list of known technologies is in:
    `specs/architecture/meta/known_technologies.md`.
    If the name is composite, only the known third-party segment may be used
    for inference.
* Test Modules:
  - Modules whose name contains "test" or "tests" are not architecturally
    relevant modules and do not require module documentation.
