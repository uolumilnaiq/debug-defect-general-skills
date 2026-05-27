# Verification Protocol

Use this to define whether a defect fix actually worked.

## Minimal Verification Template

```text
Reproduction path:        (复现路径)
Key logs/symptoms before fix:  (修复前关键日志/现象)
Logs that should appear after fix:  (修复后应出现日志)
Logs that should no longer appear:  (修复后不应再出现)
If verification log absent, next step:  (失败时下一步)
```

## Success Log Anchor

A fix is verified only when logs prove the expected path, not just when the symptom disappears once.

Good anchors:

- Entry log: confirms the same user action or external event happened.
- Decision log: confirms the fixed branch was selected.
- Result log: confirms the expected callback, state, or resource operation completed.

## Failure Branch Examples

If the entry log is absent:

Check caller path, event routing, activity/service lifecycle, or channel registration.

If the decision log is absent:

Check state snapshot, stale cache, race, or guard condition.

If the result log is absent:

Check async timeout, callback dispatch, resource close order, or exception swallowing.

If logs show success but UI is wrong:

Check state subscription, rendering condition, view layering, and stale component state.

If logs show success but remote side is wrong:

Check request payload, response correlation, transport connection, and server support.

## Regression Scope

Choose regression cases based on changed behavior:

- State change: normal entry, retry path, cancel path, repeated call.
- Callback change: success callback, failure callback, duplicate callback, no-callback path.
- Resource lifecycle: acquire, pause/resume, close, repeated close.
- Network/media: connected, disconnected, reconnect success, reconnect failure.
- Cross-process: service alive, service restarted, host process killed, callback re-registered.
