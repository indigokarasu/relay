# Relay Schemas

## UserMessage
```json
{"type":"user_message","id":"string","timestamp":"string","device_id":"string","text":"string","attachments":["string"]}
```

## DeviceContext
```json
{"type":"device_context","id":"string","timestamp":"string","device_id":"string","device_model":"string","os":"string","locale":"string","timezone":"string","battery":{"level":"number","charging":"boolean"},"network":{"type":"string","ssid":"string"}}
```

## LocationEvent
```json
{"type":"location_event","id":"string","timestamp":"string","device_id":"string","lat":"number","lon":"number","accuracy_m":"number","source":"string"}
```
Requires permission grant.

## AttachmentUpload
```json
{"type":"attachment","attachment_id":"string","timestamp":"string","device_id":"string","media_type":"string","filename":"string","sha256":"string","bytes":"number","storage_ref":"string"}
```

## PermissionRequest
```json
{"request_id":"string","timestamp":"string","permission":"string — location|camera|microphone|notifications|contacts|calendar|files","reason":"string","scope":"string — one_time|session|ongoing","alternatives":"string|null","retention":"string|null","status":"string — queued|requested|granted|denied|deferred"}
```
