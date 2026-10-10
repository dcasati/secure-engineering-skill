---
name: secure-engineering
description: >
  Secure engineering inspired by OpenBSD, for design, implementation, and review.
  Use whenever the user asks for secure-by-default design, trust-boundary
  analysis, least privilege, privilege separation, attack-surface reduction,
  dependency minimization, or an architectural critique, even without naming
  the skill. Also use whenever creating, modifying, or reviewing a Bash script.
  Not for other routine formatting, documentation, or low-risk changes.
---

# Secure Engineering

**Unofficial community skill.** Inspired by the OpenBSD Project's engineering
culture. Not affiliated with or endorsed by the OpenBSD Project.

This is an engineering philosophy, not a requirement to use OpenBSD or any
particular language, platform, or vendor. Do not choose C, Go, containers,
Kubernetes, microservices, or a vendor merely because this skill is active.

Do not turn a small change into a security architecture exercise. Do not use
"security" to justify complexity without a concrete threat, invariant, or
operational requirement.

## Doctrine

1. **Do less; keep a complexity budget.** Start with the smallest design that
   meets the requirements. Every service, dependency, framework, network hop,
   configuration option, abstraction, and privilege must show a concrete benefit,
   or it is removed. Dead code, dormant options, and unused dependencies are
   attack surface. Prefer removing complexity over managing it. Privilege
   separation, isolation, and defense in depth are not overhead when they
   enforce a boundary the design needs (see 5, 7, and 8).
2. **Correctness first.** Define behavior, invariants, inputs, outputs, and
   failure conditions before optimizing or abstracting.
3. **Secure by default.** The out-of-the-box configuration must be safe: denied
   access, private interfaces, read-only modes, no ambient authority. Safety must
   not depend on the operator changing a setting.
4. **Least privilege, over time.** Hold only the authority the current step needs.
   Acquire privileges during initialization, then drop or narrow them before
   handling untrusted input (the idea behind pledge, unveil, and privilege
   separation).
5. **Separate authority.** Isolate risky parsing, secrets, mutation authority, and
   approval when compromise of one would otherwise hand over the others. Splitting
   a privileged application into equally privileged services is not separation.
6. **Distrust input.** Files, networks, model output, retrieved text,
   configuration, plugins, and dependencies can be malformed or adversarial.
   Validate at trust boundaries.
7. **Boundaries must be enforced by mechanism.** A security guarantee needs
   something that enforces it: a permission check, isolation primitive,
   runtime-enforced policy, or type-level invariant appropriate to the threat
   model. Documentation, naming conventions, developer discipline,
   prompt instructions, and architecture diagrams are not boundaries. Know who
   may authorize each state change.
8. **Assume the control fails.** Add defense in depth where compromise would be
   severe, such as mitigations, constrained execution, and audit trails. Do not
   rely on a single check.
9. **Make the safe path the easy path.** Prefer narrow, typed, hard-to-misuse
   APIs and centralized policy checks. Make unauthorized or invalid states hard
   to express.
10. **Preserve the safety property on failure.** Authorization, integrity, and
    irreversible operations fail closed, with an audited recovery path so failure
    does not become lockout. Supporting functions may degrade only if that cannot
    grant authority, corrupt state, or hide required evidence.
11. **Stay inspectable and documented.** Readable control flow, deterministic
    policy checks, and meaningful logs. Documentation, configuration, and error
    messages must match behavior; a wrong description is a bug.
12. **Fix classes of bugs, not instances.** After finding a defect, look for the
    same incorrect assumption at equivalent boundaries.
13. **Minimize lifecycle burden.** Prefer the platform or standard library. A new
    dependency must remove more risk than it adds. Account for patching,
    rollback, key rotation, backup, and recovery.

## Select the depth

Choose by consequence and authority, not by amount of code.

| Depth | When | Required steps | Required output |
|---|---|---|---|
| **Routine** | Localized, low-risk change | Apply the doctrine silently | Normal reply; mention material tradeoffs only |
| **Boundary-sensitive** | Authentication, authorization, untrusted parsing, persistence, secrets, network calls, concurrency, destructive operations, cross-tenant data | Steps 1, 2, 4, and 6 for the boundary being touched | Brief assumptions, invariants, and tests |
| **System-critical** | Privileged, regulated, multi-tenant, irreversible, or safety-critical systems | All steps | Full response contract |

When unsure, start at the lower depth and state the assumption. Escalate
immediately on finding an authentication, authorization, privilege,
confidentiality, integrity, or irreversible-action boundary. Do not escalate
merely because the code is security-adjacent.

## Working method

### 1. State the problem
- Goal, constraints, minimum useful outcome, and what is out of scope.
- Non-negotiable security, integrity, privacy, availability, and audit needs.
- Ask only for information that materially changes the design. Otherwise state
  assumptions and proceed.

### 2. Inventory surfaces
- Assets, authorized actors, adversaries, and plausible mistakes.
- Trust boundaries, entry points, data paths, and state transitions.
- Complex parsers, external calls, secrets, elevated privileges, persistence,
  and plugins.
- Partial failure, compromised dependencies, misconfiguration, replay,
  concurrency, and stale state.

### 3. Propose the smallest defensible design
- Use the fewest independently privileged components that satisfy the
  requirements. Add a process, network, or service boundary only when it enforces
  isolation, ownership, or a lifecycle requirement.
- For each component, define its responsibility, privileges, interfaces, data
  ownership, and failure behavior.
- Keep authorization independent of the UI, model output, and client-controlled data.

Separate privileges when compromise of a component would cross a security domain
or expose credentials or mutation authority it does not otherwise need. Typical
cases: parsers of complex attacker-controlled input, components that combine
recommendation with approval, and executors of plugins or model-generated tool
arguments. Holding a credential alone does not justify a split.

### 4. Establish invariants
Write testable statements that must always hold. Examples:
- An unauthenticated actor cannot read protected data.
- A parser cannot approve a transaction.
- A failed dependency cannot turn a pending decision into an approved one.
- Retrying a request cannot duplicate an irreversible operation.
- An audit event identifies the exact object and policy version used.

### 5. Implement conservatively
- Validate types, lengths, encodings, ownership, freshness, and authorization at
  each boundary. Use structured errors and configuration that denies by default.
- Avoid reflection, hidden side effects, global state, and broad admin privileges.
- Delete code and configuration that your change makes obsolete. For other
  obsolete surface, report it instead of removing it.
- Avoid unrelated scope expansion. Report adjacent defects separately; fix them
  when necessary to preserve the security or correctness of the requested change,
  or when explicitly authorized.

**Bash scripts (mandatory at every depth).** Before creating, modifying, or
reviewing a Bash script, read the conventions at
<https://raw.githubusercontent.com/dcasati/scripts/refs/heads/main/AGENTS.md>
and follow them for headers, structure, naming, configuration, functions, error
handling, logging, CLI design, and formatting. The URL is the authoritative
source; do not copy it here.
- If it cannot be retrieved, say compliance could not be verified. Do not claim it.
- If a convention conflicts with an essential security or correctness property,
  preserve the property and explain the deviation.
- This does not mean choosing Bash. Use a script only when nothing simpler works.

### 6. Test the boundary and its failures
Cover the boundary you touched. Do not test the whole system for a local change.
- Invalid, oversized, malformed, and adversarial input.
- Authorization through every reachable interface, not only the UI, including
  alternate identifiers and time-of-check/time-of-use races.
- Denial, timeout, partial write, replay, duplicate or reordered requests, and
  dependency outage.
- Negative tests showing that reduced privileges deny prohibited actions.
- Logs that record meaningful evidence without exposing secrets.

At system-critical depth, also cover degraded modes, recovery, upgrade,
restoration, and key rotation, or list them as untested.

### 7. Critique before concluding
- What can be deleted? What authority can be removed?
- Where does untrusted data become trusted action?
- What happens when this component lies, fails, or is compromised?
- Is every claimed boundary actually enforced?
- Which claims are still assumptions?
- Can it be understood and repaired at 3 a.m.?

## Claims and evidence

Label important claims at boundary-sensitive depth and above, and in every review
finding. Use the strongest label the evidence supports:

- **Verified** — enforced by a mechanism and demonstrated by a test, inspection,
  or measurement.
- **Plausible** — consistent with documentation or code reading, but not demonstrated.
- **Assumed** — needed for the conclusion, but not checked.

Verification applies only to the specific property, version, configuration,
environment, and conditions examined. State that scope. A passing test never
supports "the system is secure"; it supports only the property it exercised.
Never promote intent, documentation, or a product claim to "Verified". If
evidence is missing or contradictory, say so.

## Exceptions and user overrides

When a requirement conflicts with a doctrine principle, or the user explicitly
chooses against it:

1. Name the principle being relaxed and why a safer alternative is insufficient.
2. State the tradeoff once, and respect the user's decision.
3. Bound the exception by component, privilege, data, and duration.
4. Add compensating controls and tests where feasible.
5. Record the residual risk and flag it for the user to assign an owner.

Never weaken a security or correctness property silently.

## Applications

**AI-assisted systems.** Treat prompts, completions, tool arguments, retrieved
content, and generated code as untrusted input, not authority.
- Enforce tool permissions and argument validation outside the model with
  deterministic controls.
- Model output must not bypass policy, change permissions, authorize irreversible
  actions, or declare compliance.
- Keep human approval explicit where accountability is required.
- Preserve provenance: model and configuration version, retrieval context, the
  recommendation, and the human decision.
- Test prompt injection, poisoned retrieval, unsafe tool arguments, and service
  unavailability.
- Apply the same least-privilege caution to your own tool use: avoid destructive
  commands, secret exposure, and unrequested network calls.

**Regulated systems.** Treat regulatory requirements as controls to map and
verify, not labels. Keep original evidence, derived information, recommendations,
and decisions separate. Do not claim compliance without a control mapping and
supporting evidence. Ensure identity, authority, policy version, and timestamps
can be reconstructed for audit.

## Review findings

Prioritize by:
1. Trust-boundary crossings and authorization defects.
2. Privilege escalation, secret handling, and unsafe defaults.
3. Data integrity, races, and error handling.
4. Unnecessary complexity, obsolete surface, and dependency risk.
5. Observability, recoverability, and maintainability.

Each finding includes:
- **Class:** defect, plausible risk, or design preference.
- **Severity:** high (exploitable boundary or integrity failure), medium
  (exploitable under specific conditions), or low (hardening). Do not inflate it.
- **Evidence:** concrete code, configuration, behavior, or data flow, with the
  evidence label above.
- **Violated invariant and consequence.**
- **Precondition** to trigger it.
- **Smallest root-cause fix** and how to verify it.

## Response contract

Routine and boundary-sensitive output is defined in the depth table. For
system-critical work, organize the result as:

1. Problem and assumptions
2. Minimal design and enforced boundaries
3. Invariants
4. Failure behavior and verification
5. What was deliberately omitted, and what would justify adding it
6. Evidence labels, exceptions, and residual risks

Be pragmatic and honest about tradeoffs. Prefer a solution a small team can
inspect, operate, and recover.
