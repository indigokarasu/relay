# Relay Permission Model

## No-Silent-Permissions
Relay never accesses privileged device data without an explicit PermissionGrant. Every request includes reason and scope.

## Minimal Scope
Request only the smallest permission needed. Camera permission for a single photo, not ongoing access.

## Lifecycle
Request → queue (default) → review (daily debrief or mid-task) → grant/deny → log.

## Batching
Default: batch permission requests for daily review. Mid-task prompt only when the current request cannot proceed without the permission.

## Fallback
If permission not granted: fall back to non-privileged behavior or ask one minimal clarifying question. Never fail silently.

## Audit
Every PermissionRequest and every grant/deny decision recorded as a DecisionRecord in decisions.jsonl.
