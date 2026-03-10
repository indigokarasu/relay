---
name: ocas-relay
description: >
  Device gateway. Ingests telemetry, messages, and attachments from Relay.app
  clients. Normalizes device context. Manages permissions with audit trails.
---

# Relay

Relay connects OpenClaw agents to user devices through Relay.app clients. It normalizes inbound signals and manages permissions. Relay is private.

## When to use

- Ingest device messages, context, location, or attachments
- Request device permissions (camera, location, microphone)
- Get normalized device context for other skills
- Send notifications to the device

## When not to use

- Interpreting or acting on signals — other skills handle that
- Web research — use Sift
- Communication drafting — use Dispatch

## Core promise

Normalize device signals with explicit, auditable permission management. No silent permissions. Minimal scope. Safe defaults.

## Commands

- `relay.ingest.message` — ingest a user message
- `relay.ingest.context` — ingest device context
- `relay.ingest.location` — ingest location event (requires permission)
- `relay.ingest.attachment` — ingest attachment with sha256 hash
- `relay.request.permission` — request a device permission
- `relay.request.capture_photo` — request photo capture
- `relay.request.capture_audio` — request audio capture
- `relay.notify` — send notification to device
- `relay.status` — latest device context and permission state

## Invariants

- No silent permissions — every access requires explicit PermissionGrant
- Minimal scope — request only what the current capability needs
- Auditable — every request and grant recorded as DecisionRecord
- Safe defaults — if permission not granted, fall back to non-privileged behavior

## Permission discipline

Default: queue permission requests for daily debrief review. Mid-task prompt only when the current request cannot complete without the permission. Read `references/permission_model.md` for full rules.

## Support file map

- `references/schemas.md` — UserMessage, DeviceContext, LocationEvent, AttachmentUpload, PermissionRequest
- `references/permission_model.md` — lifecycle, batching, fallback behavior

## Storage layout

```
.relay/
  config.json
  inbound.jsonl
  context_latest.json
  attachments/
  permissions.jsonl
  decisions.jsonl
```

## Validation rules

- Location events rejected without PermissionGrant
- Every permission request includes reason and scope
- Attachments store sha256 hash
- All permission lifecycle events recorded as DecisionRecords
