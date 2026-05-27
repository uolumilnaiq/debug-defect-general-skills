# Diagnostic Logging Checklist

Use this when existing evidence is insufficient and temporary logs are needed.

## Required Fields

Pick fields that match the defect domain. Do not include all fields blindly.

**Universal fields (apply to most defects):**

- `traceId` or `flowId`: one investigation flow end-to-end.
- `requestId`: request/response correlation.
- `source`, `trigger`, `reason`: who initiated the action and why.
- `oldState`, `newState`: state transitions.
- `attempt`, `timeoutMs`, `elapsedMs`: retry and timeout behavior.
- `thread`, `process`, `pid`: cross-process or lifecycle issues.

**Domain-specific ID fields (add the ones relevant to your system):**

Choose the identity fields that make sense for your domain — for example, `userId`, `orderId`, `sessionId` for a web app; `roomId`, `peerId`, `consumerId` for a media/RTC system; `jobId`, `taskId` for a pipeline. The goal is correlation across log lines, not exhaustive coverage.

## Levels

- `debug`: temporary diagnostic logs, use a searchable problem tag.
- `info`: durable state transitions and externally visible actions.
- `warning`: abnormal branch that is handled.
- `error`: final failure, exception, unrecoverable outcome.

## Temporary Log Tagging

Use a unique tag or prefix so logs can be removed later:

```text
[diag:<problem-name>] flowId=... trigger=... state=...
```

Examples:

```text
[diag:photo-back] flowId=... source=navigationBack photoActive=true action=closePhoto
[diag:manual-switch] traceId=... peerId=... consumerId=... firstFrame=false stats=...
[diag:ws-reconnect] flowId=... oldUrl=... newUrl=... relinkResult=...
```

## Log Placement

Place logs at boundaries, not everywhere:

- External entry: UI click, native API, AIDL call, MethodChannel, EventChannel.
- State decision: if/else branch that chooses behavior.
- Async boundary: request sent, response received, timeout, callback.
- Resource lifecycle: create, pause/resume, close, dispose.
- Output boundary: callback sent to native, UI state emitted, file written.

## Removal Rule

Temporary debug logs should either be removed after the root cause is proven or converted to durable `info/warning/error` logs if they describe useful production behavior.
