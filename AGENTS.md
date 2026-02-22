# AGENTS.md
Pillowfort / Busy Contribution Standards (Agents + Humans)

Automated and AI-assisted contributions are welcome, provided they meet the same production standards as human-written code.

This repo assumes extremely capable reviewers (including adversarial ones). Code must be defensible under scrutiny: explicit boundaries, fail-closed behavior, and visible failures.

---

## 0) Definitions

**Production code**  
Anything that runs at runtime, touches user data, affects execution flow, tool dispatch, parsing, permissions, scheduling, networking, persistence, or UI behavior.

**Security-critical / Authority code**  
Any code that:
- Parses or interprets commands, tags, or tool directives
- Makes permission/role decisions
- Dispatches tools or side effects
- Mutates system state in long-running loops
- Implements governance, policies, gates, or watchdog behavior
- Handles secrets/keys, model integrity checks, or plugin boundaries

Security-critical code must be minimal, explicit, and boring.

---

## 1) Non-Negotiables

### 1.1 No placeholders in production code
Placeholders are not acceptable in runtime or user-facing code.

**Do not commit or merge** production code containing:
- `TODO`, `FIXME`, `HACK` (unless explicitly scoped to non-production and tracked)
- `NotImplementedError`
- Placeholder returns (e.g., `return None`) used in place of implemented behavior
- “temporary” logic that silently skips behavior

**Note:** This does not forbid legitimate `None`/null semantics when they are part of the defined API or domain model. It forbids placeholder/sentinel returns used to avoid implementing real behavior. Legitimate `None` returns must be documented (briefly) and test-backed.

Unit tests may use mocks/stubs. Production code must be complete and test-backed.

### 1.2 Fail closed, not open
Security and authority boundaries must reject ambiguous or invalid input.

- Invalid or ambiguous directives must **not** execute “best effort.”
- No auto-repair of malformed directives in authority code.
- No silent fallback behavior that hides failures.

### 1.3 Failure states are telemetry
Failures are signal. Keep them visible.

- Do not introduce “graceful fallback” that masks runtime failures.
- Log and surface errors intentionally.
- Do not swallow exceptions without explicit rationale and telemetry.

### 1.4 Substantial architectural changes require tracking
Do not merge substantial architectural changes without:
- A tracked issue number, or
- An associated public forum thread, or
- An explicit design note referenced in the PR

This prevents accidental refactors driven by aesthetics alone.

---

## 2) Abstraction Discipline (Hard Rule)

**Abstractions only when absolutely necessary.**  
Nested abstractions must be used sparingly, and avoided entirely if possible.

### 2.1 Default: explicit code
Prefer:
- Explicit conditionals
- Explicit validation steps
- Explicit state transitions
- Explicit error handling
- Explicit logging

Over:
- Generic “smart helpers”
- Meta-framework patterns
- Deep middleware stacks
- Highly generalized validators
- Clever functional composition

### 2.2 Security-critical code: minimal abstraction
In authority/security-critical modules:
- Prefer explicit duplication over clever reuse
- Avoid multi-layer helper chains
- Avoid “unified” dispatch/validation frameworks unless duplication itself creates correctness risk

**If an abstraction makes authority decisions harder to trace, it is prohibited.**

### 2.3 Abstraction justification comment (required)
Any new abstraction in core execution paths must include a short comment explaining:
- Why the abstraction is necessary
- What risk it reduces
- Why it does not obscure authority boundaries

---

## 3) Tool System Parsing Standards

Parsing may be expressive. Execution must be literal.

### 3.1 Separate parsing from validation from execution
The tool pipeline must be layered:

1) Parse into a structured command object (no side effects)  
2) Validate schema strictly (fail closed)  
3) Validate permissions/role/scope (fail closed)  
4) Dispatch via whitelist only (explicit side effects)  
5) Log/audit all actions  

Parsing must never imply permission.

### 3.2 Strict syntax; no inference in authority
Authority parsing must not:
- “Guess” intent
- Auto-correct malformed tags
- Partially match namespaces/actions
- Execute on near-misses

If it’s not exactly valid, it is invalid.

### 3.3 Regex usage rules (if regex is used)
Regex in parsing/authority code must be:

- **Anchored** when matching full directives (`^...$`)
- **Whitelisted** character classes (no `.+` for identifiers)
- **Bounded** with explicit length limits
- **Safe** from catastrophic backtracking (avoid nested quantifiers)
- **Non-ambiguous** (no overlapping captures for critical fields)

Normalize before parsing:
- Normalize line endings
- Strip/deny control characters
- Reject zero-width characters if unsupported

No regex-based “repair.”

---

## 4) Commenting Standards (Institutional Memory)

Comments are institutional memory: time-travel notes for future maintainers. They preserve rationale, constraints, invariants, threat context, and refactor hazards that are not obvious from reading the code alone.

Comments must explain **why** an architectural/security decision exists, and in isolated cases **when/where** it applies. Comments must not narrate what the code obviously does.

### 4.1 Required uses of comments
Use comments for:
- Security/authority boundaries (parsing, permissions, dispatch, state mutation)
- Intentional strictness or explicit duplication ("ugly" code by design)
- Non-obvious constraints or invariants
- Refactor hazards (what breaks if “cleaned up”)
- Architectural rationale at module boundaries (module header preferred)

### 4.2 Prohibited comment patterns
Do not write comments that:
- Restate what is clearly visible in code
- Narrate control flow line-by-line
- Explain obvious conditionals (“if invalid, return”)
- Use vague intent (“important”, “hacky”, “temporary”) without rationale and tracking

### 4.3 Preferred comment content
Prefer short, specific notes that preserve:
- Why: rationale / tradeoff
- Prevents: failure mode / exploit class
- Invariant: what must remain true
- Scope: when/where the constraint applies (rare)
- Link: issue/CURRENT_STATE reference when relevant

### 4.4 Security boundary markers
Security-critical sections should include a clear marker:
- `// 🔒 SECURITY BOUNDARY`
- `// [SECURITY CRITICAL]`
- `// DO NOT ABSTRACT`
- `// FAIL-CLOSED REQUIRED`

Pick a convention and remain consistent within a module.

### 4.5 Comment budget rule
If removing the comment would not reduce a reviewer’s ability to understand risk, constraints, invariants, or safe refactor boundaries, delete the comment.

---

## 5) Testing Requirements

### 5.1 New functionality requires tests
New functionality must include unit tests (or updates to existing tests) covering the behavior.

### 5.2 Mocks and stubs
Mocks/stubs are allowed in tests and fixtures. Production logic must not depend on mocked behavior.

### 5.3 Test the edges
Authority/parsing changes must include tests for:
- invalid inputs
- ambiguous inputs
- boundary cases
- expected failure modes (fail closed)

---

## 6) Error Handling & Visibility

- Handle errors and edge cases explicitly.
- Do not silently return defaults on failure in authority paths.
- Prefer loud, logged failures over hidden “resilience.”

If a failure is expected, it should be visible (telemetry/logging) and documented.

---

## 7) Documentation Requirements

### 7.1 CURRENT_STATE.md updates
Behavioral changes must be documented in `CURRENT_STATE.md`:
- What changed
- Why it changed
- Any tradeoffs
- Any new constraints or expected failure modes

### 7.2 Open questions and ambiguities
If you encounter ambiguous requirements that affect:
- authority
- parsing
- permissions
- execution boundaries
- long-running state

Document them as open questions and request clarification rather than guessing.

---

## 8) Pre-Submission Checklist (Required)

Before submitting generated or automated changes, verify:

- [ ] No production file includes placeholders (`TODO`, `FIXME`, `NotImplementedError`, placeholder returns used as stubs)
- [ ] Mocked/stubbed behavior is limited to tests and fixtures
- [ ] New logic has accompanying tests or integration checks
- [ ] Errors and edge cases are handled explicitly
- [ ] Failure states remain visible (telemetry/logging); no silent graceful fallback
- [ ] Behavioral changes are documented in `CURRENT_STATE.md`
- [ ] Substantial architectural changes have an issue/thread reference
- [ ] All relevant tests pass before merge

---

## 9) Cultural Principle

We build systems that withstand creative iteration, aesthetic pressure, and adversarial review.

Elegance is welcome in safe layers. Authority layers must remain explicit, minimal, and auditable.

---

## 10) Multi-Agent Operational Standards

These are the required standards for all coding and review agents in this repo.

### 10.1 Role definitions

- **Coding agent**
  - Implement only the requested scope.
  - Prefer small, reversible changes over broad rewrites.
  - Keep authority/security changes explicit and test-backed.
  - Track all assumptions in comments or `docs/internal/OPEN_QUESTIONS_AND_DECISIONS.md` if unresolved.

- **Review agent**
  - Return issues in priority order:
    1. Security/authority regressions
    2. Data/persistence correctness
    3. Parser/dispatch/permission edge cases
    4. Resource/perf regressions
    5. Style/readability
  - Focus on failure modes and explicitness before polish.
  - Do not approve ambiguous behavior in fail-closed paths.

### 10.2 Shared workflow

- Always run a focused validation pass after changes:
  - targeted tests for changed module(s)
  - one end-to-end smoke check if behavior crosses transport/runtime boundary
- If the repository is in a hardening phase, prefer deterministic, minimal behavior over convenience.
- Use explicit telemetry over silent fallback in all authority and parsing paths.
- Escalate blockers (missing dependency, unclear contract, missing test fixtures) instead of inventing behavior.

### 10.3 Communication contract for agents

For every significant task handoff, report in a standard format:

- **What changed**: files + functional effect
- **Why**: rationale and tradeoff
- **Risks**: failure/attack/regression risk
- **Tests run**: commands + result
- **Open items**: unresolved questions and next actions

### 10.4 Review discipline (mandatory)

- Any code that changes routing, parsing, command execution, plugin loading, secrets, or governance:
  - must include explicit tests
  - must remain fail-closed on invalid/ambiguous inputs
  - must not introduce or keep placeholder behavior in hot paths
- If a change touches one authority module, inspect adjacent modules for coupled side effects in the same sweep.

### 10.5 Non-negotiables for all agents

- Never infer user intent outside the documented schema and parser contract.
- Never implement behavior to satisfy style only; preserve security invariants first.
- Never merge production changes without updating:
  - `docs/internal/OPEN_QUESTIONS_AND_DECISIONS.md` (if ambiguities remain), and
  - `CURRENT_STATE.md` (if behavior changed).

### 10.6 On ambiguity

- If a requirement is underspecified in an authority path:
  1. pause implementation at that point,
  2. propose a concrete interpretation,
  3. flag the exact ambiguity,
  4. proceed only after confirmation.
