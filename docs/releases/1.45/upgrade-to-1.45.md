# Instructions

## Versions

1. Helm chart versions:
   - dial: `6.4.0`
   - dial-core: `6.0.0`
   - dial-extension: `3.1.1`
   - dial-admin: `0.15.0`
2. Main components versions:
   - ai-dial-adapter-bedrock: `0.40.0`
   - ai-dial-adapter-openai: `0.40.0`
   - ai-dial-adapter-vertexai: `0.36.0`
   - ai-dial-adapter-dial: `0.15.0`
   - ai-dial-chat-themes: `0.16.0`
   - ai-dial-chat: `0.46.3`
   - ai-dial-core: `0.45.0-rc.0`
   - ai-dial-analytics-realtime: `0.24.2`
   - ai-dial-rag: `0.42.0`
   - ai-dial-log-parser: `0.3.0`
   - ai-dial-code-interpreter: `0.2.0`
   - ai-dial-app-controller: `0.4.0`
   - ai-dial-app-builder-python: `0.1.0`
   - ai-dial-quickapps-backend: `0.9.0-rc.1`
   - ai-dial-mind-map-backend: `0.14.1`
   - ai-dial-mind-map-frontend: `0.13.0`
   - ai-dial-admin-backend: `0.18.0-rc.0`
   - ai-dial-admin-frontend: `0.18.0-rc.0`
   - ai-dial-admin-deployment-manager-backend: `0.18.0-rc.0`
   - ai-dial-admin-evaluation-framework-backend: `0.1.0-rc.0`

## Before upgrade

### General notes

- Please review the [Config changes](#config-changes) chapter carefully for each component that is used in your DIAL installation. Changes in components' configuration may be required.
- Please check if any image tag overrides (`image.tag`) are present and remove them if they are not required anymore.
- Please check and add `image.repository` to change the image location for `redis`, `postgresql`, `keycloak` and `keycloakConfigCli` components to start using alternative Docker registries (e.g. Amazon ECR Public Gallery) if required.

### Release-specific notes

#### ai-dial-admin-deployment-manager-backend `0.18.0-rc.0`

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.18.0-rc.0/docs/upgrade-plans/0.18.0.md) before proceeding.

##### Breaking changes

**Spring Boot upgraded to 4.0.6 (Spring Framework 7, Hibernate 7.2) with Jackson 3 JSON serialization**

Major framework version upgrade. Jackson 3 migration caused a known regression in image entrypoint/cmd binding (fixed in this release). Deployments relying on prior serialization behavior or Hibernate/Spring internals may be affected.

| Previous configuration | Required action |
|---|---|
| Running on Spring Boot 3.5 with Jackson 2 | Review the full upgrade guide. Validate JSON serialization of image entrypoint/cmd and any custom serialization configs after upgrade. |

**OpenTelemetry env var names changed and OTEL export is now OFF by default**

Three OpenTelemetry env vars have been renamed or removed: (1) OTEL_SDK_DISABLED replaced by OTEL_EXPORT_ENABLED (inverted logic), (2) OTEL_EXPORTER_OTLP_PROTOCOL replaced by OTEL_EXPORTER_OTLP_TRANSPORT, (3) OTEL_EXPORTER_OTLP_HEADERS removed. Export is now disabled by default. Existing telemetry exporters will stop silently until migrated.

| Previous configuration | Required action |
|---|---|
| OTEL_SDK_DISABLED=false (telemetry enabled) | Replace with OTEL_EXPORT_ENABLED=true |
| OTEL_SDK_DISABLED=true (telemetry disabled) | Remove the var; export is now off by default. Or set OTEL_EXPORT_ENABLED=false explicitly. |
| OTEL_EXPORTER_OTLP_PROTOCOL set to a value | Rename to OTEL_EXPORTER_OTLP_TRANSPORT with equivalent value |
| OTEL_EXPORTER_OTLP_HEADERS set | Remove the var; find alternative configuration per the upgrade guide as this var no longer exists |

##### Removed environment variables

| Variable | Description |
|---|---|
| `OTEL_SDK_DISABLED` | Replaced by OTEL_EXPORT_ENABLED (inverted logic). Must be migrated or telemetry exporters will stop silently. |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | Replaced by OTEL_EXPORTER_OTLP_TRANSPORT. Must be renamed or telemetry exporters will stop silently. |
| `OTEL_EXPORTER_OTLP_HEADERS` | Removed with no direct replacement mentioned in release notes. Telemetry exporters will stop silently if this was relied upon. |

##### Environment variables with changed defaults

| Variable | Old default | New default | Description |
|---|---|---|---|
| `OTEL_EXPORT_ENABLED` | `true (effective: OTEL_SDK_DISABLED defaulted to false, meaning export was on)` | `false (export is now off by default)` | OpenTelemetry export is now disabled by default. Previously export was on unless OTEL_SDK_DISABLED was explicitly set to true. |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `OTEL_EXPORT_ENABLED` | `false` | No | Replaces OTEL_SDK_DISABLED with inverted logic. Set to true to enable OpenTelemetry export. Export is now off by default. |
| `OTEL_EXPORTER_OTLP_TRANSPORT` | — | No | Replaces OTEL_EXPORTER_OTLP_PROTOCOL. Configures the OTLP exporter transport. |

---

#### ai-dial-admin-evaluation-framework-backend `0.1.0-rc.0`

##### Breaking changes

**EvalSummary CSV column-group separator changed from `:` to `::`**

Any downstream consumer that parses exported CSV headers by splitting on `:` will break. Column names such as `data:prompt` are now `data::prompt`; `metric:Accuracy:score` is now `metric::Accuracy::score`.

| Previous configuration | Required action |
|---|---|
| Consumer splits CSV header on single `:` to parse column families | Update parsing logic to split on `::` instead of `:` |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `TEST_CASE_BULK_MAX_DELETE_IDS` | `10000` | No | Maximum number of IDs accepted in a single bulk-delete-by-IDs request (`DELETE /test-cases:bulk`). Must be ≥ 1. |

---

#### ai-dial-adapter-vertexai `0.36.0`

##### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment configs referencing this field must be updated.

| Previous configuration | Required action |
|---|---|
| Veo config contains `pubsub_topic` field | Remove `pubsub_topic` from the Veo model configuration |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

---

#### ai-dial-admin-backend `0.18.0-rc.0`

##### Breaking changes

**ApplicationResourceDto: flat `applicationTypeSchemaId` field replaced by polymorphic `source` field**

The `applicationTypeSchemaId` field on `ApplicationResourceDto` and `CreateApplicationResourceDto` has been removed. It is replaced by a `source` field that is a `$type`-discriminated polymorphic object with `schema` and `endpoints` variants. Any API clients, integration scripts, or tooling that reads or writes `applicationTypeSchemaId` must be updated to use the new `source` structure.

| Previous configuration | Required action |
|---|---|
| API payloads include flat `applicationTypeSchemaId` field on ApplicationResourceDto / CreateApplicationResourceDto | Update all API clients and integrations to use the new polymorphic `source` field (`$type`-discriminated with `schema` and `endpoints` variants) instead of `applicationTypeSchemaId` |

##### Config / Helm changes

- **Default changed** `features.maxTokensSupported`: `unset / not present` → `true` — New field introduced in this version; defaults to true when not explicitly configured
- **Default changed** `features.customTemperatureSupported`: `unset / not present` → `true` — New field introduced in this version; defaults to true when not explicitly configured
- **Default changed** `applicationProperties`: `null / absent` → `empty map {}` — applicationProperties for application assets now defaults to an empty map instead of being absent/null
- **Added** `features.maxTokensSupported`: New DIAL Core v0.45.0 configuration property for features; defaults to true
- **Added** `features.maxCompletionTokensSupported`: New DIAL Core v0.45.0 configuration property for features
- **Added** `features.customTemperatureSupported`: New DIAL Core v0.45.0 configuration property for features; defaults to true
- **Added** `features.reasoningEfforts`: New DIAL Core v0.45.0 configuration property for features
- **Added** `upstreams.secretExtraData`: New DIAL Core v0.45.0 configuration property for upstreams
- **Added** `models.embeddingDimensions`: New DIAL Core v0.45.0 configuration property for models

---

#### ai-dial-quickapps-backend `0.9.0-rc.1`

##### Breaking changes

**DIAL files tools now active regardless of ENABLE_PREVIEW_FEATURES**

The DIAL files tools (list / read_lines / search / find / write / edit / delete / copy / move) and the features.dial_files config field have graduated to GA. They are now enabled unconditionally, even when ENABLE_PREVIEW_FEATURES is false/unset. Deployments that relied on ENABLE_PREVIEW_FEATURES=false to suppress these tools will find them active after upgrade. The tool_call_result_offload sub-feature (features.dial_files.tool_call_result_offload) remains behind the preview flag.

| Previous configuration | Required action |
|---|---|
| ENABLE_PREVIEW_FEATURES=false — DIAL files tools were suppressed | After upgrade these tools are always active; if you need to disable them, use the features.dial_files config field explicitly or review application config, as ENABLE_PREVIEW_FEATURES no longer gates them |
| ENABLE_PREVIEW_FEATURES=true — DIAL files tools were enabled via preview flag | No action; behavior unchanged — tools remain active |

##### Config / Helm changes

- **Added** `features.dial_files`: Config field controlling DIAL files tools. Previously preview-gated (required ENABLE_PREVIEW_FEATURES=true to take effect); now always active. Operator can use this field to control DIAL files tool behavior regardless of preview flag.
- **Added** `features.dial_files.tool_call_result_offload`: Sub-feature for tool call result offload. Remains behind the ENABLE_PREVIEW_FEATURES gate; TOOL_CALL_RESULT_OFFLOAD__* env defaults still apply only when preview features are enabled.

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. Review release notes for updated storage enablement behavior. |

**Migration:** _DIAL_USE_FILE_STORAGE set to enable file storage_ → Check updated documentation for how DIAL Storage is now enabled; remove or replace this env var as directed

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' color property added to config.json theme configuration.

---

#### ai-dial-core `0.45.0-rc.0`

##### Config / Helm changes

- **Added** `features.reasoningEffortsSupported`: New feature flag to indicate support for reasoning efforts on a model/deployment.
- **Added** `features.max_tokens / features.max_completion_tokens / features.temperature`: New feature flags to expose max_tokens, max_completion_tokens, and temperature capabilities in model listings.
- **Added** `features (available endpoints flags)`: New flags to indicate available endpoints per deployment, exposed in feature listings.
- **Added** `dial-unified-config (Configuration API / MergedConfigStore / secret encryption)`: New server-side unified configuration API with merged config store and secret encryption support.
- **Added** `models[].embeddingDimensions (or equivalent model listing field)`: Embedding vector dimensions are now exposed in model listing responses.
- **Added** `models[].features.reasoningEfforts (string array)`: Reasoning efforts exposed as a string array in features listing.
- **Added** `roles.readonly-admin`: New readonly-admin role introduced to allow reading user data without write access.
- **Added** `schemas listing — mcp endpoint`: MCP endpoint is now included in the schemas listing result.
- **Added** `applications (MCP server config delivery without schema)`: Applications without a schema can now receive config delivery to MCP servers.

---
