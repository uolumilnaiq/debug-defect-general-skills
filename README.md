# debug-defect-general

A Claude Code / Codex skill for evidence-first defect investigation.

Enforces a structured root-cause workflow before any code changes: build a log-and-code fact chain, classify statements as Fact / Inference / To-verify, gate fixes behind explicit confidence thresholds, and handle non-deterministic defects (race conditions, flaky tests) with an iterative convergence path.

## When to use

Invoke this skill when diagnosing bugs, regressions, crashes, stuck states, missing callbacks, wrong UI behavior, network/media failures, or any user-reported defect where the root cause is not yet confirmed.

## Key behaviors

- **Three-Layer Method** — establish the defect → prove it explains the symptom → propose repair options
- **Confidence levels** — objective criteria for High / Medium / Low; hard block on code changes at Low
- **Change Gate** — 6-dimension impact check + explicit user confirmation required before editing code
- **Iterative Convergence** — dedicated path for race conditions and non-deterministic failures
- **Evidence Discipline** — logs must be verified for source path and timing before being classified as Fact

## Install

### Claude Code

```bash
cp -r debug-defect-general ~/.claude/skills/debug-defect-general
```

### Codex

```bash
cp -r debug-defect-general ~/.codex/skills/debug-defect-general
```

## Usage

```
/debug-defect-general
```

Or reference it in a prompt:

```
Use debug-defect-general to investigate this crash.
```

## Files

```
debug-defect-general/
├── SKILL.md                          # Main skill
└── references/
    ├── logging-checklist.md          # Diagnostic log field guide
    └── verification-protocol.md      # Fix verification templates
```
