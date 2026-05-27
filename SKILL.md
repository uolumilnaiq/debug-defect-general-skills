---
name: debug-defect-general
description: Use when diagnosing bugs, regressions, crashes, stuck states, missing callbacks, wrong UI behavior, network/media failures, or any user-reported defect requiring root-cause analysis before code changes.
---

# General Defect Investigation

## Core Rule

Do not modify code before the root cause is clear and the user confirms the proposed fix.

If evidence is insufficient, only propose or add targeted diagnostic logs after user confirmation. A plausible guess is not a root cause.

## When to Use

- Bug, regression, crash, or unexpected behavior reported by a user or test.
- Missing callback, stuck state, wrong UI behavior, network or media failure.
- Any defect where cause is unknown and code changes are being considered.

## When NOT to Use

- Requirement change disguised as a bug (behavior is correct, spec changed). Clarify with the user first.
- Pure performance degradation without a clear failure path — use a profiling-focused approach instead.
- Known, already-confirmed root cause with an agreed fix — skip directly to Change Gate.

## Three-Layer Method

1. **Establish the code defect from logs and code.**
   Use existing logs, repro steps, stack traces, state snapshots, and code paths to identify the concrete suspicious branch, state update, callback, request, or resource lifecycle. If logs are missing, state the hypothesis and propose debug logs with searchable tags.

2. **Prove whether that code defect explains the user-visible problem.**
   Check whether the candidate root cause explains all known symptoms and has no obvious counterexample. If logs conflict, do not choose a side silently; propose the next logs or experiment needed to break the tie.

3. **Give repair options.**
   Always provide a minimum-work fix and an architecture-aligned fix. Add a learning-oriented alternative only when the root cause reveals an architectural decision worth reconsidering (e.g., wrong abstraction, missing retry boundary, incorrect ownership). State impact, risks, rollback point, and exact verification evidence for each option.

## Required Output Shape

Use this structure unless the user asks for a shorter answer:

```text
Problem restatement     (问题复述)
Known facts             (已知事实)
Code path               (代码路径)
Root-cause judgment     (根因判断)
Confidence              (置信度)
Missing evidence        (还缺的证据)
Fix options             (修复方案)
Impact scope            (影响面)
Verification plan       (验证方案)
Points needing user confirmation  (需要用户确认的点)
```

Keep each section concise. Omit sections that are truly irrelevant, but never omit root-cause confidence or confirmation requirements before code changes.

## Evidence Discipline

Separate every important statement into one of these classes:

- **Fact**: directly supported by logs, code, reproduction, or observed behavior. Before treating a log as Fact, verify its source path and timing completeness — truncated, out-of-order, or non-repro-path logs are Inference at best.
- **Inference**: reasoned from facts but not directly proven. Cap inference chains at two hops; a third-hop conclusion must be labeled To-verify.
- **To verify**: missing evidence needed to confirm or reject the inference.

Root-cause confidence — use objective criteria, not gut feel:

- **High**: reproducible + code path confirmed in source + log directly witnesses the failure branch. No counterexample found.
- **Medium**: main symptom explained, but one key log or one reproduction step is missing.
- **Low**: cause is plausible from code reading alone, with no supporting log or reproduction. **Do not enter Change Gate at Low confidence.**

## Priority

When multiple defects exist, rank by impact and probability. If unclear, ask for reproduction frequency or test results.

Fix in this order:

1. Data loss, crash, wrong external callback, unrecoverable stuck state.
2. Repeated failure, degraded core workflow, incorrect retry or resource lifecycle.
3. Logging gaps, confusing UI state, low-frequency edge cases.

## Logs

Prefer logs over speculation. If logs conflict or cannot prove the root cause, propose additional logs.

Log levels: `debug` for temporary diagnostic logs (use a searchable problem tag); `info` for important state transitions; `warning` for abnormal but handled branches; `error` for final failure outcomes.

Load [logging-checklist.md](references/logging-checklist.md) when adding or recommending diagnostic logs.

## Change Gate

**Hard block at Low confidence:** If confidence is Low, do not enter this gate. Instead, output a diagnostic log plan and stop. Only proceed after the user confirms the logs and new evidence elevates confidence to Medium or High.

Before implementing a fix, confirm both:

1. The root cause is explicit enough to defend (confidence ≥ Medium).
2. The solution has no obvious negative impact.

Check negative impact across:

- State machine transitions.
- Cross-process or native callback timing.
- Retry, timeout, and reconnect policy.
- Resource acquisition and release order.
- Concurrency, duplicate calls, and idempotency.
- Backward compatibility if the user still cares about it.

After presenting the fix plan, wait for explicit user confirmation before editing code. "Just fix it" or urgency in tone is not confirmation. Confirmation means the user has acknowledged the specific fix plan.

## Iterative Convergence (Non-Deterministic Defects)

For defects that cannot be reliably reproduced — race conditions, timing-dependent failures, flaky tests, intermittent crashes — the Three-Layer Method cannot complete in a single pass. Use this iterative path instead:

1. **Narrow the window**: identify the smallest time or code range where the failure can occur. Log entry and exit of that window.
2. **Instrument and observe**: add diagnostic logs at the race boundary or non-deterministic branch. Run until the failure recurs.
3. **Re-enter Layer 1** with the new evidence. Repeat until confidence reaches Medium.
4. Only then proceed to Layer 2 and 3.

Do not force a Low-confidence root cause just because the defect is hard to reproduce. State the iteration status explicitly in the output under "Missing evidence."

## Verification

Every fix proposal must include:

- Reproduction path.
- Logs that should appear after the fix.
- Logs or symptoms that should no longer appear.
- Failure branch: if the verification log is absent, what to check next.

Load [verification-protocol.md](references/verification-protocol.md) for reusable verification templates.

## Common Mistakes

| Mistake | What to do instead |
|---|---|
| Treating an Inference as a Fact | Label it Inference and state what evidence would elevate it to Fact |
| Treating unverified logs as Fact | Confirm the log came from the repro path and has correct timing before classifying as Fact |
| Proposing a fix at Low confidence | Hard stop — output a diagnostic log plan, wait for user confirmation and new evidence |
| "User seems urgent, just fix it" | Urgency in tone is not explicit confirmation. State the fix plan and wait |
| Skipping Change Gate when fix "looks obvious" | Always run the 6-dimension impact check, no exceptions |
| Forcing a root cause on a non-deterministic defect | Use the Iterative Convergence path; do not fabricate certainty |
| Carrying over prior-turn diagnosis as confirmed Fact | Re-state confidence explicitly each turn; prior conclusions are Inference until re-verified |
| Removing diagnostic logs too early | Keep until fix is verified in production, then remove or promote to `info` |
