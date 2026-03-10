---
name: relay
version: 1.0.0
description: >
  Device gateway AgentSkill. Ingests telemetry, messages, and attachments from
  Relay.app clients and exposes normalized device context and device actions to
  OpenClaw agents with explicit permission discipline.
metadata:
  openclaw:
    id: ocas.relay
    emoji: "📡"
    requires:
      bins: ["python3"]
    install:
      - kind: pip
        package: fastapi
      - kind: pip
        package: pydantic
      - kind: pip
        package: websockets
    storage:
      root: ".relay"
    permissions:
      os: ["optional:location", "optional:camera", "optional:microphone", "optional:notifications"]
      network: ["required:local_ws_http"]
---

# Relay (ocas.relay) — Device Interface Gateway

## Overview

Relay connects OpenClaw agents to user devices through Relay.app clients. It normalizes inbound device signals (messages, context, attachments, location) and offers a controlled action surface (notify, request capture, request permission, etc.). Relay is designed to batch permission requests where possible and keep the permission model explicit and auditable.


## Suite Conventions (shared across ocas.*)

- Versioning: semantic, pinned at 1.0.0 for this release.
- Storage: each skill writes only under its own dot-folder in the workspace (e.g., .praxis/), plus optional human-facing artifacts under reports/.
- Logs: append-only JSONL for events/decisions wherever feasible.
- Explainability: any decision that changes behavior or triggers an external action must be recorded as a DecisionRecord with evidence references.
- Independence: skills are designed to interoperate when co-installed but must not require each other to function.


## Invariants (non-negotiable)

- No silent permissions: Relay must not access privileged device data without an explicit PermissionGrant.
- Minimal scope: request only the smallest permission needed for a specific capability.
- Auditable: every permission request and grant is recorded as a DecisionRecord.
- Safe defaults: if permission is not granted, fall back to non-privileged behavior or ask one minimal question.

## External Interface

### Commands

- relay.ingest.message
- relay.ingest.context
- relay.ingest.location
- relay.ingest.attachment
- relay.request.permission
- relay.request.capture_photo
- relay.request.capture_audio
- relay.notify
- relay.status

### Hooks (recommended)

- Inbound websocket message -> ingest.*
- Agent asks for device context -> relay.status (returns latest normalized context)
- Any permission requirement -> relay.request.permission (queue by default)

### Config

File: .relay/config.json (namespace "relay")
- server.listen_host
- server.listen_port
- retention.days (default 7)
- permissions.batch_by_default (default true)
- attachments.max_mb

## Input Contract (normalized payloads)

Relay accepts JSON messages over local websocket/HTTP.

### UserMessage
{
  "type":"user_message",
  "id":"string",
  "timestamp":"string",
  "device_id":"string",
  "text":"string",
  "attachments":["attachment_id"]
}

### DeviceContext
{
  "type":"device_context",
  "id":"string",
  "timestamp":"string",
  "device_id":"string",
  "device_model":"string",
  "os":"string",
  "locale":"string",
  "timezone":"string",
  "battery":{"level":"number","charging":"boolean"},
  "network":{"type":"string","ssid":"string"}
}

### LocationEvent (requires permission)
{
  "type":"location_event",
  "id":"string",
  "timestamp":"string",
  "device_id":"string",
  "lat":"number",
  "lon":"number",
  "accuracy_m":"number",
  "source":"string"
}

### AttachmentUpload
{
  "type":"attachment",
  "attachment_id":"string",
  "timestamp":"string",
  "device_id":"string",
  "media_type":"string",
  "filename":"string",
  "sha256":"string",
  "bytes":"integer",
  "storage_ref":"string"
}

## Permission Model

### PermissionRequest
{ "type":"object",
  "required":["request_id","timestamp","permission","reason","scope"],
  "properties":{
    "request_id":{"type":"string"},
    "timestamp":{"type":"string"},
    "permission":{"enum":["location","camera","microphone","notifications","contacts","calendar","files"]},
    "reason":{"type":"string"},
    "scope":{"enum":["one_time","session","ongoing"]},
    "alternatives":{"type":"string"},
    "retention":{"type":"string"},
    "status":{"enum":["queued","requested","granted","denied","deferred"]}
  }
}

Rules:
- Default: queue permission requests for daily debrief (Praxis) or explicit user review.
- Mid-task prompt only when the current request cannot be completed without it.

## Storage Layout

.relay/
  config.json
  inbound.jsonl
  context_latest.json
  attachments/
  permissions.jsonl
  decisions.jsonl

## Acceptance Tests

1) No-permission path: location_event is rejected unless a PermissionGrant exists.
2) Audit: every request.permission writes a PermissionRequest and a DecisionRecord.
3) Attachment: attachment upload stores file and records sha256.
4) Minimality: camera permission request must include reason and scope; otherwise invalid.

---

## Skill Identity

- Skill name: `ocas-relay`
- Version: `1.0.0`
- Skill type: `system`
- Author: `Indigo Karasu`
- Email: `mx.indigo.karasu@gmail.com`

---

## Responsibility Boundary

Relay normalizes inbound device signals and manages device permissions.

Relay does not interpret or act on the signals it ingests. Signal analysis belongs to other skills.

Relay does not send outbound communications. Message delivery belongs to Dispatch.

---

## Optional Skill Cooperation

This skill may cooperate with other skills when present but must never depend on them.
If a cooperating skill is absent, this skill must still function normally.

- Look — provide device-originated images and pre-parse data for image-to-action processing.
- Dispatch — provide inbound message signals for communication thread tracking.
- Vesper — provide device context signals for briefing generation.
- All skills — provide normalized device context when requested via relay.status.

---

## Journal Outputs

This skill emits the following journal types as defined in the OCAS Journal Specification (spec-ocas-Journals.md):

- Observation Journal

Relay emits Observation Journal entries recording inbound device signals, permission grants, and attachment ingestions.

---

## Visibility

visibility: private

This skill is private and must not be published or distributed openly.

---

## Universal OKRs

This skill must implement the universal OKRs defined in the OCAS Journal Specification (spec-ocas-Journals.md).

Required universal OKRs:

- Reliability: success_rate >= 0.95, retry_rate <= 0.10
- Validation Integrity: validation_failure_rate <= 0.05
- Efficiency: latency trending downward, repair_events <= 0.05
- Context Stability: context_utilization <= 0.70
- Observability: journal_completeness = 1.0

Skill-specific OKRs should be defined in the built SKILL.md to measure domain-relevant outcomes.

---

## Required Package Output

Forge must produce a complete Agent Skill package:

```text
ocas-relay/
  skill.json
  SKILL.md
  references/
    schemas.md
    permission_model.md
```

This skill is private and must not be published or distributed.

### `skill.json` Requirements

Create a valid `skill.json` with:
- `name`: `ocas-relay`
- `version`: `1.0.0`
- `description`: routing-optimized text
- `author`: `Indigo Karasu`
- `email`: `mx.indigo.karasu@gmail.com`

The description must make clear that this skill is for:
- ingesting device telemetry, messages, and attachments from Relay.app clients
- normalizing device context for other skills
- managing explicit device permissions with audit trails
- providing controlled device actions (notify, capture, request permission)

### `SKILL.md` Requirements

`SKILL.md` must begin on line 1 with valid YAML frontmatter delimited by `---`.

Target size: 120 to 200 lines.

#### Required `SKILL.md` Sections

The markdown body must contain these sections in this order:

1. `# Relay`
2. `## When to use`
3. `## When not to use`
4. `## Core promise`
5. `## Commands`
6. `## Invariants`
7. `## Permission discipline`
8. `## Support file map`
9. `## Storage layout`
10. `## Validation rules`

### Commands

The built skill must implement these commands exactly as defined in the existing spec:

- `relay.ingest.message` — ingest a user message from a device client
- `relay.ingest.context` — ingest device context (model, OS, locale, timezone, battery, network)
- `relay.ingest.location` — ingest a location event (requires permission grant)
- `relay.ingest.attachment` — ingest an attachment with media type, filename, sha256 hash, and storage ref
- `relay.request.permission` — request a device permission with reason, scope, and alternatives
- `relay.request.capture_photo` — request a photo capture from the device camera
- `relay.request.capture_audio` — request an audio capture from the device microphone
- `relay.notify` — send a notification to the device
- `relay.status` — return latest normalized device context and permission state

### Storage Layout

As defined in the existing spec:

```text
.relay/
  config.json
  inbound.jsonl
  context_latest.json
  attachments/
  permissions.jsonl
  decisions.jsonl
```

### Config

File: `.relay/config.json`

Required fields:
- `server.listen_host`: websocket/HTTP listen host
- `server.listen_port`: websocket/HTTP listen port
- `retention.days`: data retention period (default 7)
- `permissions.batch_by_default`: whether to batch permission requests for daily review (default true)
- `attachments.max_mb`: maximum attachment size

### `references/schemas.md`

Must define schemas for all input payloads exactly as specified in the existing Input Contract section:
- `UserMessage` — type, id, timestamp, device_id, text, attachments
- `DeviceContext` — type, id, timestamp, device_id, device_model, os, locale, timezone, battery, network
- `LocationEvent` — type, id, timestamp, device_id, lat, lon, accuracy_m, source (requires permission)
- `AttachmentUpload` — type, attachment_id, timestamp, device_id, media_type, filename, sha256, bytes, storage_ref
- `PermissionRequest` — request_id, timestamp, permission (location|camera|microphone|notifications|contacts|calendar|files), reason, scope (one_time|session|ongoing), alternatives, retention, status (queued|requested|granted|denied|deferred)
- `DecisionRecord` — decision_id, decision_type, evidence_refs, outcome, timestamp

### `references/permission_model.md`

Must document:
- the no-silent-permissions invariant
- minimal-scope principle
- auditable permission lifecycle (request -> queue -> grant/deny -> log)
- default behavior: queue permission requests for daily debrief review
- mid-task prompt only when the current request cannot complete without the permission
- that every permission request and grant is recorded as a DecisionRecord
- safe fallback behavior when permission is not granted

---

## Validation Requirements

A build is complete only if all of the following are satisfied.

### Routing Validation

Should trigger:

- "What is the current device context?"
- "Request camera permission for photo capture."
- "Show the status of device connections."
- "Ingest this attachment from the iOS client."

Should not trigger:

- "Research this company's background."
- "Draft a reply to this email."
- "What restaurants should I try?"
- "Analyze my behavioral patterns."

### Structural Validation

Confirm:

- `skill.json` exists with valid metadata
- `SKILL.md` starts with valid YAML frontmatter on line 1
- all referenced support files exist
- `SKILL.md` points to any support file it depends on

### Usefulness Validation

Confirm:

- the skill has one sharp promise
- first useful action is obvious
- the package is installable without hidden context

## Final Response Format for the Coder

Return:

1. package tree
2. full contents of every file
3. brief validation summary

Do not return planning commentary or references to absent documents.
